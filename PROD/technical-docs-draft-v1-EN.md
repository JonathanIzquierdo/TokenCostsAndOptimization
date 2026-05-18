# Technical Guide: AI Token Optimization
## From basic settings to enterprise architecture

**Audience:** Developers, tech leads and architects at Visma
**Goal:** Practical guide covering all implementation levels — from five-minute settings to architecture decisions for production systems
**Prerequisite:** Basic familiarity with LLMs, AI APIs, and development environments

---

## Index

1. [Fundamentals: understanding the cost model](#1-fundamentals)
2. [Level 0 — Immediate settings (no code)](#2-level-0)
3. [Level 1 — Context architecture](#3-level-1)
4. [Level 2 — Model selection and assignment](#4-level-2)
5. [Level 3 — Automatic model routing](#5-level-3)
6. [Level 4 — Cost infrastructure and governance](#6-level-4)
7. [Level 5 — Observability with OpenTelemetry](#7-otel)
8. [Level 6 — Alternative lower-cost providers](#8-alternatives)
9. [Verified metrics reference](#9-metrics)

---

## 1. Fundamentals: understanding the cost model

### 1.1 The anatomy of an LLM request

Every request has three components billed separately:

**Input tokens:** everything you send to the model — system prompt, conversation history, tool definitions, context documents, current user message.

**Output tokens:** everything the model generates — the response, reasoning (in extended thinking models), and tool calls.

**Cache reads (when applicable):** the portion of input served from cache instead of being reprocessed.

Most important rule: **output tokens cost significantly more than input tokens**, and cache reads cost a minimal fraction of normal input.

### 1.2 2026 reference pricing

| Model | Input /MTok | Output /MTok | Cache read /MTok | Cache write /MTok |
|-------|------------|-------------|-----------------|-------------------|
| Claude Haiku 4.5 | $1.00 | $5.00 | $0.10 | $1.25 |
| Claude Sonnet 4.6 | $3.00 | $15.00 | $0.30 | $3.75 |
| Claude Opus 4.7 | $5.00 | $25.00 | $0.50 | $6.25 |
| DeepSeek V4-Flash (direct) | $0.14 | $0.28 | — | — |
| DeepSeek V4-Flash (via Azure) | ~$0.19 | ~$0.38 | — | — |
| Llama 4 via Groq | ~$0.11 | ~$0.34 | — | — |
| Llama 4 via Together AI | ~$0.18 | ~$0.59 | — | — |

Sources: Anthropic official (April 2026), DeployBase, ToolHalla, inference.net

### 1.3 The accumulated context problem

The cost of a long session isn't linear — it's quadratic without active management:
- Turn 1: 5,000 input tokens
- Turn 10: ~15,000 tokens
- Turn 50: 200,000 tokens per call

Audit of 30 production teams (LeanOps, May 2026): **62% of the AI bill is re-sent context.**

### 1.4 The output multiplier

Output tokens cost 5x more than input in Anthropic. Reducing outputs has more cost impact than reducing inputs.

**Practical rule:** always set `max_tokens` explicitly.

### 1.5 Extended thinking: the cost you can't see

```python
# WRONG — display omitted does NOT reduce the cost
response = client.messages.create(
    model="claude-opus-4-7",
    thinking={
        "type": "enabled",
        "budget_tokens": 5000,
        "display": "omitted"  # Those 5000 tokens are still billed
    }
)

# CORRECT — limit the budget based on actual task complexity
response = client.messages.create(
    model="claude-opus-4-7",
    thinking={"type": "adaptive", "effort": "low"}
)
```

**Quantified impact:** 500 visible output tokens + 2,000 thinking tokens = 5x more expensive than without thinking. (Source: PECollective, 2026)

---

## 2. Level 0 — Immediate settings

### 2.1 Auto Mode in GitHub Copilot

Switch the selector to "Auto" in VSCode. Dynamically selects the most efficient model per request.

**Verified benefit:** 10% discount on the model multiplier. (Source: GitHub Changelog, April 2026)

### 2.2 chat.tools.compressOutput.enabled

```json
{ "chat.tools.compressOutput.enabled": true }
```

Post-processes terminal output: collapses unchanged git diff hunks, discards lockfile diffs, reduces `ls -l` to filenames, removes progress bars. Available since VSCode 1.120.

### 2.3 Tool search — deferred MCPs

Active by default for Anthropic models (Sonnet 4.5+). For GPT models:

```json
{ "github.copilot.chat.responsesApi.toolSearchTool.enabled": true }
```

**Verified benefit:** up to 20% token savings per request. (Source: Visual Studio Magazine, 2026)

### 2.4 Audit active MCPs

10 MCPs × 500 tokens/schema = 5,000 tokens overhead/request = **$1,500/month** on 100K requests to Sonnet without using any of them.

---

## 3. Level 1 — Context architecture

### 3.1 Prompt caching

| Operation | Cost |
|-----------|------|
| Cache write TTL 5 min | 1.25x base |
| Cache write TTL 1 hour | 2.0x base |
| Cache read | 0.10x base (90% discount) |

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": "You are a Visma engineering assistant...",
            "cache_control": {"type": "ephemeral"}
        },
        {
            "type": "text",
            "text": knowledge_base_content,
            "cache_control": {"type": "ephemeral"}
        }
    ],
    messages=[{"role": "user", "content": user_message}]
)

usage = response.usage
print(f"Cache creation: {usage.cache_creation_input_tokens}")
print(f"Cache read: {usage.cache_read_input_tokens}")
```

**Impact:** RAG app 50K tokens × 1000 queries/day → 84% savings. (Source: Finout, 2026)

### 3.2 Conversation history management

**Strategy A: sliding window**
```python
def get_windowed_history(messages, window_size=10):
    return messages[-window_size:] if len(messages) > window_size else messages
```

**Strategy B: progressive summarization** (use Haiku to summarize — cheaper)
```python
def compress_old_history(messages, recent_window=5):
    if len(messages) <= recent_window:
        return messages
    old = messages[:-recent_window]
    recent = messages[-recent_window:]
    summary = client.messages.create(
        model="claude-haiku-4-5",
        max_tokens=500,
        messages=[{"role": "user",
            "content": f"Summarize in 3 paragraphs:\n{format_messages(old)}"}]
    ).content[0].text
    return [{"role": "user", "content": f"[Summary]\n{summary}"},
            {"role": "assistant", "content": "Understood, continuing."}] + recent
```

**Strategy C: Compaction API (recommended for agents)**
```python
response = client.beta.messages.create(
    betas=["compact-2026-01-12"],
    model="claude-sonnet-4-6",
    max_tokens=4096,
    messages=messages,
    context_management={"edits": [{"type": "compact_20260112",
                                    "context_token_threshold": 50_000}]}
)
if hasattr(response, 'context_compact_usage'):
    before = response.context_compact_usage.tokens_before
    after = response.context_compact_usage.tokens_after
    print(f"Compaction: {before} → {after} tokens")
```

Beta since January 2026. ZDR eligible. (Source: Claude Lab, Anthropic Docs)

### 3.3 RAG and output budgets

```python
# Only top-K relevant chunks, not the entire knowledge base
chunks = vector_store.search(query=user_query, top_k=5)
context = "\n\n".join([c.text[:500] for c in chunks])

# Output budgets per task type
OUTPUT_BUDGETS = {
    "classification": 50, "extraction": 200, "summarization": 500,
    "code_review": 1000, "code_generation": 2000, "architecture": 3000
}
```

RAG reduces context tokens up to 70% vs context stuffing. (Source: Koombea, 2026)

---

## 4. Level 2 — Model selection and assignment

```python
MODEL_ASSIGNMENT = {
    # Haiku — pattern matching, no reasoning required
    "classification": "claude-haiku-4-5",
    "entity_extraction": "claude-haiku-4-5",
    "data_formatting": "claude-haiku-4-5",
    "file_navigation": "claude-haiku-4-5",
    "simple_qa": "claude-haiku-4-5",
    "translation": "claude-haiku-4-5",
    # Sonnet — optimal quality/cost balance
    "code_generation": "claude-sonnet-4-6",
    "code_review": "claude-sonnet-4-6",
    "summarization": "claude-sonnet-4-6",
    "general_chat": "claude-sonnet-4-6",
    "debugging": "claude-sonnet-4-6",
    # Opus — only when complex reasoning justifies the price
    "architecture_design": "claude-opus-4-7",
    "complex_reasoning": "claude-opus-4-7",
    "agent_coordination": "claude-opus-4-7",
}

def classify_and_route(user_input, client):
    """Use Haiku to classify — cheap and fast."""
    result = client.messages.create(
        model="claude-haiku-4-5", max_tokens=50,
        system="Classify the message into: classification, extraction, summarization, "
               "code_generation, code_review, debugging, architecture_design, "
               "complex_reasoning, general_chat. ONLY the category name.",
        messages=[{"role": "user", "content": user_input}]
    )
    task_type = result.content[0].text.strip().lower()
    return task_type, MODEL_ASSIGNMENT.get(task_type, "claude-sonnet-4-6")
```

---

## 5. Level 3 — Automatic model routing

### 5.1 RouteLLM (open source, UC Berkeley/LMSYS)

**ICLR 2025 results:** 95% GPT-4 quality with 26% calls → 48% savings. With augmentation: 75% cost reduction.

```bash
pip install routellm
```

```python
from routellm.controller import Controller

client = Controller(
    routers=["mf"],
    strong_model="anthropic/claude-opus-4-7",
    weak_model="anthropic/claude-haiku-4-5",
    config={"mf": {"threshold": 0.3}}
)

response = client.chat.completions.create(
    model="router",
    messages=[{"role": "user", "content": user_message}]
)
print(f"Model used: {response.model}")
```

### 5.2 Enterprise AI Gateways

**LiteLLM (self-hosted, open source):**
```python
from litellm import Router
router = Router(
    model_list=[
        {"model_name": "prod", "litellm_params": {"model": "claude-sonnet-4-6"},
         "tpm": 100000, "rpm": 1000},
        {"model_name": "prod", "litellm_params": {"model": "azure/claude-sonnet-4-6"},
         "tpm": 100000, "rpm": 1000}  # Azure fallback
    ],
    budget_manager={"type": "redis", "redis_url": os.getenv("REDIS_URL")}
)
```

**Portkey (managed, compliance):**
```python
from portkey_ai import Portkey
client = Portkey(api_key="PORTKEY_API_KEY", virtual_key="ANTHROPIC_VIRTUAL_KEY")
response = client.chat.completions.create(
    model="claude-sonnet-4-6",
    messages=messages,
    metadata={"team": "engineering", "feature": "code-review"}
)
```

| Gateway | Type | Models | Best for |
|---------|------|--------|----------|
| LiteLLM | Open source | 100+ | Full control, zero fees |
| Portkey | Managed | 1,600+ | Compliance, guardrails, SOC2 |
| OpenRouter | SaaS | 300+ | Fast experimentation |

---

## 6. Level 4 — Cost infrastructure and governance

### 6.1 Batch API

**50% discount, identical quality.** Combined with caching: up to 95% savings. (Source: Anthropic official, PECollective)

```python
batch = client.messages.batches.create(requests=[
    {
        "custom_id": f"doc-{doc.id}",
        "params": {
            "model": "claude-sonnet-4-6", "max_tokens": 200,
            "system": [{"type": "text", "text": system_prompt,
                        "cache_control": {"type": "ephemeral"}}],
            "messages": [{"role": "user", "content": doc.text[:1000]}]
        }
    } for doc in documents
])

# 300K output tokens in Batch (beta)
batch = client.messages.batches.create(
    requests=requests,
    extra_headers={"anthropic-beta": "output-300k-2026-03-24"}
)

# Polling
import time
while client.messages.batches.retrieve(batch.id).processing_status != "ended":
    time.sleep(60)

results = {r.custom_id: r.result.message.content[0].text
           for r in client.messages.batches.results(batch.id)
           if r.result.type == "succeeded"}
```

### 6.2 DeepSeek via Azure

```python
from openai import AzureOpenAI
client = AzureOpenAI(
    api_key=os.getenv("AZURE_API_KEY"),
    api_version="2024-12-01-preview",
    azure_endpoint=os.getenv("AZURE_ENDPOINT")  # EU region
)
response = client.chat.completions.create(
    model="deepseek-v4-flash", max_tokens=50,
    messages=[{"role": "user", "content": text_to_classify}]
)
```

~15x cheaper than Sonnet for high-volume tasks, with European compliance. (Source: DeployBase, 2026)

### 6.3 Cost observability and budget alerts

```python
PRICING = {
    "claude-haiku-4-5": {"input": 1e-6, "output": 5e-6,
                          "cache_read": 0.1e-6, "cache_write": 1.25e-6},
    "claude-sonnet-4-6": {"input": 3e-6, "output": 15e-6,
                           "cache_read": 0.3e-6, "cache_write": 3.75e-6},
    "claude-opus-4-7": {"input": 5e-6, "output": 25e-6,
                         "cache_read": 0.5e-6, "cache_write": 6.25e-6},
}

def calculate_cost(model, input_tokens, output_tokens,
                   cache_read=0, cache_write=0):
    p = PRICING.get(model, {})
    if not p:
        return 0.0
    return round(
        input_tokens * p["input"] + output_tokens * p["output"] +
        cache_read * p["cache_read"] + cache_write * p["cache_write"], 6
    )

class BudgetMonitor:
    def __init__(self, monthly_budget_usd, user_id):
        self.budget = monthly_budget_usd
        self.user_id = user_id
        self.spend = 0.0
        self.thresholds = [0.5, 0.8, 1.0]
        self.alerted = set()

    def check(self, new_spend):
        self.spend += new_spend
        ratio = self.spend / self.budget
        for t in self.thresholds:
            if ratio >= t and t not in self.alerted:
                self.alerted.add(t)
                send_notification(
                    f"⚠️ {self.user_id}: {ratio:.0%} of budget "
                    f"(${self.spend:.2f}/${self.budget:.2f})",
                    channel="ai-costs-alerts"
                )
```

---

## 7. Level 5 — Observability with OpenTelemetry

Without instrumentation, you're flying blind. The billing shows you the monthly total — not which developer drove it, which workflow caused it, or how it changes over time.

### 7.1 OTel in Claude Code (CLI)

Claude Code has native telemetry. Off by default — opt-in with one environment variable.

```bash
# Add to ~/.zshrc or ~/.bashrc to persist
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=otlp
export OTEL_LOGS_EXPORTER=otlp
export OTEL_TRACES_EXPORTER=otlp
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
export OTEL_METRIC_EXPORT_INTERVAL=10000  # 10 seconds

# Custom attributes to filter by team in the dashboard
export OTEL_RESOURCE_ATTRIBUTES="team.id=platform,department=engineering"
```

Sources: Claude Code Docs — Observability, SigNoz Docs, Bindplane Blog 2026

By default, **no prompt content is captured** — only metadata (model, tokens, duration, tools). To capture full content:

```bash
export COPILOT_OTEL_CAPTURE_CONTENT=true
# Or in VSCode settings:
# "github.copilot.chat.otel.captureContent": true
```

> Treat the telemetry stream as safer than raw prompt logging, but not automatically safe for all backends. Evaluate privacy implications before enabling in production.

Source: VSCode Copilot Chat — Monitoring Agents, Microsoft GitHub

### 7.2 OTel in Copilot Chat (VSCode)

Native OTel support since VSCode 1.118. Following OTel GenAI Semantic Conventions.

```json
{
  "github.copilot.chat.otel.enabled": true,
  "github.copilot.chat.otel.endpoint": "http://localhost:4317"
}
```

**Span hierarchy produced:**
```
client invoke_agent copilot [~15s]
├── chat gpt-4o [~3s]
├── execute_tool readFile [~50ms]
├── execute_tool runCommand [~2s]
└── chat gpt-4o [~4s]
```

Source: Microsoft VSCode Copilot Chat — Agent Monitoring Docs

### 7.3 Local observability stack (quickstart)

```bash
# Aspire Dashboard — UI on :18888, OTLP endpoint on :4317
docker run --rm -d \
  -p 18888:18888 \
  -p 4317:18889 \
  --name aspire-dashboard \
  mcr.microsoft.com/dotnet/aspire-dashboard:latest
```

Open http://localhost:18888 — Claude Code and Copilot traces appear in real time.

Source: Microsoft VSCode Copilot Chat — Agent Monitoring Docs

### 7.4 Production stack with Grafana

```yaml
# docker-compose.yml — OTel Collector + Prometheus + Grafana
version: '3.8'
services:
  otel-collector:
    image: otel/opentelemetry-collector-contrib:latest
    command: ["--config", "/etc/otel/config.yaml"]
    volumes:
      - ./otel-config.yaml:/etc/otel/config.yaml
    ports:
      - "4317:4317"   # gRPC
      - "4318:4318"   # HTTP

  prometheus:
    image: prom/prometheus:latest
    ports: ["9090:9090"]
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./data/prometheus:/prometheus
    command: ["--storage.tsdb.retention.time=90d",
              "--config.file=/etc/prometheus/prometheus.yml"]

  grafana:
    image: grafana/grafana:latest
    ports: ["3000:3000"]
    environment: ["GF_SECURITY_ADMIN_PASSWORD=admin"]
```

Source: AI.cc — Claude Code Monitor Tutorial, Sealos Blog — Claude Code Metrics

### 7.5 Centralized enterprise configuration (admin-managed)

```json
// Managed settings file — distributable via MDM
// High precedence — users cannot override
{
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_METRICS_EXPORTER": "otlp",
    "OTEL_LOGS_EXPORTER": "otlp",
    "OTEL_EXPORTER_OTLP_PROTOCOL": "grpc",
    "OTEL_EXPORTER_OTLP_ENDPOINT": "http://collector.visma.internal:4317",
    "OTEL_EXPORTER_OTLP_HEADERS": "Authorization=Bearer <token>",
    "OTEL_METRIC_EXPORT_INTERVAL": "10000"
  }
}
```

Source: SigNoz Docs — Claude Code Monitoring, Bindplane Blog 2026

### 7.6 Key metrics exported

- `gen_ai.client.token.usage` — tokens by type (input, output, cache read, cache write)
- `gen_ai.client.operation.duration` — latency per operation
- Estimated cost per session, per model, per user

**Available filtering attributes:**
- `gen_ai.request.model`, `gen_ai.provider.name`, `gen_ai.tool.name`
- `team.id`, `department`, `user.id` — custom attributes for team-level filtering

### 7.7 Compatible observability backends

| Backend | Type | Best for |
|---------|------|----------|
| Aspire Dashboard | Local, Docker | Quickstart, local dev |
| Grafana + Prometheus | Self-hosted | Full control, zero cost |
| Jaeger | Self-hosted | Detailed tracing |
| Datadog | Managed | Integration with existing infra |
| Honeycomb | Managed | Ad-hoc queries and exploration |
| Langfuse | Managed/self-hosted | LLM-specific observability |
| SigNoz | Managed/self-hosted | OTel native, open source |
| Azure Monitor | Managed | Microsoft ecosystem |

All support OTLP — no lock-in.

Source: Microsoft VSCode Copilot Chat — Agent Monitoring Docs

---

## 8. Level 6 — Alternative lower-cost providers

The inference API market in 2026 has 11+ production providers. Prices vary **625x** between the cheapest and the most expensive. Most teams pay 4 to 30x more than necessary for many tasks.

> "Defaulting to the newest model without measuring whether the quality difference justifies the cost is how teams end up in quarterly reviews they'd rather skip."

Source: CloudZero — LLM API Pricing Comparison 2026

### 8.1 Market map 2026

**Tier 1 — Frontier models (premium):** OpenAI GPT-5, Claude Opus 4.7, Gemini 3 Pro. Top benchmarks. High price. Justified for complex reasoning and critical tasks.

**Tier 2 — Mid-tier models (optimal quality/cost):** Claude Sonnet 4.6, GPT-5.4 Mini, Gemini 3 Flash. For most production work.

**Tier 3 — Open-weight via inference providers (high volume, low cost):** Llama 4, Mixtral, Qwen 2.5, DeepSeek via providers. For well-defined, predictable tasks. 4–30x cheaper than frontier.

**Self-hosted:** Eliminates per-token cost. Only makes sense above $5K–10K/month in API spend with 70%+ GPU utilization.

Source: inference.net — LLM API Pricing Comparison 2026, morphllm.com

### 8.2 Inference providers: Groq, Together AI, Fireworks AI

These providers don't develop their own models — they run open-weight models (Llama, Mixtral, Qwen, DeepSeek) and compete on price and latency. Their APIs are OpenAI-compatible, making migration a simple endpoint change.

#### Groq — LPU hardware, extreme speed

Custom silicon (Language Processing Unit) built specifically for transformer inference. 500–800 tokens/second on Llama 4 vs 50–150 tok/s on standard GPUs.

**Representative pricing (Groq, March 2026):**
- Llama 3.1 70B: ~$0.59/MTok input (OpenAI GPT-4o is $2.50/MTok → 4x more expensive)
- Llama 4 models: $0.07–$0.20/MTok input

Source: ToolHalla — Groq vs Together AI vs Fireworks 2026

```python
from openai import OpenAI
client = OpenAI(
    api_key=os.getenv("GROQ_API_KEY"),
    base_url="https://api.groq.com/openai/v1"
)
response = client.chat.completions.create(
    model="llama-4-scout-17b-16e-instruct",
    messages=[{"role": "user", "content": text_to_classify}],
    max_tokens=50
)
```

**When to use Groq:** latency-sensitive tasks needing sub-100ms response. Classification, extraction, end-user FAQs. NOT for complex reasoning tasks.

#### Together AI — The widest marketplace

100+ open-source models, LoRA and full fine-tuning support, dedicated deployments.

**Representative pricing (Together AI, 2026):**
- Llama 4 Maverick: ~$0.18/MTok input
- DeepSeek V3.2: ~$0.14/MTok input

```python
from openai import OpenAI
client = OpenAI(
    api_key=os.getenv("TOGETHER_API_KEY"),
    base_url="https://api.together.xyz/v1"
)
response = client.chat.completions.create(
    model="meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8",
    messages=messages,
    max_tokens=OUTPUT_BUDGETS.get(task_type, 500)
)
```

**When to use Together:** high-volume batch workloads, fine-tuning on proprietary data.

Source: PkgPulse — Groq vs Together AI vs Fireworks 2026

#### Fireworks AI — Production-grade with SLAs

Formal SLAs, optimized function calling, dedicated deployments.

**When to use Fireworks:** production needing formal availability SLA, heavy function calling pipelines.

Source: ToolHalla — Groq vs Together AI vs Fireworks 2026

### 8.3 Mistral — The European option

Mistral is the only major provider headquartered in Europe (Paris). For organizations with strict European data residency requirements, it's the most direct option.

**Advantages for Visma:**
- EU-based, EU data — native GDPR compliance
- Pricing significantly below OpenAI
- Open-weight models available for self-hosting
- OpenAI-compatible API

```python
from openai import OpenAI
client = OpenAI(
    api_key=os.getenv("MISTRAL_API_KEY"),
    base_url="https://api.mistral.ai/v1"
)
response = client.chat.completions.create(
    model="mistral-small-latest",
    messages=messages, max_tokens=200
)
```

Source: PE Collective — Best OpenAI Alternatives 2026

### 8.4 Decision guide: when to move away from Anthropic/OpenAI

It's not a full migration — it's a routing strategy.

| Task | Recommended provider | Reason |
|------|---------------------|--------|
| Complex reasoning, architecture | Claude Opus / GPT-5 | Irreplaceable quality |
| General code work | Claude Sonnet | Best balance |
| Bulk classification/extraction | Groq (Llama 4) | 4–10x cheaper, sub-100ms |
| Offline batch, high volume | Together AI / Fireworks | Low price with SLA |
| EU data residency requirement | Mistral | Only major EU-based provider |
| Very long context (>200K tokens) | Gemini Flash | Best price at long context |
| EU compliance + minimum cost | DeepSeek via Azure | Compliance + 15x cheaper than Sonnet |

**Practical rule:** start with the free tier of Groq, Together, and Fireworks. Benchmark on your actual specific workload. Commit when you have data, not opinions.

Source: ToolHalla 2026, inference.net 2026

### 8.5 Self-hosting: when does it make sense

vLLM eliminates per-token cost entirely, but has conditions.

**Only makes sense when:**
- API spend exceeds $5K–10K/month
- Team has DevOps capacity to maintain the infrastructure
- Can sustain 70%+ GPU utilization (below that, API providers win on cost)
- Full privacy or custom fine-tuning on proprietary data is required

Source: morphllm.com — LLM API Comparison 2026

---

## 9. Verified metrics reference

| Metric | Value | Source | URL |
|--------|-------|--------|-----|
| Re-sent context = % of bill | 62% | LeanOps 30 teams 2026 | https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/ |
| Tokens in 20-turn conversation | 5,000–10,000 unnecessary | Redis 2026 | https://redis.io/blog/llm-token-optimization-speed-up-apps/ |
| Tokens turn 1 vs turn 50 | 5K → 200K | Redblink | https://redblink.com/ai-token-cost-optimization/ |
| Output vs input multiplier | 4–6x | Redis 2026 | https://redis.io/blog/llm-token-optimization-speed-up-apps/ |
| Anthropic cache reads discount | 90% | TokenOptimize.dev | https://www.tokenoptimize.dev/guides/llm-token-optimization-strategies |
| RAG app savings with caching | 88–95% | Finout | https://www.finout.io/blog/anthropic-api-pricing |
| VSCode prompt caching reuse rate | 93% | VS Magazine | https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx |
| RAG token reduction | up to 70% | Koombea | https://ai.koombea.com/blog/llm-cost-optimization |
| RouteLLM quality/calls | 95% quality / 26% calls | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| RouteLLM savings vs baseline | 48% | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| RouteLLM with augmentation | 75% reduction | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| VSCode tool search savings | up to 20% | VS Magazine | https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx |
| Copilot Auto Mode discount | 10% multiplier | GitHub Changelog | https://github.blog/changelog/2026-04-17-github-copilot-cli-now-supports-copilot-auto-model-selection/ |
| Copilot top users = spend | 10–15% = 60–70% | Synapx | https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/ |
| Agentic workflow cost | ~3.5x flat fee | Synapx | https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/ |
| Batch API discount | 50% input and output | Anthropic official | https://www.finout.io/blog/anthropic-api-pricing |
| Batch + caching combined | up to 95% | PECollective | https://pecollective.com/tools/anthropic-api-pricing/ |
| Extended thinking display omitted | No savings | CheckThat.ai | https://checkthat.ai/brands/anthropic/pricing |
| Thinking 2000t + output 500t | 5x more expensive | PECollective | https://pecollective.com/tools/anthropic-api-pricing/ |
| Azure EA only discount | 15–25% | Microsoft Negotiations | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| Azure EA + MACC discount | 23–28% | Microsoft Negotiations | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| DeepSeek via Azure markup | +20–35% | DeployBase | https://deploybase.ai/articles/deepseek-v3-pricing |
| DeepSeek V4-Flash vs Sonnet | ~15x cheaper input | 2026 prices | https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/ |
| AI % of total IT spend | up to 50% | Deloitte 2026 | https://redblink.com/ai-token-cost-optimization/ |
| Groq vs OpenAI GPT-4o | 4–10x cheaper Llama 4 | ToolHalla 2026 | https://toolhalla.ai/blog/groq-vs-together-vs-fireworks-2026 |
| 2026 market price range | $0.08–$25/MTok output | morphllm.com | https://www.morphllm.com/llm-api |
| Self-hosting break-even | $5K–10K/month in API spend | morphllm.com | https://www.morphllm.com/llm-api |
| Prices dropped | 60–80% in 12 months | ToolHalla 2026 | https://toolhalla.ai/blog/groq-vs-together-vs-fireworks-2026 |
