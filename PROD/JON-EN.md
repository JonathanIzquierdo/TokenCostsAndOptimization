# JON — Executive Index: AI Token Optimization

Summary of all topics covered in the full technical documentation (`technical-docs-draft-v1-EN.md`). No code, no deep dives. The goal is to understand what's here, why it matters, and how much it moves the bill.

---

## The core problem

**62% of the AI bill** is not for the work the model does. It's repeated context being sent over and over in every request. (LeanOps, audit of 30 teams, 2026)

**Output tokens cost 4–6x more** than input tokens. Nobody limits outputs.

An agentic session that starts at 5,000 tokens can reach **200,000 tokens per call by turn 50**. Without anyone noticing. Without anyone stopping it.

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

**DeepSeek via Azure** — ~$0.19/MTok input via Azure — **~15x cheaper than Sonnet**. The direct DeepSeek API is not viable for Visma (data routes through Chinese servers). Azure solves European compliance with a 20–35% markup.

**Microsoft EA negotiation** — Documented range: 15–25% with EA alone, **23–28% with EA + Azure MACC**. Not a published price — it's negotiated.

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
|-------|-----------------|------------|
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
