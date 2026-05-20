# JON — Executive Index: AI Token Optimization

Summary of all topics covered in the full technical documentation (`technical-docs-draft-v1-EN.md`). No code, no deep dives. The goal is to understand what's here, why it matters, and how much it moves the bill.

---

## The core problem

**62% of the AI bill** is not for the work the model does. It's repeated context being sent over and over in every request. (LeanOps, audit of 30 teams, 2026)

**Output tokens cost 4–6x more** than input tokens. Nobody limits outputs.

An agentic session that starts at 5,000 tokens can reach **200,000 tokens per call by turn 50**. Without anyone noticing. Without anyone stopping it.

---

## The paradigm shift: closed product vs building with API

Before getting into the optimization levers, there's a foundational decision that determines which levers exist for you at all: **what relationship are we in with the model.**

### Closed product vs API key

**As users of a closed product** (Claude.ai, ChatGPT, Copilot, Cursor): the provider controls all parameters. You pay the subscription, you consume. The optimization levers available are limited but real:
- Pick the model from the dropdown when the product allows it
- Enable Auto Mode in Copilot (10% discount)
- Configure `compressOutput` and tool search in VSCode (up to 20% savings)
- Audit which MCPs are active
- Inspect consumption with Agent Debug Log and Chat Debug View

They work, but the ceiling is low. You can't touch thinking budgets, you can't enable prompt caching, you can't choose batch processing, you can't route to Haiku from Sonnet, you can't set `max_tokens`.

**As builders with an API key** (Anthropic, OpenAI, DeepSeek, etc., consumed from your own code): all the levers are yours. The 50%, 90%, and 95% savings that appear throughout this document live here:
- Prompt caching: up to **90% discount** on cached content
- Batch API: **50% discount** on input and output, stacks with caching → up to **95%**
- Explicit output budgets per task (5–25x fewer tokens on expensive outputs)
- Extended thinking controlled or disabled (avoids the 5x multiplier)
- Routing across models (up to 25x difference between Haiku and Opus)
- Gateways with per-team budget controls (LiteLLM, Portkey, OpenRouter)

**This document assumes you're building, or moving toward it.** The layers that follow are levers actionable from application code, IDE settings, or routing layers — not from the desktop app.

---

## Direct vendor vs hyperscaler: the procurement decision

Once you've decided to build with API, the next question is **where to buy it**: direct vendor (Anthropic, OpenAI, DeepSeek) or via hyperscaler (Azure OpenAI / Microsoft Foundry, AWS Bedrock, Google Vertex AI).

The common intuition — "hyperscaler is cheaper because of the EA discount" — is **frequently false** or barely breaks even. Verified data, all from 2026 sources:

### Nominal prices per model (May 2026)

| Model | Direct vendor | Via hyperscaler | Nominal difference |
|-------|---------------|------------------|---------------------|
| Claude Sonnet 4.6 | $3 / $15 per MTok ([Anthropic](https://platform.claude.com/docs/en/about-claude/pricing)) | Same price on Bedrock global, Vertex global, Foundry ([Anthropic docs](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry)) | 0% nominal |
| Claude Sonnet 4.6 (regional EU) | $3 / $15 + 10% data residency | $3 / $15 + 10% regional endpoint | Same overhead |
| Claude Opus 4.7 | $5 / $25 per MTok | Same across hyperscalers | 0% nominal |
| Claude Haiku 4.5 | $1 / $5 per MTok | Same across hyperscalers | 0% nominal |
| GPT-5.4 | $2.50 / $15 ([OpenAI](https://openai.com/api/pricing/)) | Azure OpenAI same per-token | 0% nominal |
| GPT-5.5 (flagship) | $5 / $30 ([Finout](https://www.finout.io/blog/openai-pricing-in-2026)) | Azure OpenAI same per-token | 0% nominal |
| DeepSeek V4-Flash | $0.14 / $0.28 ([TLDL](https://www.tldl.io/resources/anthropic-api-pricing)) | Via Azure: ~$0.19 / $0.38 | +20–35% ([DeployBase](https://deploybase.ai/articles/deepseek-v3-pricing)) |

**Reading:** for Claude and GPT, the nominal per-token price is **identical** between direct vendor and hyperscaler. For DeepSeek there's a direct 20–35% markup via Azure (Azure charges a premium for hosting a non-Microsoft model).

### Effective TCO: the overhead trap

The problem isn't nominal pricing — it's what accumulates around it:

**Azure OpenAI:** effective TCO **15–40% above OpenAI direct**. Typical customer average: **+22%**. Sources:
- [TokenMix, May 2026](https://tokenmix.ai/blog/azure-openai-cost): "Token pricing identical. Total cost runs 15-40% higher on Azure due to support plans, data transfer, storage, and network infrastructure."
- [Inference.net, January 2026](https://inference.net/content/azure-openai-pricing-explained/): same figure confirmed with per-component breakdown.
- [CloudZero, May 2026](https://www.cloudzero.com/blog/azure-openai-pricing/): "production deployments typically add 20–40% above listed token rates"

Where does that overhead come from?
- Support plans: $100–$1,000+/month (mandatory for production)
- Data egress: $0.087/GB after the first 100GB
- Fine-tuned model hosting: $1.70–$3/hour **even when idle** = $1,224–$2,160/month
- VNet integration, Private Link, content filtering: $200–$2,000/month
- Log Analytics and monitoring infra

**AWS Bedrock for Claude:** identical per-token price to direct, **but effective TCO 20–35% higher on average**. ([TokenMix Bedrock 2026](https://tokenmix.ai/blog/aws-bedrock-pricing)) Causes:
- Regional/multi-region endpoints: +10% over global (Anthropic official)
- Cross-region inference: +10% additional
- Data transfer fees (Bedrock counts egress)
- Bedrock bills all HTTP 500 errors (direct API has a 3% forgiveness buffer)
- Mandatory CloudWatch / CloudTrail logging

**Microsoft Foundry for Claude:** identical per-token price to direct. **Documented trap**: Azure / Founders Hub / MACC credits **do NOT apply** to Claude on Foundry — it bills as third-party Marketplace directly to credit card. ([AZ365.ai March 2026](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/), [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/5851352)). Documented case: a startup accumulated ~$13,000 USD in a single month thinking their Foundry credits would cover Claude. They didn't.

### EA / MACC discounts

- **Azure EA only:** 15–25% negotiated. ([Microsoft Negotiations](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing))
- **Azure EA + MACC:** 23–28%. (same source)
- **Anthropic Enterprise / volume discounts:** available, directly negotiated.
- **OpenAI Enterprise tier:** custom pricing, negotiated.

### The net calculation for an informed decision

| Scenario | Direct cost | Azure cost | Bedrock cost | Net vs direct |
|----------|-------------|-------------|---------------|----------------|
| GPT-5.4, no compliance, no discount | 100 | 115–140 | n/a | **+15% to +40%** |
| GPT-5.4, EA only (-20%) | 100 | 92–112 | n/a | **-8% to +12%** |
| GPT-5.4, EA + MACC (-25%) | 100 | 86–105 | n/a | **-14% to +5%** |
| Claude Sonnet, no compliance | 100 | n/a (Foundry no disc.) | 120–135 | **+20% to +35% Bedrock** |
| DeepSeek vs Azure (no compliance) | 100 | 120–135 | n/a | **+20% to +35%** |
| DeepSeek vs Azure (with EU GDPR) | not viable | 120–135 | n/a | **compliance premium** |

**Actionable conclusion**: the EA/MACC discount typically **breaks even with but does not beat** Azure overhead. The final bill lands between -14% and +12% vs the direct vendor. The real decision **isn't won on discount — it's won or lost on compliance and corporate mandate**.

### When the hyperscaler is actually worth it

1. **Mandatory GDPR / data residency**: the premium buys compliance that direct doesn't offer. Essential when handling EU customer data.
2. **Corporate mandate "everything on Azure/AWS"**: the overhead is absorbed as a unified-governance cost.
3. **Geopolitically sensitive model**: the DeepSeek-via-Azure case. Direct DeepSeek access isn't viable under GDPR (data routes through Chinese servers). Azure resolves European compliance even with +20–35% markup — and even so, DeepSeek via Azure is still ~15x cheaper than Sonnet for high-volume tasks. ([DeployBase](https://deploybase.ai/articles/deepseek-v3-pricing))
4. **MACC burn**: if the organization has already committed annual Microsoft spend, consuming Foundry/Azure OpenAI applies against that commitment (except Claude on Foundry — see trap above).

**If none of those four apply**, direct vendor wins on net price and on feature velocity — betas and new capabilities ship first on the vendor's own API.

### Input for vendor negotiations

Verified data worth having on the table for in-flight agreements:

| Datapoint | Value | Source | Use in negotiation |
|-----------|-------|--------|---------------------|
| Azure overhead vs OpenAI direct | 15–40% (avg 22%) | TokenMix, Inference.net, CloudZero | "We need a discount covering at least 25% just to break even" |
| Bedrock overhead vs Anthropic direct | 20–35% effective | TokenMix Bedrock | "Foundry or direct gives us the same price without that markup" |
| Foundry Claude credits exclusion | Documented | Microsoft Q&A, AZ365.ai | "Explicit contract clause on which credits apply to Claude usage" |
| Anthropic Enterprise volume discount | Available | CloudZero, Anthropic sales | "Request terms comparable to the hyperscaler with MACC" |
| OpenAI Enterprise custom pricing | Available | Finout, OpenAI | "Request same regime as Azure but without the overhead" |

---

## Level 0 — Immediate settings (5 minutes, no code)

**Auto Mode in Copilot** — Switch the model selector to "Auto" in VSCode. Copilot dynamically picks the most efficient model per task. 10% automatic discount on the model multiplier.

**chat.tools.compressOutput.enabled** — One line in `settings.json`. VSCode compresses terminal output before sending it to the model: collapses unchanged diff hunks, discards lockfile diffs, removes progress bars. Significantly reduces context tokens in sessions with frequent terminal operations.

**Tool search (deferred MCPs)** — Active by default for Anthropic models. Splits tools into core (~30, always loaded) and deferred (loaded on demand). Up to 20% token savings per request. 10 active MCPs you never use = potentially $1,500/month in pure overhead on 100K requests.

**Agent Debug Log panel** — The fastest way to see what's happening without configuring any external backend. Shows the complete session trace: total tokens, tool calls, model turns, errors, and a visual flow chart of the agent. Open it from the `(...)` overflow menu in the Copilot Chat panel → "Show Agent Debug Logs". Requires enabling `github.copilot.chat.agentDebugLog.enabled`. To persist past sessions to disk: `github.copilot.chat.agentDebugLog.fileLogging.enabled`.

**Chat Debug View** — For request-by-request inspection: shows the full system prompt, the context sent, and the model response. Open from the same `(...)` menu → "Show Chat Debug View".

---

## Level 1 — Context architecture

**Prompt caching** — Static system content (instructions, tool definitions, knowledge base) gets cached. Cache reads cost 10% of the normal price — **90% discount**. A RAG app with 50,000 tokens in the system prompt, queried 1,000 times a day, saves up to 84% of that cost just by enabling caching. Break-even: a single cache hit.

**Conversation history management** — Three approaches, from simple to sophisticated:
- Sliding window: keep only the last N turns
- Progressive summarization: compress old history (using Haiku, the cheapest model)
- Anthropic Compaction API (beta January 2026): automatically summarizes when approaching the limit, with full semantic understanding, eligible for Zero Data Retention

**RAG instead of context stuffing** — Retrieve only the relevant fragments for the current query, not entire documents. Can reduce context tokens by up to **70%**.

**Output budgets per task** — Set `max_tokens` explicitly per task type. Classification: 50 tokens. Summary: 500. Code generation: 2,000. Without this, the model generates freely and outputs (5x more expensive) spiral out of control.

---

## Level 2 — Correct model selection

The price difference between Haiku and Opus is **25x**. Most workflows use Sonnet or Opus for tasks that Haiku handles just as well.

| Model | Best for | Input price |
|-------|----------|-------------|
| Haiku 4.5 | Classification, extraction, formatting, file navigation | $1/MTok |
| Sonnet 4.6 | General code, chat, summarization, debugging | $3/MTok |
| Opus 4.7 | Complex reasoning, architecture, agent coordination | $5/MTok |

**Important note:** Opus 4.7 ships with a new tokenizer that can generate up to 35% more tokens for the same input text vs Opus 4.6. The per-token price didn't change, but the effective cost per request can be up to 35% higher. Benchmark your workloads before migrating from Opus 4.6.

**Extended thinking** — Internal reasoning tokens are billed as output tokens. Using `display: omitted` does NOT reduce the cost — it only hides the reasoning from the user. 500 visible output tokens + 2,000 thinking tokens = **5x more expensive** than without thinking. Reserve for tasks that genuinely require it.

---

## Level 3 — Automatic model routing

**Auto Mode Copilot + OpenRouter** — The simplest level. Zero setup. Auto Mode gives 10% discount. OpenRouter routes across 300+ models powered by NotDiamond, no markup.

**RouteLLM** (open source, UC Berkeley/LMSYS, ICLR 2025) — 95% of GPT-4 quality using only 26% of calls — 48% cheaper than baseline. With data augmentation: **75% cost reduction**. Drop-in replacement for the OpenAI client.

**Enterprise AI Gateways** (LiteLLM, Portkey, OpenRouter) — The most important argument isn't routing — it's protection: without a gateway, a single runaway agent can burn the monthly budget overnight.

---

## Level 4 — Infrastructure levers

**Batch API** — 50% discount on input and output, identical quality. Combined with prompt caching: up to **95% savings**. Ideal for bulk classification, test generation, nightly analysis.

**DeepSeek via Azure** — ~$0.19/MTok input via Azure — **~15x cheaper than Sonnet**. The direct DeepSeek API is not viable with EU data (it routes through Chinese servers). Azure resolves European compliance with a 20–35% markup over direct — you're paying for compliance, not discount, and even so it's 15x cheaper than Sonnet.

**Microsoft EA negotiation** — Documented range: 15–25% with EA alone, **23–28% with EA + Azure MACC**. Not a published price — it's negotiated. Important: as shown in the procurement section, this discount typically breaks even but doesn't beat Azure overhead. The decision is won or lost on compliance, not on discount.

---

## Level 5 — Observability with OpenTelemetry

For team-level governance: who consumes the most, which workflow is the most expensive, what's the actual cache hit rate.

**Claude Code has native OTel** — One environment variable turns it on: `CLAUDE_CODE_ENABLE_TELEMETRY=1`. Exports tokens, latency, and tool calls to any OTLP backend.

**Copilot Chat also has OTel** — Enabled from VSCode settings. Produces a complete span hierarchy.

**Local stack in 30 seconds** — Microsoft Aspire Dashboard in Docker. No cloud account needed.

**Centralized configuration** — Via managed settings distributable through MDM. Every developer automatically instrumented.

Compatible backends: Grafana, Jaeger, Datadog, Honeycomb, Langfuse, SigNoz, Azure Monitor.

---

## Level 6 — Alternative lower-cost providers

The inference API market in 2026 has 11+ production providers. Prices vary **625x** between the cheapest and the most expensive.

**Groq** — Custom silicon (LPU). Llama 4 at ~$0.11/MTok input — 4–10x cheaper than GPT-4o. Sub-100ms latency.

**Together AI** — 100+ open-source models, fine-tuning available. Ideal for high-volume batch workloads.

**Fireworks AI** — Production-grade with formal SLAs.

**Mistral** — The only major provider headquartered in Europe. Native GDPR compliance. Most direct option for EU data residency without routing through Azure.

**Google Gemini Flash** — $0.075/MTok input. Cheapest high-quality model from a major provider. 1M token context window.

**Self-hosting with vLLM** — Only makes sense above $5K–10K/month and with 70%+ sustained GPU utilization.

---

## Key metrics summary

| Lever | Savings / Impact | Complexity |
|-------|------------------|------------|
| Copilot Auto Mode | 10% discount | ⭐ Immediate |
| compressOutput + tool search | up to 20% tokens | ⭐ Immediate |
| Agent Debug Log + Chat Debug View | Instant visibility, no setup | ⭐ Immediate |
| Prompt caching | 90% on cached tokens | ⭐⭐ Low |
| RAG vs context stuffing | up to 70% reduction | ⭐⭐ Low |
| Correct model selection | up to 25x difference | ⭐⭐ Low |
| Batch API | 50% discount | ⭐⭐ Low |
| Batch + caching combined | up to 95% savings | ⭐⭐ Low |
| RouteLLM | 75% reduction in strong model calls | ⭐⭐⭐ Medium |
| AI Gateway (LiteLLM/Portkey) | Budget controls + protection | ⭐⭐⭐ Medium |
| OTel observability | Full team-level visibility | ⭐⭐⭐ Medium |
| DeepSeek via Azure | ~15x cheaper than Sonnet | ⭐⭐⭐ Medium |
| Groq/Together/Mistral | 4–30x cheaper than frontier | ⭐⭐⭐⭐ High |
| Self-hosting vLLM | 100% token savings (own infra) | ⭐⭐⭐⭐⭐ High |

---

*Full documentation with code, examples, and implementation details: `PROD/technical-docs-draft-v1-EN.md`*
