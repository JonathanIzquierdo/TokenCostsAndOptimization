# Claude Design Brief — article-draft-v1-EN.md

This file contains all the instructions, data, and verified sources for Claude Design to add charts and visual metrics to the article `PROD/article-draft-v1-EN.md`.

---

## CRITICAL RULE — No number without a verified source

**Every chart, diagram, or visual metric must have its source linked directly below it or in the caption.** No number can appear in a visual without a real URL backing it up. If a number is not in `data/verified-metrics.md` with a verified URL, it cannot be used in a visual.

This is non-negotiable. It applies to every chart, every percentage, every dollar figure.

---

## Article context

**File to enhance:** `PROD/article-draft-v1-EN.md`
**Audience:** Enterprise engineering teams (Visma context) — developers and tech leads
**Tone:** Technical but approachable. A colleague sharing something important — not a corporate manual.
**Goal of visuals:** Make the data land harder. The numbers are strong — the visuals should make them impossible to ignore.

---

## Proposed visualizations with verified data

### Visual 1 — Where the money actually goes (Section: Opening)

**Type:** Pie chart or donut chart
**Message:** Most of the bill is repeated noise, not actual work.

| Segment | Value | Source |
|---------|-------|--------|
| Re-sent context (waste) | 62% | https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/ |
| Actual new work | 38% | Derived from above (100% - 62%) |

**Caption:** *"62% of your AI bill is context the model already processed. Source: LeanOps audit of 30 production teams, May 2026."*

---

### Visual 2 — Context explosion over a session (Section 1: The invisible problem)

**Type:** Line chart
**Message:** Token cost per call grows exponentially in long agentic sessions.

| Turn | Tokens per call |
|------|-----------------|
| 1 | 5,000 |
| 10 | ~15,000 |
| 20 | ~40,000 |
| 30 | ~80,000 |
| 40 | ~140,000 |
| 50 | 200,000 |

**Source:** https://redblink.com/ai-token-cost-optimization/ and https://redis.io/blog/llm-token-optimization-speed-up-apps/
**Caption:** *"A session starting at 5,000 tokens can reach 200,000 tokens per call by turn 50 — without anyone noticing. Source: Redblink AI Token Cost Optimization 2026."*

---

### Visual 3 — The output token multiplier (Section 1)

**Type:** Bar chart comparing input vs output price per model
**Message:** Output tokens cost 4–6x more. Nobody limits them.

| Model | Input /MTok | Output /MTok | Multiplier |
|-------|------------|-------------|------------|
| Claude Haiku 4.5 | $1.00 | $5.00 | 5x |
| Claude Sonnet 4.6 | $3.00 | $15.00 | 5x |
| Claude Opus 4.7 | $5.00 | $25.00 | 5x |

**Source:** https://www.finout.io/blog/anthropic-api-pricing (verified April 2026)
**Caption:** *"Output tokens cost 5x more than input across all current Anthropic models. Source: Finout, Anthropic API Pricing 2026."*

---

### Visual 4 — The Copilot spend distribution (Section 2: The urgent change)

**Type:** Bar or pareto chart
**Message:** A small group of users drives most of the cost.

| User group | % of users | % of spend |
|------------|-----------|------------|
| Heavy users | 10–15% | 60–70% |
| Rest of team | 85–90% | 30–40% |

**Source:** https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/
**Caption:** *"The top 10–15% of users concentrates 60–70% of AI spend. Source: Synapx, GitHub Copilot Usage-Based Billing Executive Guide 2026."*

---

### Visual 5 — The model price gap (Section 3, Layer 2)

**Type:** Horizontal bar chart
**Message:** The price difference between models is massive — and most teams ignore it.

| Model | Input /MTok | Use case |
|-------|------------|----------|
| Claude Haiku 4.5 | $1.00 | Classification, extraction, formatting |
| Claude Sonnet 4.6 | $3.00 | General code, chat, summarization |
| Claude Opus 4.7 | $5.00 | Complex reasoning, architecture |

**Source:** https://www.finout.io/blog/anthropic-api-pricing
**Caption:** *"Haiku is 5x cheaper than Sonnet and 5x cheaper than Opus on input. Most workflows use Sonnet or Opus for tasks Haiku handles equally well. Source: Anthropic official pricing, April 2026."*

---

### Visual 6 — The savings stack (Section 3, Layer 4)

**Type:** Waterfall or stacked bar chart
**Message:** Optimizations compound — you can get to 95% savings.

| Scenario | Cost relative to baseline |
|----------|--------------------------|
| Standard request (baseline) | 100% |
| With Batch API | 50% |
| With prompt caching | ~10% of cached portion |
| Batch + caching combined | ~5% |

**Sources:**
- Batch API 50%: https://www.finout.io/blog/anthropic-api-pricing
- Batch + caching = 5%: https://pecollective.com/tools/anthropic-api-pricing/
- Prompt caching 90% discount: https://www.tokenoptimize.dev/guides/llm-token-optimization-strategies

**Caption:** *"Batch API alone cuts costs 50%. Combined with prompt caching, a request can cost as little as 5% of the standard rate. Sources: Anthropic official, PECollective 2026."*

---

### Visual 7 — The optimization layers (Section 3 overview)

**Type:** Pyramid or tiered diagram
**Message:** There are layers of optimization — start anywhere, get value immediately.

```
Layer 4 — Infrastructure (Batch, DeepSeek, EA negotiation)
Layer 3 — Intelligent routing (RouteLLM, LiteLLM, Portkey)
Layer 2 — Model selection (right model for right task)
Layer 1 — Context architecture (caching, RAG, history management)
Layer 0 — Settings (Auto Mode, compressOutput, tool search) ← Start here
```

No numeric data needed for this one — it's structural.
**Note:** No specific metric caption needed. Add a note: *"Each layer is independent — Layer 0 takes 5 minutes and zero code changes."*

---

### Visual 8 — RouteLLM results (Section 3, Layer 3)

**Type:** Simple comparison bar or callout boxes
**Message:** ML-based routing achieves near-GPT-4 quality at a fraction of the calls.

| Approach | GPT-4 call rate | Quality vs GPT-4 | Cost savings |
|----------|----------------|------------------|--------------|
| Always GPT-4 | 100% | 100% | 0% |
| RouteLLM (matrix factorization) | 26% | 95% | ~48% |
| RouteLLM with data augmentation | 14% | 95% | ~75% |

**Source:** https://www.lmsys.org/blog/2024-07-01-routellm/ (LMSYS / UC Berkeley, ICLR 2025)
**Caption:** *"RouteLLM achieves 95% of GPT-4 quality using only 26% of GPT-4 calls. With data augmentation: same quality, 14% of calls. Source: LMSYS, RouteLLM ICLR 2025."*

---

## Design guidelines

**Color palette:** Use a consistent 2–3 color scheme. Suggest: a neutral dark for baselines, a strong accent (red or orange) for the problem, a positive accent (green or blue) for savings/solutions.

**Typography:** Clean, technical. No decorative fonts.

**Every visual must have:**
1. A short title (what this shows)
2. A caption with the source URL
3. No numbers that aren't in this brief

**Format:** The article is markdown. Visuals can be:
- Mermaid diagrams (renders natively in GitHub)
- SVG embedded in markdown
- PNG/SVG attached and referenced via `![alt](path)`

**Placement:** Each visual goes immediately after the paragraph that introduces its data point. Don't cluster visuals — space them throughout the article.

---

## Source reference — all verified URLs

| Data point | URL |
|------------|-----|
| 62% re-sent context | https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/ |
| 5K → 200K tokens turn 50 | https://redblink.com/ai-token-cost-optimization/ |
| Output 4–6x more than input | https://redis.io/blog/llm-token-optimization-speed-up-apps/ |
| Anthropic model pricing 2026 | https://www.finout.io/blog/anthropic-api-pricing |
| Top 10–15% users = 60–70% spend | https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/ |
| Agentic workflows ~3.5x flat fee | https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/ |
| Prompt caching 90% discount | https://www.tokenoptimize.dev/guides/llm-token-optimization-strategies |
| Batch API 50% discount | https://www.finout.io/blog/anthropic-api-pricing |
| Batch + caching = 5% of baseline | https://pecollective.com/tools/anthropic-api-pricing/ |
| RouteLLM 95% quality / 26% calls | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| RouteLLM with augmentation 75% | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| Auto Mode 10% discount | https://github.blog/changelog/2026-04-17-github-copilot-cli-now-supports-copilot-auto-model-selection/ |
| VSCode tool search 20% savings | https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx |
| DeepSeek via Azure ~15x cheaper | https://deploybase.ai/articles/deepseek-v3-pricing |
| Azure EA 15–28% negotiated | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| Opus 4.7 tokenizer +35% tokens | https://www.finout.io/blog/anthropic-api-pricing |
