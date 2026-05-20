# Technical Guide: AI Token Optimization
## From basic settings to enterprise architecture

**Audience:** Developers, tech leads and architects
**Prerequisite:** Basic familiarity with LLMs, AI APIs, and development environments

---

## Index

1. [Fundamentals](#1-fundamentals)
2. [Where each optimization lever lives](#2-where-each-lever-lives)
3. [Direct vendor vs hyperscaler: the procurement decision](#3-procurement)
4. [Level 0 — Immediate settings](#4-level-0)
5. [Level 1 — Context architecture](#5-level-1)
6. [Level 2 — Model selection](#6-level-2)
7. [Level 3 — Automatic model routing](#7-level-3)
8. [Level 4 — Cost infrastructure and governance](#8-level-4)
9. [Level 5 — Observability with OpenTelemetry](#9-otel)
10. [Level 6 — Alternative lower-cost providers](#10-alternatives)
11. [Verified metrics reference](#11-metrics)

---

## Visual convention for snippets

Every block of application code carries a micro-header so you know where it lives and who touches it:

> 🔧 **Layer X** (physical location description) · **Technology** · **Who touches it**

For example:
> 🔧 **Layer 1** (application code) · Anthropic Python SDK · Application developers calling `api.anthropic.com` directly

Snippets without this header are **IDE settings** (VSCode, Copilot) — any individual dev can apply them, no custom app needed.

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
| GPT-5.4 | $2.50 | $15.00 | $0.25 | — |
| GPT-5.5 (flagship) | $5.00 | $30.00 | $0.50 | — |
| GPT-5 Nano | $0.05 | $0.40 | — | — |
| DeepSeek V4-Flash (direct) | $0.14 | $0.28 | — | — |
| DeepSeek V4-Flash (via Azure) | ~$0.19 | ~$0.38 | — | — |
| Llama 4 via Groq | ~$0.11 | ~$0.34 | — | — |

Sources: [Anthropic official May 2026](https://platform.claude.com/docs/en/about-claude/pricing), [OpenAI official](https://openai.com/api/pricing/), [Finout 2026](https://www.finout.io/blog/openai-pricing-in-2026), [DeployBase](https://deploybase.ai/articles/deepseek-v3-pricing), [ToolHalla](https://toolhalla.ai/blog/groq-vs-together-vs-fireworks-2026)

> **Opus 4.7 note:** Ships with a new tokenizer that can generate up to 35% more tokens for the same input text vs Opus 4.6. The per-token price didn't change, but effective cost per request can be up to 35% higher. Benchmark your workloads before migrating.
> Source: [Finout — Anthropic API Pricing 2026](https://www.finout.io/blog/anthropic-api-pricing)

### 1.3 The accumulated context problem

Turn 1: 5K tokens. Turn 10: ~15K. Turn 50: **200K tokens per call.**
**62% of the AI bill is re-sent context.** ([LeanOps, 30 teams, 2026](https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/))

### 1.4 Extended thinking: the invisible cost

> 🔧 **Layer 1** (application code) · Anthropic Python SDK · Application developers calling `api.anthropic.com/v1/messages` directly

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
500 output tokens + 2,000 thinking = **5x more expensive** than without thinking. ([PECollective, 2026](https://pecollective.com/tools/anthropic-api-pricing/))

---

## 2. Where each optimization lever lives <a id="2-where-each-lever-lives"></a>

This is the section worth reading **before** touching any configuration. Most teams lose time hunting for levers in the wrong place: they try to optimize from an IDE setting something that's only configurable in application code, or assume a code change will affect a user who only uses Claude.ai.

There are four physical places where token optimization gets touched. Each has a distinct audience inside the team, its own parameters, and its own ceiling of impact.

### 2.1 The four layers of control

| Layer | Physical location | Who touches it | Levers that live here | Savings ceiling |
|-------|---------------------|------------------|-------------------------|-------------------|
| **Layer 1** | Application code (Python/JS/etc) calling the API directly | Developers building internal LLM-powered apps | Prompt caching, thinking budgets, max_tokens, model per task, batch API, output budgets, beta headers | **Up to 95% combined** |
| **Layer 2** | IDE settings (VSCode, Copilot, Claude Code, Cursor) | Any individual dev, or admin via MDM | Auto Mode, compressOutput, tool search, debug panels, local OTel | **Up to 30% combined** |
| **Layer 3** | Billing console / platform admin | Tech lead, tenant admin | Allowed/blocked models, workspace budgets, alerts, audit logs | **Indirect: governance** |
| **Layer 4** | Gateway / routing layer (LiteLLM, Portkey, OpenRouter, RouteLLM) | Platform / infra team | Automatic model routing, fallbacks, retries, per-team budgets, guardrails | **20–80% per [IDC](https://www.infoworld.com/article/4164236/github-shifts-copilot-to-usage-based-billing)** |

### 2.2 The prior paradigm: closed product vs API key

Before any of the four layers, there's a binary decision that defines which layers you have access to at all:

**If you consume AI as a closed product** (Claude.ai, ChatGPT, Copilot, Cursor):
- You have access to Layer 2 (IDE settings) and a limited version of Layer 3 (product config)
- **You don't have access to Layer 1**: you can't touch caching, thinking, max_tokens, batch, or any call-level parameter
- **You don't have access to Layer 4**: routing is decided by the provider (Auto Mode in Copilot is the closest thing)
- Realistic total ceiling: ~30% savings over unoptimized consumption

**If you build with an API key** (Anthropic, OpenAI, etc., consumed from your own code):
- You have access to all four layers
- The big levers (caching 90%, batch 50%, routing 20–80%) live in Layer 1 and Layer 4
- Realistic total ceiling: 90%+ savings over unoptimized consumption

**This document assumes you're building, or moving toward it.** The layers that follow assume your team writes application code against an API.

### 2.3 How each snippet maps to its layer

To make it easy to visually locate any code block in its physical place:

| Snippet type | Layer | Who touches it |
|----------------|-------|----------------|
| `client.messages.create(...)`, SDK calls | Layer 1 | App developers |
| `client.messages.batches.create(...)` | Layer 1 | App developers |
| `cache_control: {type: "ephemeral"}` | Layer 1 | App developers |
| `thinking={"type": ...}` | Layer 1 | App developers |
| `OUTPUT_BUDGETS = {...}` | Layer 1 | App developers |
| `MODEL_ASSIGNMENT = {...}` | Layer 1 (in an internal wrapper) or Layer 4 (in a gateway) | Platform or developers |
| `chat.tools.compressOutput.enabled` in `settings.json` | Layer 2 | Any dev |
| `github.copilot.chat.agentDebugLog.enabled` | Layer 2 | Any dev |
| `export CLAUDE_CODE_ENABLE_TELEMETRY=1` | Layer 2 (local env vars) or Layer 3 (enterprise MDM) | Individual dev or admin |
| `Controller(routers=["mf"], ...)` (RouteLLM) | Layer 4 | Platform |
| `Router(model_list=[...], budget_manager=...)` (LiteLLM) | Layer 4 | Platform |
| `Portkey(api_key=..., virtual_key=...)` | Layer 4 | Platform |

### 2.4 How to govern Layer 1 at scale

There's an important operational note about Layer 1: if your team has 10 different microservices calling the Anthropic SDK directly, each one decides on its own thinking, caching, max_tokens, model, etc. You'll have 10 different levels of "optimization" and an unpredictable bill.

The two ways to govern it:

**Option A — Shared internal wrapper**: an internal library that wraps the SDK with sane defaults hardcoded (`adaptive thinking effort=low`, Haiku model unless explicit override, prompt caching active by default, max_tokens per task type). All services consume the wrapper, not the SDK directly. One decision, replicated everywhere.

**Option B — Shared gateway (LiteLLM/Portkey)**: calls go through a gateway that applies policy: "if nobody specified thinking, force low", "if nobody specified model, use Haiku", "if request cost exceeds $X, alert". Service code doesn't change — the gateway intercepts and normalizes.

For teams up to ~5 services, a wrapper is enough. Beyond that, a gateway starts to justify itself through multi-team budget control.

---

## 3. Direct vendor vs hyperscaler: the procurement decision <a id="3-procurement"></a>

This section documents the verified data worth having on the table when evaluating or negotiating an LLM consumption agreement with a vendor.

### 3.1 The price structure

When you consume an LLM with API key, there are three elements in total cost:

1. **Nominal price per token**: what's on the model's pricing page
2. **Request modifiers**: caching, batch, long context, fast mode, data residency
3. **Vendor overhead**: support plans, data transfer, fine-tuned model hosting, monitoring infra

The nominal price is where **most companies compare** and where the hyperscaler-with-discount appears to win. Overhead is where the math flips.

### 3.2 Nominal pricing parity across vendors (May 2026)

Anthropic maintains **per-token pricing parity** between its direct API and the major hyperscalers:

| Claude model | Anthropic direct | AWS Bedrock global | Vertex AI global | Microsoft Foundry |
|-----------------|---------------------|----------------------|--------------------|---------------------|
| Sonnet 4.6 | $3 / $15 | $3 / $15 | $3 / $15 | $3 / $15 |
| Opus 4.7 | $5 / $25 | $5 / $25 | $5 / $25 | $5 / $25 |
| Haiku 4.5 | $1 / $5 | $1 / $5 | $1 / $5 | $1 / $5 |

Source: [Anthropic Pricing Docs official](https://platform.claude.com/docs/en/about-claude/pricing), [Anthropic Claude in Microsoft Foundry](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry)

**Explicit multipliers published by Anthropic:**
- Bedrock regional endpoints: +10% over global
- Vertex regional/multi-region: +10% over global
- Direct Claude API with `inference_geo: "us"`: +10% (1.1x multiplier)

The same applies to OpenAI vs Azure OpenAI: identical per-token pricing.

### 3.3 Effective TCO: the real overhead

**Azure OpenAI vs OpenAI direct — 15–40% effective overhead, avg +22%**

Three independent 2026 sources converge:

| Source | Documented range | Direct quote |
|--------|---------------------|---------------|
| [TokenMix May 2026](https://tokenmix.ai/blog/azure-openai-cost) | 15–40%, avg 22% | "Token pricing identical. Total cost runs 15-40% higher on Azure due to support plans, data transfer, storage, and network infrastructure." |
| [Inference.net January 2026](https://inference.net/content/azure-openai-pricing-explained/) | 15–40% | "Total cost runs 15-40% higher on Azure due to support plans, data transfer, storage, and network infrastructure." |
| [CloudZero May 2026](https://www.cloudzero.com/blog/azure-openai-pricing/) | 20–40% | "Production deployments typically add 20-40% above listed token rates." |

Components of the Azure overhead:
- **Support plans**: $100–$1,000+/month (Standard support is de facto mandatory for production)
- **Data egress**: free first 100GB outbound, then $0.087/GB
- **Fine-tuned model hosting**: $1.70–$3/hour **even when idle** = $1,224–$2,160/month per model
- **VNet integration, Private Link, content filtering**: $200–$2,000/month depending on config
- **Log Analytics and monitoring infra**: variable

**AWS Bedrock for Claude — 20–35% effective overhead**

Nominal pricing identical to Anthropic direct, but ([TokenMix Bedrock 2026](https://tokenmix.ai/blog/aws-bedrock-pricing)):
- Regional endpoints: +10% (Anthropic official)
- Cross-region inference: +10% additional
- Bedrock counts data transfer (egress)
- Bedrock bills all HTTP 500 errors (the direct API has a 3% forgiveness buffer)
- Mandatory CloudWatch / CloudTrail logging

Direct quote: *"TokenMix.ai cost tracking shows enterprises running Claude on Bedrock pay an average of 20-35% more than those using Anthropic's direct API — and most do not realize it."*

**Microsoft Foundry for Claude — identical pricing but documented trap**

Claude is billed in Foundry as a third-party Marketplace item, not as a native Azure resource. Critical implication documented by [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/5851352) and [AZ365.ai March 2026](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/):

> "Claude models in Azure AI Foundry are billed as third-party Marketplace items, meaning their usage is typically not eligible for Azure sponsorship or startup credits... This often results in charges being applied directly to the credit card on file even if you have a significant remaining credit balance."

**Documented real cases** ([The Register / AZ365.ai](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/)):
- Japanese founder (Leach): ¥237,081 (~$1,600 USD) charged direct to credit card, credits did not apply
- German founder (Bogdan Sevriukov): €999.60 charged directly
- Another Japanese founder: ¥2,000,000 (~$13,000 USD) in one month
- Additional case reported by The Register: ~$3,000 USD

If your company is negotiating a Microsoft agreement assuming MACC absorbs Claude consumption via Foundry, **request explicit written clarification** on Claude credit eligibility.

### 3.4 Real enterprise discounts

| Discount type | Documented range | Source |
|-----------------|--------------------|---------|
| Azure EA alone | 15–25% negotiated | [Microsoft Negotiations](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing) |
| Azure EA + MACC | 23–28% negotiated | [Microsoft Negotiations](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing) |
| Anthropic Enterprise / volume | Available, custom | [CloudZero](https://www.cloudzero.com/blog/claude-pricing/) |
| OpenAI Enterprise tier | Custom pricing | [Finout, OpenAI](https://www.finout.io/blog/openai-pricing-in-2026) |
| AWS Bedrock private offers | Available | Anthropic docs |

### 3.5 The net calculation: discount vs overhead

Assuming baseline workload = 100 units of cost on direct vendor:

| Scenario | Direct cost | Azure cost | Bedrock cost | Net vs direct |
|----------|----------------|---------------|----------------|-------------------|
| GPT-5.4, no compliance, no discount | 100 | 115–140 | n/a | **+15% to +40%** |
| GPT-5.4, EA only (-20%) on 115–140 | 100 | 92–112 | n/a | **-8% to +12%** |
| GPT-5.4, EA + MACC (-25%) on 115–140 | 100 | 86–105 | n/a | **-14% to +5%** |
| Claude Sonnet, no compliance | 100 | n/a (Foundry no disc.) | 120–135 | **+20% to +35%** |
| DeepSeek vs Azure (no compliance) | 100 | 120–135 | n/a | **+20% to +35%** |
| DeepSeek vs Azure (with EU GDPR) | not viable | 120–135 | n/a | **compliance premium** |

**Conclusion:** the EA/MACC discount typically breaks even with but doesn't beat the overhead. The final bill lands between -14% and +12% vs the direct vendor. The real decision **is won or lost on compliance**, not on discount.

### 3.6 When the hyperscaler is worth it (decision matrix)

| Dominant criterion | Recommended path |
|----------------------|---------------------|
| Mature enterprise vendor's own model (Anthropic, OpenAI), no Azure-only mandate | **Direct vendor** — features first, better net price, factory support |
| Geopolitically sensitive model (DeepSeek, Qwen, Chinese models) | **Hyperscaler (Azure preferred)** — the premium buys compliance |
| Corporate mandate "everything on Azure/AWS" | **Hyperscaler** — absorb 15–40% overhead as governance cost |
| Need for MACC burn / unified Microsoft billing | **Hyperscaler** — accounting decision, **exclude Claude on Foundry** |
| Self-hosted open-source model (Llama, Mistral) | **Hyperscaler with managed endpoint or self-host** — Bedrock/Vertex or own GPUs |
| Team in exploration, prototypes, POCs | **Direct vendor** — fast onboarding, less bureaucracy |
| Mandatory GDPR / data residency compliance | **Hyperscaler** or **Mistral** (only major European direct option) |

### 3.7 Input for negotiation

Verified data to bring to the table with any vendor (Microsoft, AWS, Google, OpenAI, Anthropic):

| Datapoint | Value | Source | Use in negotiation |
|------|-------|--------|---------------------|
| Azure overhead vs OpenAI direct | 15–40% (avg 22%) | TokenMix, Inference.net, CloudZero | "We need a discount covering at least 25% just to break even against direct" |
| Bedrock overhead vs Anthropic direct | 20–35% effective | TokenMix Bedrock | "Foundry or direct gives us the same price without that markup" |
| Foundry Claude credits exclusion | Documented | Microsoft Q&A, AZ365.ai | "Explicit written clarification on which credits apply to Claude" |
| Anthropic Enterprise volume discount | Available | CloudZero | "Request terms comparable to hyperscaler with MACC" |
| OpenAI Enterprise custom pricing | Available | Finout | "Request the same regime as Azure but without the overhead" |
| Pricing parity Claude on hyperscalers | Confirmed | Anthropic docs | "The hyperscaler doesn't give us a better nominal price — only procurement" |

---

## 4. Level 0 — Immediate settings <a id="4-level-0"></a>

> 🔧 **Layer 2** (IDE settings) · All snippets in this section are VSCode `settings.json` JSON · Any individual dev can enable them

### 4.1 Auto Mode in GitHub Copilot
Switch the selector to "Auto" in VSCode. 10% automatic discount on the model multiplier. ([GitHub Changelog, April 2026](https://github.blog/changelog/2026-04-17-github-copilot-cli-now-supports-copilot-auto-model-selection/))

### 4.2 chat.tools.compressOutput.enabled
```json
{ "chat.tools.compressOutput.enabled": true }
```
Post-processes terminal output: collapses unchanged diff hunks, discards lockfile diffs, reduces `ls -l` to filenames, removes progress bars. VSCode 1.120.

### 4.3 Tool search — deferred MCPs
Default for Anthropic models (Sonnet 4.5+). For GPT models:
```json
{ "github.copilot.chat.responsesApi.toolSearchTool.enabled": true }
```
Up to 20% token savings per request. ([Visual Studio Magazine, 2026](https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx))

### 4.4 Agent Debug Log Panel — Full trace inside VSCode

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

### 4.5 Chat Debug View — Request-by-request inspection

Shows the detail of each individual request: full system prompt sent, user prompt, included context (files, snippets, history), and model response.

**How to open:** `(...)` menu → **"Show Chat Debug View"** or Command Palette → **"Developer: Show Chat Debug View"**

Source: [Medium — GitHub Copilot Token Usage Explained, Simform Engineering, May 2026](https://medium.com/simform-engineering/github-copilot-token-usage-explained-with-practical-cost-control-03062b15ecb0)

### 4.6 Token usage visibility for BYOK (VSCode 1.120)
Fixes token accounting for models with a personal API key — previously always showed 0%.

### 4.7 Audit active MCPs
10 MCPs × 500 tokens = 5,000 tokens overhead/request = **$1,500/month** on 100K requests to Sonnet without using any of them.

---

## 5. Level 1 — Context architecture <a id="5-level-1"></a>

### 5.1 Prompt caching

| Operation | Cost |
|-----------|------|
| Cache write TTL 5 min | 1.25x base |
| Cache write TTL 1 hour | 2.0x base |
| Cache read | 0.10x base (90% discount) |

> 🔧 **Layer 1** (application code) · Anthropic Python SDK · Application developers calling the API directly and designing the prompt shape

```python
response = client.messages.create(
    model="claude-sonnet-4-6", max_tokens=1024,
    system=[
        {"type": "text", "text": "You are an internal assistant...",
         "cache_control": {"type": "ephemeral"}},
        {"type": "text", "text": knowledge_base_content,
         "cache_control": {"type": "ephemeral"}}
    ],
    messages=[{"role": "user", "content": user_message}]
)
print(f"Cache read: {response.usage.cache_read_input_tokens}")
```
RAG app 50K tokens × 1,000 queries/day → **84% savings** on that portion. ([Finout, 2026](https://www.finout.io/blog/anthropic-api-pricing))

### 5.2 History management

> 🔧 **Layer 1** (application code) · Python helpers on top of the SDK · Application developers designing agentic session management

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

### 5.3 RAG and output budgets

> 🔧 **Layer 1** (application code) · RAG pipeline + budget dictionary · Application developers designing retrieval flow and per-task caps

```python
chunks = vector_store.search(query=user_query, top_k=5)
context = "\n\n".join([c.text[:500] for c in chunks])
OUTPUT_BUDGETS = {
    "classification": 50, "extraction": 200, "summarization": 500,
    "code_review": 1000, "code_generation": 2000, "architecture": 3000
}
```
RAG reduces context tokens up to 70% vs context stuffing. ([Koombea, 2026](https://ai.koombea.com/blog/llm-cost-optimization))

---

## 6. Level 2 — Model selection and assignment <a id="6-level-2"></a>

> 🔧 **Layer 1** (application code) or **Layer 4** (in a gateway if centralized) · Assignment dictionary + classifier router · Application developers, or platform if moved to a shared gateway

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

## 7. Level 3 — Automatic model routing <a id="7-level-3"></a>

### 7.1 RouteLLM (ICLR 2025, UC Berkeley/LMSYS)
95% GPT-4 quality with 26% calls → 48% savings. With augmentation: **75% reduction**. ([LMSYS ICLR 2025](https://www.lmsys.org/blog/2024-07-01-routellm/))

> 🔧 **Layer 4** (gateway / routing layer) · UC Berkeley/LMSYS open source · Platform or infra team placing a layer between services and providers

```python
from routellm.controller import Controller
client = Controller(
    routers=["mf"], strong_model="anthropic/claude-opus-4-7",
    weak_model="anthropic/claude-haiku-4-5", config={"mf": {"threshold": 0.3}}
)
response = client.chat.completions.create(model="router", messages=[...])
```

### 7.2 Enterprise AI Gateways

> 🔧 **Layer 4** (shared gateway) · LiteLLM self-hosted / Portkey managed · Platform or infra team. Consumer service code does NOT change — the gateway intercepts

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

## 8. Level 4 — Cost infrastructure and governance <a id="8-level-4"></a>

### 8.1 Batch API (50% discount, identical quality)

> 🔧 **Layer 1** (application code) · Anthropic Batches API SDK · Application developers processing non-realtime jobs (ETL, evaluations, bulk generation)

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
With caching: up to **95% savings**. ([PECollective, 2026](https://pecollective.com/tools/anthropic-api-pricing/))

### 8.2 DeepSeek via Azure

> 🔧 **Layer 1** (application code) · OpenAI SDK pointing to Azure endpoint · Application developers processing bulk cheap tasks with EU compliance

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

**Procurement context:** ~15x cheaper than Sonnet on nominal price, with European compliance secured. Direct DeepSeek access (`api.deepseek.com`) is not viable for companies with EU data because it routes all requests through Chinese servers. Azure adds a markup of **20–35% over direct** ([DeployBase, 2026](https://deploybase.ai/articles/deepseek-v3-pricing)), but that markup is a **compliance premium, not overhead**: data stays in the Azure region you choose, the contract is with Microsoft, and European compliance is covered. Even with that 20–35% on top, DeepSeek via Azure still costs ~1/15 of what Sonnet costs, so for bulk classification, structured extraction, or offline batch processing, it remains the best price/compliance ratio on the market.

**Critical note:** this is one of the few cases where the hyperscaler is the right choice for model-specific reasons, not for general procurement reasons. For Claude or GPT, the same calculation does not apply (see section 3 — nominal pricing parity).

### 8.3 Observability and alerts

> 🔧 **Layer 1** (application code) or **Layer 4** (centralized gateway) · Python helper on top of the SDK or gateway middleware · App developers or platform

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

## 9. Level 5 — Observability with OpenTelemetry <a id="9-otel"></a>

### 9.1 OTel in Claude Code (CLI)

> 🔧 **Layer 2** (local env vars) or **Layer 3** (enterprise MDM) · Shell environment variables · Individual developers or admin via managed settings

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

### 9.2 OTel in Copilot Chat
```json
{ "github.copilot.chat.otel.enabled": true, "github.copilot.chat.otel.endpoint": "http://localhost:4317" }
```

### 9.3 Local stack in 30 seconds

> 🔧 **Layer 2** (developer workstation) · Aspire Dashboard Docker container · Individual dev for local use, no cloud account needed

```bash
docker run --rm -d -p 18888:18888 -p 4317:18889 --name aspire-dashboard \
  mcr.microsoft.com/dotnet/aspire-dashboard:latest
```
Open http://localhost:18888 — real-time traces.

### 9.4 Production stack (Grafana + Prometheus)

> 🔧 **Layer 3** (observability infra) · docker-compose for OTel + Prometheus + Grafana stack · Infra/SRE team

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

### 9.5 Centralized configuration (MDM)

> 🔧 **Layer 3** (enterprise managed settings) · JSON distributed via MDM (Intune, Jamf) · Developer platform admin

```json
{ "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1", "OTEL_METRICS_EXPORTER": "otlp",
    "OTEL_LOGS_EXPORTER": "otlp", "OTEL_EXPORTER_OTLP_PROTOCOL": "grpc",
    "OTEL_EXPORTER_OTLP_ENDPOINT": "http://collector.internal:4317",
    "OTEL_EXPORTER_OTLP_HEADERS": "Authorization=Bearer <token>"
}}
```

---

## 10. Level 6 — Alternative lower-cost providers <a id="10-alternatives"></a>

Prices vary **625x** in the market. Most teams pay 4–30x more than necessary.

### Groq — LPU hardware, extreme speed

> 🔧 **Layer 1** (application code) · OpenAI SDK with alternative base_url · Application developers needing sub-100ms latency on simple tasks

```python
from openai import OpenAI
client = OpenAI(api_key=os.getenv("GROQ_API_KEY"), base_url="https://api.groq.com/openai/v1")
response = client.chat.completions.create(model="llama-4-scout-17b-16e-instruct",
    messages=[{"role": "user", "content": text}], max_tokens=50)
```
Llama 4 ~$0.11/MTok. 4–10x cheaper than GPT-4o. Sub-100ms.

### Together AI — 100+ open-source models

> 🔧 **Layer 1** (application code) · OpenAI SDK with alternative base_url · Application developers needing high-volume batch with fine-tuning

```python
from openai import OpenAI
client = OpenAI(api_key=os.getenv("TOGETHER_API_KEY"), base_url="https://api.together.xyz/v1")
response = client.chat.completions.create(
    model="meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8",
    messages=messages, max_tokens=OUTPUT_BUDGETS.get(task_type, 500))
```

### Mistral — The only major European provider

> 🔧 **Layer 1** (application code) · OpenAI SDK with alternative base_url · Application developers needing native GDPR without going through a hyperscaler

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

## 11. Verified metrics reference <a id="11-metrics"></a>

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
| **Azure OpenAI overhead vs OpenAI direct** | **15–40% (avg 22%)** | **TokenMix** | **https://tokenmix.ai/blog/azure-openai-cost** |
| **Azure OpenAI overhead vs OpenAI direct** | **15–40%** | **Inference.net** | **https://inference.net/content/azure-openai-pricing-explained/** |
| **Azure OpenAI overhead vs OpenAI direct** | **20–40%** | **CloudZero** | **https://www.cloudzero.com/blog/azure-openai-pricing/** |
| **Bedrock Claude overhead vs Anthropic direct** | **20–35% effective** | **TokenMix Bedrock** | **https://tokenmix.ai/blog/aws-bedrock-pricing** |
| **Bedrock regional endpoints premium** | **+10% over global** | **Anthropic docs official** | **https://platform.claude.com/docs/en/about-claude/pricing** |
| **Foundry Claude credits exclusion** | **Azure credits don't apply** | **Microsoft Q&A** | **https://learn.microsoft.com/en-us/answers/questions/5851352** |
| **Documented Foundry trap case** | **~$13K USD/month one startup** | **AZ365.ai March 2026** | **https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/** |
| **Pricing parity Claude Bedrock/Vertex/Foundry** | **Identical to direct** | **Anthropic docs official** | **https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry** |
| DeepSeek via Azure markup | +20–35% | DeployBase | https://deploybase.ai/articles/deepseek-v3-pricing |
| DeepSeek V4-Flash vs Sonnet | ~15x cheaper | 2026 prices | https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/ |
| AI % of total IT spend | up to 50% | Deloitte 2026 | https://redblink.com/ai-token-cost-optimization/ |
| Groq vs GPT-4o | 4–10x cheaper | ToolHalla 2026 | https://toolhalla.ai/blog/groq-vs-together-vs-fireworks-2026 |
| Self-hosting break-even | $5K–10K/month | morphllm.com | https://www.morphllm.com/llm-api |
| Anthropic official pricing May 2026 | Sonnet $3/$15, Opus $5/$25, Haiku $1/$5 | Anthropic | https://platform.claude.com/docs/en/about-claude/pricing |
| OpenAI official pricing 2026 | GPT-5.4 $2.50/$15, GPT-5.5 $5/$30, Nano $0.05/$0.40 | OpenAI / Finout | https://openai.com/api/pricing/ |
