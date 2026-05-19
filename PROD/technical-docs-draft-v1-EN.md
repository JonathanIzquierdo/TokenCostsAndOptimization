# Technical Guide: AI Token Optimization
## From basic settings to enterprise architecture

**Audience:** Developers, tech leads and architects at Visma
**Prerequisite:** Basic familiarity with LLMs, AI APIs, and development environments

---

## Index

1. [Fundamentals](#1-fundamentals)
2. [Level 0 - Immediate settings](#2-level-0)
3. [Level 1 - Context architecture](#3-level-1)
4. [Level 2 - Model selection](#4-level-2)
5. [Level 3 - Automatic model routing](#5-level-3)
6. [Level 4 - Cost infrastructure and governance](#6-level-4)
7. [Level 5 - Observability with OpenTelemetry](#7-otel)
8. [Level 6 - Alternative lower-cost providers](#8-alternatives)
9. [Verified metrics reference](#9-metrics)

---

## 1. Fundamentals

### 1.1 Anatomy of an LLM request

**Input tokens:** system prompt, history, tool definitions, documents, user message.
**Output tokens:** response, reasoning (extended thinking), tool calls. **Cost 5x more.**
**Cache reads:** 10% of normal input price.

### 1.2 2026 reference pricing

| Model | Input /MTok | Output /MTok | Cache read /MTok | Cache write /MTok |
|-------|------------|-------------|-----------------|-------------------|
| Claude Haiku 4.5 | $1.00 | $5.00 | $0.10 | $1.25 |
| Claude Sonnet 4.6 | $3.00 | $15.00 | $0.30 | $3.75 |
| Claude Opus 4.7 | $5.00 | $25.00 | $0.50 | $6.25 |
| DeepSeek V4-Flash (direct) | $0.14 | $0.28 | — | — |
| DeepSeek V4-Flash (via Azure) | ~$0.19 | ~$0.38 | — | — |
| Llama 4 via Groq | ~$0.11 | ~$0.34 | — | — |

Sources: Anthropic official (April 2026), DeployBase, ToolHalla

> **Opus 4.7 note:** Ships with a new tokenizer that can generate up to 35% more tokens for the same input text vs Opus 4.6. The per-token price didn't change, but effective cost per request can be up to 35% higher. Benchmark your workloads before migrating.
> Source: [Finout — Anthropic API Pricing 2026](https://www.finout.io/blog/anthropic-api-pricing)

### 1.3 The accumulated context problem

Turn 1: 5K tokens. Turn 10: ~15K. Turn 50: **200K tokens per call.**
**62% of the AI bill is re-sent context.** (LeanOps, 30 teams, 2026)

### 1.4 Extended thinking: the invisible cost

```python
# WRONG — display: omitted does NOT reduce cost
response = client.messages.create(
    model="claude-opus-4-7",
    thinking={"type": "enabled", "budget_tokens": 5000, "display": "omitted"}
    # Those 5000 tokens are still billed as output
)
# CORRECT — limit the budget
response = client.messages.create(
    model="claude-opus-4-7",
    thinking={"type": "adaptive", "effort": "low"}
)
```
500 output tokens + 2,000 thinking = **5x more expensive** than without thinking. (PECollective, 2026)

---

## 2. Level 0 — Immediate settings

### 2.1 Auto Mode in GitHub Copilot
Switch the selector to "Auto" in VSCode. 10% automatic discount on the model multiplier. (GitHub Changelog, April 2026)

### 2.2 chat.tools.compressOutput.enabled
```json
{ "chat.tools.compressOutput.enabled": true }
```
Post-processes terminal output: collapses unchanged diff hunks, discards lockfile diffs, reduces `ls -l` to filenames, removes progress bars. VSCode 1.120.

### 2.3 Tool search — deferred MCPs
Default for Anthropic models (Sonnet 4.5+). For GPT models:
```json
{ "github.copilot.chat.responsesApi.toolSearchTool.enabled": true }
```
Up to 20% token savings per request. (Visual Studio Magazine, 2026)

### 2.4 Agent Debug Log Panel — Full trace inside VSCode

The most direct way to see what's happening in an agent session without configuring any external backend. Shows tokens consumed, tool calls executed, model turns, subagents, errors, and a **visual flow chart of the agent** — all rendered directly inside the editor.

**How to open:**
- Overflow menu `(...)` in the Copilot Chat panel → **"Show Agent Debug Logs"**
- Or Command Palette (`Cmd/Ctrl+Shift+P`) → **"Developer: Open Agent Debug Logs"**

**Requires enabling:**
```json
{ "github.copilot.chat.agentDebugLog.enabled": true }
```

**What it shows per session:**
- **Overview:** total model turns, tool calls executed, total tokens consumed, errors
- **View Logs:** chronological list of all events with timestamps and details. Filtering by event type
- **Agent Flow Chart:** visual graph of the execution flow. Shows how model turns, tool calls, and nested subagents relate to each other

Source: [VSCode Docs — Debug Chat Interactions](https://code.visualstudio.com/docs/copilot/chat/chat-debug-view)

**Persist past sessions to disk:**
```json
{ "github.copilot.chat.agentDebugLog.fileLogging.enabled": true }
```
With this enabled you can go back to any past session and inspect tool calls, LLM requests, token usage, and errors after the fact.
Source: [dvlprlife.com — Quick Tips: Debug Copilot Agent Session Logs, May 2026](https://www.dvlprlife.com/2026/05/quick-tips-debug-copilot-agent-session-logs-in-vs-code/)

**Export to OTLP JSON:** Each session can be exported for sharing, archiving, or ingesting into any OTel backend (Jaeger, Grafana, Langfuse, etc.). Click the Export (download) icon in the panel toolbar.

**The /troubleshoot command:** With file logging enabled, ask Copilot to analyze its own logs:
```
/troubleshoot how many tokens did I use?
/troubleshoot why did it skip that tool?
```
Source: [VSCode Docs — Troubleshoot AI in Visual Studio Code](https://code.visualstudio.com/docs/copilot/troubleshooting)

### 2.5 Chat Debug View — Request-by-request inspection

Shows the detail of each individual request: full system prompt sent, user prompt, included context (files, snippets, history), and model response.

**How to open:** `(...)` menu → **"Show Chat Debug View"** or Command Palette → **"Developer: Show Chat Debug View"**

Source: [Medium — GitHub Copilot Token Usage Explained, Simform Engineering, May 2026](https://medium.com/simform-engineering/github-copilot-token-usage-explained-with-practical-cost-control-03062b15ecb0)

### 2.6 Token usage visibility for BYOK (VSCode 1.120)
Fixes token accounting for models with a personal API key — previously always showed 0%.

### 2.7 Audit active MCPs
10 MCPs × 500 tokens = 5,000 tokens overhead/request = **$1,500/month** on 100K requests to Sonnet without using any of them.

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
    model="claude-sonnet-4-6", max_tokens=1024,
    system=[
        {"type": "text", "text": "You are a Visma engineering assistant...",
         "cache_control": {"type": "ephemeral"}},
        {"type": "text", "text": knowledge_base_content,
         "cache_control": {"type": "ephemeral"}}
    ],
    messages=[{"role": "user", "content": user_message}]
)
print(f"Cache read: {response.usage.cache_read_input_tokens}")
```
RAG app 50K tokens × 1,000 queries/day → **84% savings** on that portion. (Finout, 2026)

### 3.2 History management

**Sliding window:**
```python
def get_windowed_history(messages, window_size=10):
    return messages[-window_size:] if len(messages) > window_size else messages
```

**Progressive summarization** (use Haiku to summarize — cheaper):
```python
def compress_old_history(messages, recent_window=5):
    if len(messages) <= recent_window:
        return messages
    old, recent = messages[:-recent_window], messages[-recent_window:]
    summary = client.messages.create(
        model="claude-haiku-4-5", max_tokens=500,
        messages=[{"role": "user", "content": f"Summarize in 3 paragraphs:\n{format_messages(old)}"}]
    ).content[0].text
    return [{"role": "user", "content": f"[Summary]\n{summary}"},
            {"role": "assistant", "content": "Understood."}] + recent
```

**Compaction API** (recommended for agents, beta January 2026, ZDR eligible):
```python
response = client.beta.messages.create(
    betas=["compact-2026-01-12"], model="claude-sonnet-4-6", max_tokens=4096,
    messages=messages,
    context_management={"edits": [{"type": "compact_20260112", "context_token_threshold": 50_000}]}
)
```

### 3.3 RAG and output budgets
```python
chunks = vector_store.search(query=user_query, top_k=5)
context = "\n\n".join([c.text[:500] for c in chunks])
OUTPUT_BUDGETS = {
    "classification": 50, "extraction": 200, "summarization": 500,
    "code_review": 1000, "code_generation": 2000, "architecture": 3000
}
```
RAG reduces context tokens up to 70% vs context stuffing. (Koombea, 2026)

---

## 4. Level 2 — Model selection and assignment

```python
MODEL_ASSIGNMENT = {
    "classification": "claude-haiku-4-5", "entity_extraction": "claude-haiku-4-5",
    "data_formatting": "claude-haiku-4-5", "file_navigation": "claude-haiku-4-5",
    "simple_qa": "claude-haiku-4-5",
    "code_generation": "claude-sonnet-4-6", "code_review": "claude-sonnet-4-6",
    "summarization": "claude-sonnet-4-6", "general_chat": "claude-sonnet-4-6",
    "debugging": "claude-sonnet-4-6",
    "architecture_design": "claude-opus-4-7", "complex_reasoning": "claude-opus-4-7",
    "agent_coordination": "claude-opus-4-7",
}

def classify_and_route(user_input, client):
    result = client.messages.create(
        model="claude-haiku-4-5", max_tokens=50,
        system="Classify into: classification, extraction, summarization, code_generation, "
               "code_review, debugging, architecture_design, complex_reasoning, general_chat. "
               "ONLY the category name.",
        messages=[{"role": "user", "content": user_input}]
    )
    task_type = result.content[0].text.strip().lower()
    return task_type, MODEL_ASSIGNMENT.get(task_type, "claude-sonnet-4-6")
```

---

## 5. Level 3 — Automatic model routing

### 5.1 RouteLLM (ICLR 2025, UC Berkeley/LMSYS)
95% GPT-4 quality with 26% calls → 48% savings. With augmentation: **75% reduction**.

```python
from routellm.controller import Controller
client = Controller(
    routers=["mf"], strong_model="anthropic/claude-opus-4-7",
    weak_model="anthropic/claude-haiku-4-5", config={"mf": {"threshold": 0.3}}
)
response = client.chat.completions.create(model="router", messages=[...])
```

### 5.2 Enterprise AI Gateways

```python
# LiteLLM — self-hosted, budget controls
from litellm import Router
router = Router(
    model_list=[
        {"model_name": "prod", "litellm_params": {"model": "claude-sonnet-4-6"}, "tpm": 100000},
        {"model_name": "prod", "litellm_params": {"model": "azure/claude-sonnet-4-6"}, "tpm": 100000}
    ],
    budget_manager={"type": "redis", "redis_url": os.getenv("REDIS_URL")}
)

# Portkey — managed, compliance
from portkey_ai import Portkey
client = Portkey(api_key="PORTKEY_API_KEY", virtual_key="ANTHROPIC_VIRTUAL_KEY")
response = client.chat.completions.create(
    model="claude-sonnet-4-6", messages=messages,
    metadata={"team": "engineering", "feature": "code-review"}
)
```

| Gateway | Type | Best for |
|---------|------|----------|
| LiteLLM | Open source | Full control, zero fees |
| Portkey | Managed | Compliance, guardrails, SOC2 |
| OpenRouter | SaaS | Fast experimentation |

---

## 6. Level 4 — Cost infrastructure and governance

### 6.1 Batch API (50% discount, identical quality)

```python
batch = client.messages.batches.create(requests=[
    {"custom_id": f"doc-{doc.id}", "params": {
        "model": "claude-sonnet-4-6", "max_tokens": 200,
        "system": [{"type": "text", "text": system_prompt, "cache_control": {"type": "ephemeral"}}],
        "messages": [{"role": "user", "content": doc.text[:1000]}]
    }} for doc in documents
])
import time
while client.messages.batches.retrieve(batch.id).processing_status != "ended":
    time.sleep(60)
results = {r.custom_id: r.result.message.content[0].text
           for r in client.messages.batches.results(batch.id)
           if r.result.type == "succeeded"}
```
With caching: up to **95% savings**. (PECollective, 2026)

### 6.2 DeepSeek via Azure
```python
from openai import AzureOpenAI
client = AzureOpenAI(api_key=os.getenv("AZURE_API_KEY"),
                     api_version="2024-12-01-preview",
                     azure_endpoint=os.getenv("AZURE_ENDPOINT"))  # EU region
response = client.chat.completions.create(
    model="deepseek-v4-flash", max_tokens=50,
    messages=[{"role": "user", "content": text_to_classify}]
)
```
~15x cheaper than Sonnet with European compliance. (DeployBase, 2026)

### 6.3 Budget alerts
```python
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

### 7.1 OTel in Claude Code (CLI)
```bash
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_METRICS_EXPORTER=otlp
export OTEL_LOGS_EXPORTER=otlp
export OTEL_TRACES_EXPORTER=otlp
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
export OTEL_EXPORTER_OTLP_PROTOCOL=grpc
export OTEL_METRIC_EXPORT_INTERVAL=10000
export OTEL_RESOURCE_ATTRIBUTES="team.id=platform,department=engineering"
```

### 7.2 OTel in Copilot Chat
```json
{ "github.copilot.chat.otel.enabled": true, "github.copilot.chat.otel.endpoint": "http://localhost:4317" }
```

### 7.3 Local stack in 30 seconds
```bash
docker run --rm -d -p 18888:18888 -p 4317:18889 --name aspire-dashboard \
  mcr.microsoft.com/dotnet/aspire-dashboard:latest
```
Open http://localhost:18888 — real-time traces.

### 7.4 Production stack (Grafana + Prometheus)
```yaml
version: '3.8'
services:
  otel-collector:
    image: otel/opentelemetry-collector-contrib:latest
    command: ["--config", "/etc/otel/config.yaml"]
    volumes: ["./otel-config.yaml:/etc/otel/config.yaml"]
    ports: ["4317:4317", "4318:4318"]
  prometheus:
    image: prom/prometheus:latest
    ports: ["9090:9090"]
    command: ["--storage.tsdb.retention.time=90d", "--config.file=/etc/prometheus/prometheus.yml"]
  grafana:
    image: grafana/grafana:latest
    ports: ["3000:3000"]
    environment: ["GF_SECURITY_ADMIN_PASSWORD=admin"]
```

### 7.5 Centralized configuration (MDM)
```json
{ "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1", "OTEL_METRICS_EXPORTER": "otlp",
    "OTEL_LOGS_EXPORTER": "otlp", "OTEL_EXPORTER_OTLP_PROTOCOL": "grpc",
    "OTEL_EXPORTER_OTLP_ENDPOINT": "http://collector.visma.internal:4317",
    "OTEL_EXPORTER_OTLP_HEADERS": "Authorization=Bearer <token>"
}}
```

---

## 8. Level 6 — Alternative lower-cost providers

Prices vary **625x** in the market. Most teams pay 4–30x more than necessary.

### Groq — LPU hardware, extreme speed
```python
from openai import OpenAI
client = OpenAI(api_key=os.getenv("GROQ_API_KEY"), base_url="https://api.groq.com/openai/v1")
response = client.chat.completions.create(model="llama-4-scout-17b-16e-instruct",
    messages=[{"role": "user", "content": text}], max_tokens=50)
```
Llama 4 ~$0.11/MTok. 4–10x cheaper than GPT-4o. Sub-100ms.

### Together AI — 100+ open-source models
```python
from openai import OpenAI
client = OpenAI(api_key=os.getenv("TOGETHER_API_KEY"), base_url="https://api.together.xyz/v1")
response = client.chat.completions.create(
    model="meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8",
    messages=messages, max_tokens=OUTPUT_BUDGETS.get(task_type, 500))
```

### Mistral — The only major European provider
```python
from openai import OpenAI
client = OpenAI(api_key=os.getenv("MISTRAL_API_KEY"), base_url="https://api.mistral.ai/v1")
response = client.chat.completions.create(model="mistral-small-latest",
    messages=messages, max_tokens=200)
```
Headquartered in France, native GDPR, EU data.

### Decision guide

| Task | Provider | Reason |
|------|----------|--------|
| Complex reasoning | Claude Opus / GPT-5 | Irreplaceable quality |
| General code work | Claude Sonnet | Best balance |
| Bulk classification | Groq (Llama 4) | 4–10x cheaper, sub-100ms |
| Offline batch | Together AI / Fireworks | Low price with SLA |
| EU data residency | Mistral | Only major EU-based provider |
| EU compliance + min cost | DeepSeek via Azure | Compliance + 15x cheaper |

Self-hosting with vLLM: only above $5K–10K/month and 70%+ GPU utilization.

---

## 9. Verified metrics reference

| Metric | Value | Source | URL |
|--------|-------|--------|-----|
| Re-sent context = % of bill | 62% | LeanOps 2026 | https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/ |
| Tokens turn 1 vs turn 50 | 5K → 200K | Redblink | https://redblink.com/ai-token-cost-optimization/ |
| Output vs input multiplier | 4–6x | Redis 2026 | https://redis.io/blog/llm-token-optimization-speed-up-apps/ |
| Cache reads discount | 90% | TokenOptimize.dev | https://www.tokenoptimize.dev/guides/llm-token-optimization-strategies |
| RAG app savings with caching | 88–95% | Finout | https://www.finout.io/blog/anthropic-api-pricing |
| VSCode prompt caching reuse | 93% | VS Magazine | https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx |
| RAG token reduction | up to 70% | Koombea | https://ai.koombea.com/blog/llm-cost-optimization |
| RouteLLM quality/calls | 95%/26% | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| RouteLLM savings vs baseline | 48% | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| RouteLLM with augmentation | 75% | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| VSCode tool search savings | up to 20% | VS Magazine | https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx |
| Copilot Auto Mode discount | 10% | GitHub Changelog | https://github.blog/changelog/2026-04-17-github-copilot-cli-now-supports-copilot-auto-model-selection/ |
| Copilot top users = spend | 10–15% = 60–70% | Synapx | https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/ |
| Agentic workflow cost | ~3.5x flat fee | Synapx | https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/ |
| Batch API discount | 50% | Anthropic official | https://www.finout.io/blog/anthropic-api-pricing |
| Batch + caching | up to 95% | PECollective | https://pecollective.com/tools/anthropic-api-pricing/ |
| Extended thinking omitted | No savings | CheckThat.ai | https://checkthat.ai/brands/anthropic/pricing |
| Thinking + output vs no thinking | 5x more expensive | PECollective | https://pecollective.com/tools/anthropic-api-pricing/ |
| Opus 4.7 tokenizer overhead | up to 35% more tokens | Finout | https://www.finout.io/blog/anthropic-api-pricing |
| Azure EA only | 15–25% | Microsoft Negotiations | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| Azure EA + MACC | 23–28% | Microsoft Negotiations | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| DeepSeek via Azure markup | +20–35% | DeployBase | https://deploybase.ai/articles/deepseek-v3-pricing |
| DeepSeek V4-Flash vs Sonnet | ~15x cheaper | 2026 prices | https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/ |
| AI % of total IT spend | up to 50% | Deloitte 2026 | https://redblink.com/ai-token-cost-optimization/ |
| Groq vs GPT-4o | 4–10x cheaper | ToolHalla 2026 | https://toolhalla.ai/blog/groq-vs-together-vs-fireworks-2026 |
| Self-hosting break-even | $5K–10K/month | morphllm.com | https://www.morphllm.com/llm-api |
