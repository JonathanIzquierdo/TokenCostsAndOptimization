# JON — Executive Index: AI Token Optimization

Summary of all topics covered in the full technical documentation (`technical-docs-draft-v1-EN.md`). No code, no deep dives. The goal is to understand what's here, why it matters, and how much it moves the bill.

---

## The core problem

**62% of the AI bill** is not for the work the model does. It's repeated context being sent over and over in every request. (LeanOps, audit of 30 teams, 2026)

**Output tokens cost 4–6x more** than input tokens. Nobody limits outputs.

An agentic session that starts at 5,000 tokens can reach **200,000 tokens per call by turn 50**. Without anyone noticing. Without anyone stopping it.

---

## Level 0 — Immediate settings (5 minutes, no code)

**Auto Mode in Copilot** — Switch the model selector to "Auto" in VSCode. Copilot dynamically picks the most efficient model per task. 10% automatic discount on the model multiplier. Zero additional configuration.

**chat.tools.compressOutput.enabled** — One line in `settings.json`. VSCode compresses terminal output before sending it to the model: collapses unchanged diff hunks, discards lockfile diffs, removes progress bars. Significantly reduces context tokens in sessions with frequent terminal operations.

**Tool search (deferred MCPs)** — Active by default for Anthropic models. Splits tools into core (~30, always loaded) and deferred (loaded on demand). Up to 20% token savings per request. 10 active MCPs you never use = potentially $1,500/month in pure overhead on 100K requests.

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

The map isn't rigid, but the question is always the same: does this task actually need deep reasoning, or is it pattern matching?

**Extended thinking** — Internal reasoning tokens are billed as output tokens. Using `display: omitted` does NOT reduce the cost — it only hides the reasoning from the user. 500 visible output tokens + 2,000 thinking tokens = **5x more expensive** than without thinking. Reserve for tasks that genuinely require it.

---

## Level 3 — Automatic model routing

Instead of manually deciding which model to use, a system decides per request.

**Auto Mode Copilot + OpenRouter** — The simplest level. Zero setup. Auto Mode gives 10% discount. OpenRouter routes across 300+ models powered by NotDiamond, no markup.

**RouteLLM** (open source, UC Berkeley/LMSYS, ICLR 2025) — Uses ML to predict which model will perform best per query. The matrix factorization router achieves **95% of GPT-4 quality using only 26% of GPT-4 calls** — 48% cheaper than baseline. With data augmentation: **75% cost reduction**. Drop-in replacement for the OpenAI client — no need to rewrite the application.

**Enterprise AI Gateways** (LiteLLM, Portkey, OpenRouter) — Layers between the application and providers. The most important argument isn't routing — it's protection: without a gateway, a single runaway agent can burn the monthly budget overnight.

- LiteLLM: open source, self-hosted, budget controls per team or API key
- Portkey: managed, 1,600+ models, guardrails, PII filtering, SOC2
- OpenRouter: zero setup, 300+ models, 5.5% markup

---

## Level 4 — Infrastructure levers

**Batch API** — For workloads that don't need real-time responses. **50% discount** on input and output, identical quality. Combined with prompt caching: up to **95% savings** vs a standard request. Ideal for: bulk classification, test generation, nightly analysis, model evaluations.

**DeepSeek via Azure** — DeepSeek V4-Flash is ~$0.19/MTok input via Azure — **~15x cheaper than Sonnet** for high-volume tasks. The direct DeepSeek API is not viable for Visma (data routes through Chinese servers). Azure solves the European compliance requirement with a 20–35% markup over the direct price — which is still dramatically cheaper than frontier models.

**Microsoft EA negotiation** — The Copilot discount via Enterprise Agreement is not fixed — it's negotiated. Documented range: 15–25% with EA alone, **23–28% with EA + Azure MACC**. Requires framing the conversation in the context of the renewal, not negotiating Copilot in isolation.

---

## Level 5 — Observability with OpenTelemetry

Without instrumentation, you're flying blind. The billing shows you the monthly total — not who generated it, what workflow caused it, or how it evolves over time.

**Claude Code has native OTel** — Disabled by default. One environment variable turns it on: `CLAUDE_CODE_ENABLE_TELEMETRY=1`. From there it exports tokens (input, output, cache read, cache write), latency, and every tool call to any OTLP backend.

**Copilot Chat also has OTel** — Enabled from VSCode settings. Produces a complete span hierarchy: LLM requests, tool calls, subagents, duration, tokens.

**Local stack in 30 seconds** — Microsoft's Aspire Dashboard runs in Docker and provides a full trace viewer with no cloud account needed.

**Production stack** — OTel Collector + Prometheus + Grafana via docker-compose. Claude Code and Copilot tokens feed the same pipeline as the rest of your infra.

**Centralized configuration** — Via managed settings distributable through MDM, every developer on the team gets automatically instrumented without individual setup.

**Available filtering attributes:** model used, provider, tool executed, team, department, user. They answer: who consumes the most? which workflow is the most expensive? what's the actual cache hit rate?

Compatible backends (all speak OTLP, no lock-in): Grafana, Jaeger, Datadog, Honeycomb, Langfuse, SigNoz, Azure Monitor.

---

## Level 6 — Alternative lower-cost providers

The inference API market in 2026 has 11+ production providers. Prices vary **625x** between the cheapest and the most expensive. Most teams pay 4 to 30x more than necessary for many tasks.

**Groq** — Custom silicon (LPU) built for transformer inference. 500–800 tokens/second. Llama 4 at ~$0.11/MTok input — 4–10x cheaper than GPT-4o. Ideal for latency-sensitive simple tasks. Not for complex reasoning.

**Together AI** — 100+ open-source models, fine-tuning available. Prices from $0.14/MTok. Ideal for high-volume batch workloads and cases where fine-tuning on proprietary data is needed.

**Fireworks AI** — Production-grade with formal SLAs, optimized function calling. For production that needs formal availability guarantees.

**Mistral** — The only major provider headquartered in Europe. Native GDPR compliance, pricing below OpenAI, open-weight models available for self-hosting. For Visma, the most direct option when EU data residency is required without routing through Azure.

**Google Gemini Flash** — $0.075/MTok input — the cheapest high-quality model from a major provider in 2026. 1M token context window. Worth evaluating for workloads with very long contexts.

**Self-hosting with vLLM** — Eliminates per-token cost entirely. Only makes sense above $5K–10K/month in API spend and with 70%+ sustained GPU utilization. Below those thresholds, API providers win on cost.

**The strategy isn't to migrate everything** — it's routing by task type. Complex reasoning: Claude/GPT-5. Bulk classification: Groq. Offline batch: Together AI. EU data residency: Mistral. European compliance + minimum cost: DeepSeek via Azure.

---

## Key metrics summary

| Lever | Savings / Impact | Complexity |
|-------|-----------------|------------|
| Copilot Auto Mode | 10% discount | ⭐ Immediate |
| compressOutput + tool search | up to 20% tokens | ⭐ Immediate |
| Prompt caching | 90% on cached tokens | ⭐⭐ Low |
| RAG vs context stuffing | up to 70% reduction | ⭐⭐ Low |
| Correct model selection | up to 25x difference | ⭐⭐ Low |
| Batch API | 50% discount | ⭐⭐ Low |
| Batch + caching combined | up to 95% savings | ⭐⭐ Low |
| RouteLLM | 75% reduction in strong model calls | ⭐⭐⭐ Medium |
| AI Gateway (LiteLLM/Portkey) | Budget controls + protection | ⭐⭐⭐ Medium |
| OTel observability | Full visibility | ⭐⭐⭐ Medium |
| DeepSeek via Azure | ~15x cheaper than Sonnet | ⭐⭐⭐ Medium |
| Groq/Together/Mistral | 4–30x cheaper than frontier | ⭐⭐⭐⭐ High |
| Self-hosting vLLM | 100% token savings (own infra) | ⭐⭐⭐⭐⭐ High |

---

*Full documentation with code, examples, and implementation details: `PROD/technical-docs-draft-v1-EN.md`*
