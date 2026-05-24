# Final Technical Guide: AI Token Optimization
## From basic configurations to enterprise architecture, with narrative for all technical levels

**Audience:** Developers, tech leads, architects, technical leaders, budget decision-makers, and also people with less technical background who want to understand what to implement and why.
**Prerequisite:** Basic familiarity with LLMs and AI APIs. No need to be an expert in every tool. Each section includes enough context to follow along.

---

## How to read this guide

This document is intended as a **token optimization and cost reduction bible** for LLM consumption. It's not a mandatory linear read: each section solves a concrete problem and can be consulted independently.

Unlike classic technical documentation that assumes prior context, this doc wraps each snippet with the explanation of **where it goes, why it's done that way, when it's appropriate and how you verify it's working**. The idea is that both a senior dev who just wants to copy the snippet and a tech lead or decision-maker who needs to understand what they'll ask the team find what they need without feeling overwhelmed or lost.

The structure follows a progression of increasing impact:

- **Levels 0 and 1** are adjustments any developer can apply today in their IDE or application code. Typical savings of 20 to 40%.
- **Levels 2 and 3** are architecture decisions: how you assign models per task and how you route automatically. Savings of 50 to 80%.
- **Levels 4 to 6** are the enterprise layer: governance, cross-team observability, alternative providers. Here the control gets built so the cost doesn't overflow as usage scales.

Each code block is tagged with the **layer** where it lives (1 to 4) and **who touches it** in the team, so there's no ambiguity about where it's implemented.

### Index

1. [Fundamentals](#1-fundamentals)
2. [Where each optimization lives](#2-where-it-lives)
3. [Direct vendor vs hyperscaler: the procurement decision](#3-procurement)
4. [Level 0: immediate configurations](#4-level-0)
5. [Level 1: context architecture](#5-level-1)
6. [Level 2: model selection](#6-level-2)
7. [Level 3: automatic model routing](#7-level-3)
8. [Level 4: cost infrastructure and governance](#8-level-4)
9. [Level 5: observability with OpenTelemetry and personal governance](#9-otel)
10. [Level 6: alternative lower-cost providers](#10-alternatives)
11. [Verified metrics reference](#11-metrics)

---

## Visual convention for scripts

Each application code block carries a **micro-header** that serves two functions: locating the snippet in its physical layer (Layer 1 to Layer 4, see Section 2) and identifying which team role touches it. This avoids the classic problem of reading a fragment of code without knowing if it goes in the IDE, the app, the gateway, or the admin console.

The format is:

> 🔧 **Layer X** (description of physical location) · **Technology** · **Who touches it**

For example:
> 🔧 **Layer 1** (app code) · Anthropic Python SDK · Application developers calling `api.anthropic.com` directly

Snippets that **don't** have this header are **IDE settings** (VSCode, Copilot, Claude Code, Cursor). Those can be touched by any individual developer on their own installation, they don't require building custom apps or going through the platform.

---

## 1. Fundamentals

Before optimizing, it's worth understanding exactly what's being paid for. An LLM bill is not a black box: each component of a request has a different price, and understanding the composition of the cost is the first step to reducing it.

### 1.1 Anatomy of a request

Every call to an LLM is billed in three main blocks worth distinguishing from the start:

- **Input tokens.** Everything the model receives to process: system prompt (general assistant instructions), conversation history, tool definitions, attached documents and the user message. It's the "cheap" part of the price, but also the one that grows out of control if not governed.
- **Output tokens.** What the model generates: the response visible to the user, the internal reasoning (extended thinking) and the tool calls. **They typically cost 5x more than input tokens.** This means that limiting the length of the response has a disproportionate impact on the bill.
- **Cache reads.** When you reuse a prompt or a large block of context via prompt caching, cache reads are charged at **10% of the normal input price**. It's the biggest discount the platform offers, and the single optimization with the most impact in applications that reuse context (RAG, agents, assistants with knowledge bases).

The asymmetry 1x input, 5x output, 0.1x cache is the most useful mental rule for designing prompts: it's worth paying more in structured and cacheable input, in exchange for minimizing free-form output.

### 1.2 Reference prices 2026

This table summarizes the public prices of the main models as of May 2026, expressed in USD per million tokens (MTok). It's the starting reference for any cost analysis, but **the nominal price is not the effective price**: that's calculated including caching, batch, and the provider's overhead (explained in the following sections).

| Model | Input /MTok | Output /MTok | Cache read /MTok | Cache write /MTok |
|--------|-------------|---------------|--------------------|---------------------|
| Claude Haiku 4.5 | $1.00 | $5.00 | $0.10 | $1.25 |
| Claude Sonnet 4.6 | $3.00 | $15.00 | $0.30 | $3.75 |
| Claude Opus 4.7 | $5.00 | $25.00 | $0.50 | $6.25 |
| GPT-5.4 | $2.50 | $15.00 | $0.25 | n/a |
| GPT-5.5 (flagship) | $5.00 | $30.00 | $0.50 | n/a |
| GPT-5 Nano | $0.05 | $0.40 | n/a | n/a |
| DeepSeek V4-Flash (direct) | $0.14 | $0.28 | n/a | n/a |
| DeepSeek V4-Flash (via Azure) | ~$0.19 | ~$0.38 | n/a | n/a |
| Llama 4 via Groq | ~$0.11 | ~$0.34 | n/a | n/a |

Sources: [Anthropic official May 2026](https://platform.claude.com/docs/en/about-claude/pricing), [OpenAI official](https://openai.com/api/pricing/), [Finout 2026](https://www.finout.io/blog/openai-pricing-in-2026), [DeployBase](https://deploybase.ai/articles/deepseek-v3-pricing), [ToolHalla](https://toolhalla.ai/blog/groq-vs-together-vs-fireworks-2026)

The first thing that jumps out is that **there's a difference of up to 100x between the cheapest model (GPT-5 Nano at $0.05) and the most expensive (GPT-5.5 at $5)** just in input. That asymmetry is the basis of the whole "model assignment per task" strategy developed in Level 2: simple tasks (classification, extraction, file navigation) don't need the flagship model, and the savings are immediate.

> **Opus 4.7 note:** The new Opus 4.7 tokenizer generates **up to 35% more tokens for the same input** compared to Opus 4.6. The price per token didn't change, but the effective cost per request can be up to 35% higher. If your team is evaluating migrating from Opus 4.6 to 4.7, it's worth benchmarking with your own prompts before deciding: the increase can offset the capability improvement, or not.
> Source: [Finout, Anthropic API Pricing 2026](https://www.finout.io/blog/anthropic-api-pricing)

### 1.3 Accumulated context: the invisible problem

This is the most underestimated concept in agentic and chat applications. In a long conversation or session, **context doesn't stay constant: it accumulates**. Each turn re-sends everything previous to the model so it can reason with the complete conversation, and that multiplies billed tokens silently.

The typical progression in an agentic session is:
- Turn 1: ~5K tokens (system prompt + initial message)
- Turn 10: ~15K tokens (system prompt + 10 previous turns)
- Turn 50: **~200K tokens per call** (system prompt + 50 turns + tool outputs + loaded documents)

By turn 50 you're paying 40 times more for the same "conversation" than at turn 1. And since each call reprocesses everything from scratch, the bill grows quadratically relative to session length.

The quantified data is compelling: according to [LeanOps (study on 30 teams in 2026)](https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/), **62% of the AI bill corresponds to re-sent context**, not new information. That is: on average, almost 2/3 of what you spend on tokens is repaying content you already paid for before.

This is why prompt caching (Section 5.1) and history compaction (Section 5.2) are the two highest-impact optimizations in any conversational or agentic app.

### 1.4 Extended thinking: the invisible cost

Models with "extended thinking" (Claude Opus, Sonnet, o5-style at OpenAI) have an internal reasoning layer that's billed as output, even if the user doesn't see it. This is probably the most common and most expensive mistake in production: teams that leave thinking enabled in default mode, thinking that hiding it from the display deactivates it.

**No: hiding the reasoning doesn't deactivate it or make it cheaper.** The thinking tokens keep getting generated and billed, they're simply not shown.

**Where this snippet goes:** in your application code, in the file where you initialize the Anthropic client and build the requests to the model. Typically a file like `services/llm_client.py` or `lib/anthropic.ts`, not something touched from an IDE.

> 🔧 **Layer 1** (app code) · Anthropic Python SDK · Application developers calling `api.anthropic.com/v1/messages` directly

```python
# INCORRECT: display: omitted does NOT reduce cost
response = client.messages.create(
    model="claude-opus-4-7",
    thinking={"type": "enabled", "budget_tokens": 5000, "display": "omitted"}
    # The 5000 tokens are still billed as output
)

# CORRECT: limit the budget with adaptive mode
response = client.messages.create(
    model="claude-opus-4-7",
    thinking={"type": "adaptive", "effort": "low"}
)
```

**What each parameter does:**
- `type: "enabled"` activates thinking with a fixed token budget that's always reserved, even if the task doesn't need it.
- `display: "omitted"` only controls the visibility of the reasoning in the response. **It doesn't affect cost.**
- `type: "adaptive"` lets the model decide how much to reason based on the task complexity. It's the right option for production.
- `effort: "low"` (also `medium`, `high`) tells the model how generous to be with the reasoning budget.

**How you verify:** in the API response, the `usage` object brings `output_tokens` broken down. If you compare two equivalent calls with and without thinking, the difference in `output_tokens` is exactly what you paid extra for enabling thinking.

The concrete impact: a 500-token output response combined with 2,000 tokens of thinking ends up costing **5 times more** than the same response without thinking activated. That's why the operational recommendation is: use `adaptive` with `effort: low` by default and raise the effort only on tasks that demonstrate real benefit with extended reasoning (architecture, complex debugging, multi-step reasoning). For classification, extraction or factual responses, thinking is pure waste.

Source: [PECollective, 2026](https://pecollective.com/tools/anthropic-api-pricing/)

---

## 2. Where each optimization lives <a id="2-where-it-lives"></a>

This is probably the most important section in the document, because it solves a problem almost all teams suffer at the start: **looking for the optimization in the wrong place**. There's a very frequent pattern: a dev tries to optimize from an IDE setting something that can only be configured in application code, or a leader asks to "activate caching for everyone" when users consume AI through a closed product where that option doesn't exist.

To avoid that confusion, it's worth mapping precisely the physical places where optimization is touched. There are four, and each has a different audience, its own parameters and its own impact ceiling.

### 2.1 The four control layers

| Layer | Physical location | Who touches it | Optimizations that live here | Savings ceiling |
|------|---------------|----------------|---------------------------|-------------------|
| **Layer 1** | Application code (Python/JS/etc) that calls the API directly | Developers building internal LLM apps | Prompt caching, thinking budgets, max_tokens, model per task, batch API, output budgets, beta headers | **Up to 95% combined** |
| **Layer 2** | IDE settings (VSCode, Copilot, Claude Code, Cursor) | Any individual dev or admin via MDM (Mobile Device Management, centralized configuration distribution) | Auto Mode, compressOutput, tool search, debug panels, local OTel | **Up to 30% combined** |
| **Layer 3** | Billing console / platform admin | Tech lead, tenant admin | Allowed/blocked models, per-workspace budgets, alerts, audit logs | **Indirect: governance** |
| **Layer 4** | Gateway / routing layer (LiteLLM, Portkey, OpenRouter, RouteLLM) | Platform / infra | Automatic routing between models, fallbacks, retries, per-team budgets, guardrails | **20 to 80% per [IDC](https://www.infoworld.com/article/4164236/github-shifts-copilot-to-usage-based-billing)** |

How to read this table: if your team only has developers using closed products (Copilot, Cursor), most of your savings will come from Layer 2 (configuring the IDE well) and license governance in Layer 3. If your team builds apps that call the API directly, the strongest optimizations are in Layer 1 (caching, batch, model assignment), and eventually Layer 4 when there are several services and you need to centralize governance.

### 2.2 The spectrum: closed products, configurable tools, direct API

Before discussing the four layers there's an earlier, conceptual distinction that defines which layers you have access to. A year and a half ago, the AI world was clearly divided in two: tools (black box) and APIs (transparent). Today that binary division is obsolete. What we have is a **spectrum of three levels**, and many people still reason with the old model, which produces wrong expectations.

**Closed products.** Consumer products without instrumentation designed for the user. Examples: ChatGPT consumer version, Gemini consumer version, Claude.ai on Free/Pro plans. The only thing you see is the chat, the response and some monthly plan quota. No model picker, no exportable telemetry, you can't modify parameters. Realistic savings ceiling: change plan or cancel.

**Configurable tools.** Enterprise products or IDEs with visibility, model picker, OpenTelemetry telemetry and rich settings. Examples: Copilot in VSCode/Visual Studio, Claude Code, Claude Cowork on Team/Enterprise plans. You have access to Layer 2 (IDE settings) and part of Layer 3 (product console). Caching is done by the tool for you. Cursor is in this group but still doesn't expose OTel natively, has Admin Dashboard with its own API and CSV exports in Enterprise, and there are community hooks (cursor-otel-hook, CursorLens) that close the gap. Realistic savings ceiling: ~30% over unoptimized consumption.

**Direct API.** You write the code that calls the model. Examples: Anthropic direct API, OpenAI API, AWS Bedrock, Google Vertex, Azure OpenAI. You have access to all four layers. The big optimizations (90% caching, 50% batch, 20 to 80% routing) live in Layer 1 and Layer 4. Realistic savings ceiling: over 90%.

**A BU (Business Unit, business unit within Visma) can be at all three levels simultaneously.** Configurable tools for devs to use Copilot, direct API for a product feature that calls LLM underneath, and maybe some team using ChatGPT consumer informally. That's not a problem, it's reality. What matters is knowing **at which level each use case sits** and not assuming that anything that's not direct API is a closed product.

The simple rule for designing new implementations:
- **Configurable tools for human use** (a dev writing code, someone exploring ideas).
- **Direct API for programmatic use** (a product feature that calls a model with no human on the other side).

Closed products are a problem mainly when they're used for cases that warrant something else: if a team is using ChatGPT consumer for tasks that require observability, governance or repeatability, there's something to review there.

**This document assumes you're building, or moving toward that.** The layers that follow take for granted that your team writes application code against an API or is evaluating doing so. If today you're 100% on closed products and want to significantly reduce costs, the previous step is to move up at least to a configurable tool or build an internal wrapper with API key. Only from there do the optimizations worth the effort open up.

### 2.3 How each script connects to its layer

To make it easy to visually locate each code block in its place, this table maps the most common technologies to their layer. It's the "quick map" of the document: if later you see a snippet with `client.messages.create(...)` and doubt where it goes, this is the reference.

| Type of snippet | Layer | Who touches it |
|------------------|------|----------------|
| `client.messages.create(...)`, SDK calls | Layer 1 | App developers |
| `client.messages.batches.create(...)` | Layer 1 | App developers |
| `cache_control: {type: "ephemeral"}` | Layer 1 | App developers |
| `thinking={"type": ...}` | Layer 1 | App developers |
| `OUTPUT_BUDGETS = {...}` | Layer 1 | App developers |
| `MODEL_ASSIGNMENT = {...}` | Layer 1 (in internal wrapper) or Layer 4 (in gateway) | Platform or developers |
| `chat.tools.compressOutput.enabled` in `settings.json` | Layer 2 | Any dev |
| `github.copilot.chat.agentDebugLog.enabled` | Layer 2 | Any dev |
| `export CLAUDE_CODE_ENABLE_TELEMETRY=1` | Layer 2 (local env vars) or Layer 3 (enterprise MDM) | Individual dev or admin |
| `Controller(routers=["mf"], ...)` (RouteLLM) | Layer 4 | Platform |
| `Router(model_list=[...], budget_manager=...)` (LiteLLM) | Layer 4 | Platform |
| `Portkey(api_key=..., virtual_key=...)` | Layer 4 | Platform |

### 2.4 How to govern Layer 1 at scale

There's an operational observation worth anticipating: if your team has 10 different microservices calling the Anthropic SDK directly, each one decides on its own thinking, caching, max_tokens, model. Typical production result: 10 different levels of "optimization", drift between services, and an unpredictable bill nobody can really explain.

The root cause isn't technical, it's organizational: when configuration decisions live distributed across N repositories, good practices don't replicate themselves. Each new service starts with different defaults depending on which developer kicked it off.

There are two ways to govern it, and it's worth choosing **before** the problem appears:

**Option A: shared internal wrapper.** An internal library that wraps the provider's SDK and applies sane hardcoded defaults: `adaptive thinking effort=low`, Haiku model unless explicit override, prompt caching active by default, max_tokens according to task type. All services consume the wrapper instead of the direct SDK. The advantage is that it's a single decision, replicated everywhere, and each future change propagates with version bumps. It's the cleanest path for small or medium teams.

**Option B: shared gateway (LiteLLM / Portkey).** Instead of wrapping the SDK in code, calls go through an HTTP gateway that applies policy: "if nobody specified thinking, force low", "if nobody specified model, use Haiku", "if a request cost exceeds $X, alert and/or block". The services' code doesn't change, the gateway intercepts and normalizes from the network. Advantage: it works even if each service is in a different language and per-team budgets are governed centrally.

Practical rule: **for teams up to ~5 services, a wrapper is enough**. Beyond that, a gateway starts to justify itself because it enables multi-team, multi-language control, automatic fallbacks and unified observability without touching application code.

---

## 3. Direct vendor vs hyperscaler: the procurement decision <a id="3-procurement"></a>

This section documents the verified data worth having on the table when evaluating or negotiating an LLM consumption agreement with a provider. It's one of the areas where real cost is most misunderstood: the nominal price per token is only part of the effective price, and the difference between the two can reach 40%.

### 3.1 Price structure

When consuming an LLM with an API key, total cost is composed of **three elements**, and it's worth keeping them separated when comparing providers:

1. **Nominal price per token.** What appears on the model's pricing page. It's the number everyone looks at first and what appears in commercial proposals.
2. **Request modifiers.** Caching, batch, long context, fast mode, data residency. They are multiplicative factors (some lower the price, others raise it) and depend on how you build each call.
3. **Provider overhead.** Support plans, data transfer (egress), fine-tuned model hosting, monitoring infra, audit logging. They are **fixed or additional usage costs** that the nominal pricing doesn't capture. The sum of nominal pricing + overhead is what's known as **TCO (Total Cost of Ownership), the real total cost of operating the service**, summing everything that doesn't appear on the pricing page.

The nominal price is where most companies compare, and where apparently "the hyperscaler with the discount wins". The overhead is where the math flips and many comparisons prove wrong after the first month of production.

### 3.2 Nominal pricing parity between providers (May 2026)

A piece of data worth internalizing before any negotiation: Anthropic maintains **per-token pricing parity** between its direct API and the main hyperscalers. That is, the model costs exactly the same per token, regardless of whether you access it directly or through a cloud provider:

| Claude Model | Anthropic direct | AWS Bedrock global | Vertex AI global | Microsoft Foundry |
|----------------|---------------------|----------------------|--------------------|---------------------|
| Sonnet 4.6 | $3 / $15 | $3 / $15 | $3 / $15 | $3 / $15 |
| Opus 4.7 | $5 / $25 | $5 / $25 | $5 / $25 | $5 / $25 |
| Haiku 4.5 | $1 / $5 | $1 / $5 | $1 / $5 | $1 / $5 |

Source: [Official Anthropic Pricing Docs](https://platform.claude.com/docs/en/about-claude/pricing), [Anthropic Claude in Microsoft Foundry](https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry)

This breaks a frequent myth in enterprise negotiations: "we go with the hyperscaler because they give us a better price on Claude". **Not true at the nominal level.** The hyperscaler can give you other things (unified procurement, compliance, credits), but not a better per-token price.

**Explicit multipliers published by Anthropic:**
- Bedrock regional endpoints: +10% over global
- Vertex regional/multi-region: +10% over global
- Claude direct API with `inference_geo: "us"`: +10% (1.1x multiplier)

These are the only official and public modifiers. Any other markup that appears on the bill corresponds to provider overhead.

The same happens with OpenAI vs Azure OpenAI: identical per-token pricing, different overhead.

### 3.3 Effective TCO: the real overhead

This is where the conversation becomes concrete. Different independent sources have measured the effective total cost of consuming LLMs through hyperscalers, and the data is consistent.

**Azure OpenAI vs direct OpenAI: 15 to 40% effective overhead, average +22%**

Three independent 2026 sources agree on the range:

| Source | Documented range | Verbatim quote |
|--------|---------------------|---------------|
| [TokenMix May 2026](https://tokenmix.ai/blog/azure-openai-cost) | 15 to 40%, avg 22% | "Token pricing identical. Total cost runs 15-40% higher on Azure due to support plans, data transfer, storage, and network infrastructure." |
| [Inference.net January 2026](https://inference.net/content/azure-openai-pricing-explained/) | 15 to 40% | "Total cost runs 15-40% higher on Azure due to support plans, data transfer, storage, and network infrastructure." |
| [CloudZero May 2026](https://www.cloudzero.com/blog/azure-openai-pricing/) | 20 to 40% | "Production deployments typically add 20-40% above listed token rates." |

The components of Azure overhead are the following, worth identifying so you can demand accounting in a negotiation:

- **Support plans**: $100 to $1,000+/month. In enterprise production, Standard support is de facto mandatory, so that cost always enters.
- **Data egress**: the first 100GB outbound are free, after that $0.087/GB is charged. In applications with considerable traffic, this accumulates fast.
- **Fine-tuned model hosting**: $1.70 to $3/hour **even with no use**, that is between $1,224 and $2,160/month per model, simply for having it deployed. If you have several fine-tuned models, multiply.
- **VNet integration, Private Link, content filtering**: $200 to $2,000/month depending on configuration. They are costs many companies need for compliance, but that the model's pricing page doesn't show.
- **Log Analytics and monitoring infra**: variable, depends on volume and retention.

**AWS Bedrock for Claude: 20 to 35% effective overhead**

Identical nominal pricing to Anthropic direct, but according to [TokenMix Bedrock 2026](https://tokenmix.ai/blog/aws-bedrock-pricing):
- Regional endpoints: +10% (official figure published by Anthropic).
- Cross-region inference: +10% additional when routing between regions.
- Bedrock counts data transfer (egress). Direct API doesn't.
- Bedrock bills **all** HTTP 500 errors. Direct API has a 3% forgiveness buffer on server errors that aren't charged.
- Mandatory CloudWatch and CloudTrail logging, with its associated cost.

Verbatim quote: *"TokenMix.ai cost tracking shows enterprises running Claude on Bedrock pay an average of 20-35% more than those using Anthropic's direct API, and most do not realize it."*

**Microsoft Foundry for Claude: identical pricing but documented trap**

This is probably the most subtle and most expensive case, because at first glance it seems the same price. Claude is billed in Foundry as a **third-party Marketplace** item, not as a native Azure resource. The critical implication is documented by [Microsoft Q&A](https://learn.microsoft.com/en-us/answers/questions/5851352) and [AZ365.ai March 2026](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/):

> "Claude models in Azure AI Foundry are billed as third-party Marketplace items, meaning their usage is typically not eligible for Azure sponsorship or startup credits... This often results in charges being applied directly to the credit card on file even if you have a significant remaining credit balance."

In plain terms: **Claude consumption in Foundry doesn't draw down your Azure credits or your MACC (Microsoft Azure Consumption Commitment, the multi-year Azure spend commitment that enables volume discounts).** It goes straight to the credit card associated with the account. If your procurement plan assumed MACC absorbed Claude spend, you'll get a surprise.

**Real documented cases** ([The Register / AZ365.ai](https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/)):
- Japanese founder (Leach): ¥237,081 (~$1,600 USD) charged directly to card, with credits not applied.
- German founder (Bogdan Sevriukov): €999.60 charged directly.
- Another Japanese founder: ¥2,000,000 (~$13,000 USD) in one month.
- Additional case reported by The Register: approximately $3,000 USD.

**Concrete action:** If your company is negotiating a Microsoft agreement assuming MACC absorbs Claude consumption via Foundry, **request explicit written clarification** on what types of credits apply to Claude and under what conditions. If the answer is ambiguous, assume they don't apply.

### 3.4 Real enterprise discounts

Here the verified data on what discounts are actually achieved in real negotiations. They serve as benchmark for not accepting the first vendor offer:

| Discount type | Documented range | Source |
|--------------------|--------------------|---------|
| Azure EA alone (Enterprise Agreement, the corporate master contract with Microsoft) | 15 to 25% negotiated | [Microsoft Negotiations](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing) |
| Azure EA + MACC | 23 to 28% negotiated | [Microsoft Negotiations](https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing) |
| Anthropic Enterprise / volume | Available, custom | [CloudZero](https://www.cloudzero.com/blog/claude-pricing/) |
| OpenAI Enterprise tier | Custom pricing | [Finout, OpenAI](https://www.finout.io/blog/openai-pricing-in-2026) |
| AWS Bedrock private offers | Available | Anthropic docs |

### 3.5 The net math: discount vs overhead

This is the table worth having at hand when someone asks "but doesn't Azure suit us because we have a discount?". Suppose a base workload = 100 cost units when consumed from a direct vendor:

| Scenario | Direct cost | Azure cost | Bedrock cost | Net vs direct |
|-----------|----------------|---------------|----------------|-------------------|
| GPT-5.4, no compliance, no discount | 100 | 115 to 140 | n/a | **+15% to +40%** |
| GPT-5.4, EA alone (-20%) over 115 to 140 | 100 | 92 to 112 | n/a | **-8% to +12%** |
| GPT-5.4, EA + MACC (-25%) over 115 to 140 | 100 | 86 to 105 | n/a | **-14% to +5%** |
| Claude Sonnet, no compliance | 100 | n/a (Foundry no disc.) | 120 to 135 | **+20% to +35%** |
| DeepSeek vs Azure (no compliance) | 100 | 120 to 135 | n/a | **+20% to +35%** |
| DeepSeek vs Azure (with EU GDPR) | unviable | 120 to 135 | n/a | **compliance premium** |

**Operational conclusion:** the EA/MACC discount typically compensates but **doesn't beat** the overhead. The final bill, after discounts, sits in a range between -14% and +12% compared to direct vendor. That is: when discount is good, the hyperscaler can be slightly cheaper; when the discount is smaller or the workload generates a lot of overhead, it comes out more expensive. The real decision **is won or lost on compliance and procurement, not on price discount**.

### 3.6 When the hyperscaler is worth it (decision matrix)

From the previous data, this matrix summarizes when each path is appropriate. There's no universal answer: it depends on the dominant criterion in your organization.

| Dominant criterion | Recommended path |
|---------------------|----------------------|
| Enterprise-mature own vendor model (Anthropic, OpenAI), no Azure-only mandate | **Direct vendor**, features first, better net price, support out of the box |
| Geopolitically sensitive model (DeepSeek, Qwen, Chinese models) | **Hyperscaler (Azure preferred)**, the premium buys compliance |
| Corporate "all in Azure/AWS" mandate | **Hyperscaler**, accept 15 to 40% overhead as governance cost |
| MACC burn need / unified Microsoft billing | **Hyperscaler**, accounting decision, **exclude Claude on Foundry** |
| Self-hosted open-source model (Llama, Mistral) | **Hyperscaler with managed endpoint or self-host**, Bedrock/Vertex or own GPUs |
| Team in exploration, prototypes, POCs | **Direct vendor**, fast onboarding, less bureaucracy |
| Mandatory GDPR / data residency compliance | **Hyperscaler** or **Mistral** (only direct major European option) |

### 3.7 Input for negotiation

When you sit down with any vendor (Microsoft, AWS, Google, OpenAI, Anthropic), these are the verified data worth having at hand. They serve both for asking for discounts and for detecting exaggerated promises in a commercial proposal:

| Data | Value | Source | Use in negotiation |
|------|-------|--------|---------------------|
| Azure overhead vs direct OpenAI | 15 to 40% (avg 22%) | TokenMix, Inference.net, CloudZero | "We need a discount that covers at least 25% to break even against direct" |
| Bedrock overhead vs direct Anthropic | 20 to 35% effective | TokenMix Bedrock | "Foundry or direct gives us the same price without that markup" |
| Foundry Claude credits exclusion | Documented | Microsoft Q&A, AZ365.ai | "Explicit written clarification on what credits apply to Claude" |
| Anthropic Enterprise volume discount | Available | CloudZero | "Request comparable terms to hyperscaler with MACC" |
| OpenAI Enterprise custom pricing | Available | Finout | "Request same regime as Azure but without overhead" |
| Pricing parity Claude in hyperscalers | Confirmed | Anthropic docs | "The hyperscaler doesn't give us better nominal price, only procurement" |

---

## 4. Level 0: immediate configurations <a id="4-level-0"></a>

This level gathers all the optimizations that **any developer can activate today on their own VSCode/Copilot installation**, without needing permissions, without touching application code and without building new infrastructure. They are the "quick wins" of the document: low effort, immediate impact, mandatory baseline before moving to more complex levels.

> 🔧 **Layer 2** (IDE settings) · All snippets in this section are VSCode `settings.json` JSON · Any individual dev can activate them

**Where to edit these settings.** Everything that follows goes into VSCode's `settings.json`. The fastest way to open it:

1. Open Command Palette: `Ctrl+Shift+P` (Linux/Windows) or `Cmd+Shift+P` (Mac).
2. Type "Preferences: Open User Settings (JSON)".
3. A `settings.json` file opens where you can paste the lines from each subsection.

If you've never touched `settings.json`, you'll see a file with braces `{}`. Options are separated with commas. If unsure, there's also "Preferences: Open Settings" (without "(JSON)"), which is a searchable UI where you can type the setting name and activate it with a toggle.

### 4.1 Auto Mode in GitHub Copilot

This is the fastest optimization that exists in Copilot: change the model selector to **"Auto"** in VSCode. In Auto mode, Copilot automatically chooses the appropriate model for each request, and the platform applies a **10% automatic discount** on the multiplier. Without losing quality for standard tasks, since Auto Mode routes simple tasks to cheaper models and reserves the flagship only for those that justify it.

**How to activate:** in the Copilot Chat panel, above the input box there's a selector with the current model name (typically "GPT-4" or "Claude Sonnet"). Click there, select "Auto". Done.

**How to verify it works:** after activating Auto, in the lower bar of Copilot Chat you'll see indicated which model it chose for each response. If for simple questions you see cheap models (Haiku, GPT-5 Nano) and for complex questions you see Sonnet or Opus, the routing is doing its job.

Source: [GitHub Changelog, April 2026](https://github.blog/changelog/2026-04-17-github-copilot-cli-now-supports-copilot-auto-model-selection/)

### 4.2 chat.tools.compressOutput.enabled

VSCode 1.120 introduced a terminal output post-processing option that significantly reduces the size of outputs the agent returns. What it does concretely: collapses unchanged diffs, discards bulky lockfiles, reduces `ls -l` to just file names, removes progress bars and useless ANSI outputs.

**How to activate:** in `settings.json`, add:

```json
{ "chat.tools.compressOutput.enabled": true }
```

**What happens when you activate it.** It's transparent to the agent flow, the logic doesn't change, it simply saves the agent from having to process (and pay for) garbage tokens. The agent keeps executing the same commands, but before returning the output to you, VSCode cleans it.

**When it's appropriate.** Always. It has no known downside. Activating it is the first thing any dev should do when installing Copilot.

**How to verify.** Open Agent Debug Log (Section 4.4). When a command executes something bulky (a `pnpm install` for example), compare it with and without compressOutput activated: you'll see much shorter outputs sent to the model, and that translates directly into fewer billed tokens.

### 4.3 Tool search: deferred MCPs

When you have several MCPs configured, each call includes the definitions of all available tools, even if the model isn't going to use them. Tool search breaks that pattern: instead of sending all definitions upfront, the model receives just an index and looks up tools on demand when it needs them.

It's default from Anthropic Sonnet 4.5+. For GPT in Copilot it has to be activated manually.

**How to activate:** in `settings.json`:

```json
{ "github.copilot.chat.responsesApi.toolSearchTool.enabled": true }
```

**What happens.** Before tool search, if you had 10 active MCPs, each request sent the definitions of the 10 (typically 500 tokens per MCP = 5,000 tokens of overhead per request). With tool search, the model receives just a lightweight index and only loads the definitions of the tools it's going to use.

**When it's appropriate.** When you have 2 or more active MCPs. If you have just one, the savings are marginal. If you have 5+, the savings are significant.

Measured impact: up to **20% savings in tokens per request**, depending on how many MCPs you have active. Source: [Visual Studio Magazine, 2026](https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx)

### 4.4 Agent Debug Log Panel: the complete trace inside VSCode

This is probably the most underused tool in the stack: the most direct way to see what's happening in an agent session without configuring any external backend. It shows consumed tokens, executed tool calls, model turns, subagents, errors and a **visual flow chart of the agent**, all rendered directly in the editor.

Why does it matter for cost optimization? Because until you measure, you can't optimize. Most devs have no idea how many tokens each session consumes, nor how many tool calls fire silently, nor what subagents are generated in cascade. This panel makes all that visible in real time.

**How to open the panel:**
- Overflow menu `(...)` in the Copilot Chat panel, **"Show Agent Debug Logs"**
- Or Command Palette (`Cmd/Ctrl+Shift+P`), **"Developer: Open Agent Debug Logs"**

**Requires enabling the setting:**

```json
{ "github.copilot.chat.agentDebugLog.enabled": true }
```

Without this setting, the menu command shows nothing.

**What it shows per session:**
- **Overview:** total model turns, executed tool calls, total tokens consumed, errors
- **View Logs:** chronological list of all events with timestamps and details, filterable by type
- **Agent Flow Chart:** visual graph of the execution flow. Shows how model turns, tool calls and nested subagents relate. It's especially useful when an agent "goes off on tangents", the graph clearly shows where it diverged.

Source: [VSCode Docs, Debug Chat Interactions](https://code.visualstudio.com/docs/copilot/chat/chat-debug-view)

**Persist past sessions to disk:**

```json
{ "github.copilot.chat.agentDebugLog.fileLogging.enabled": true }
```

With file logging active, any session is logged and can be reviewed later: tool calls, LLM requests, token usage, errors. Useful for postmortem analysis when a session came out expensive and you want to understand why.

Source: [dvlprlife.com, Quick Tips: Debug Copilot Agent Session Logs, May 2026](https://www.dvlprlife.com/2026/05/quick-tips-debug-copilot-agent-session-logs-in-vs-code/)

**Export to OTLP JSON:** Each session can be exported to share, archive or ingest into any OTel backend (Jaeger, Grafana, Langfuse, etc.). Click on the Export icon (download) in the panel toolbar. This is the bridge between local observability (Layer 2) and centralized observability (Layer 3, see Section 9).

**The /troubleshoot command:** With file logging enabled, you can ask Copilot itself to analyze its own logs:

```
/troubleshoot how many tokens did I use?
/troubleshoot why did it skip that tool?
```

It's meta-debugging: the LLM analyzes its own previous behavior. Useful for finding wasteful patterns when it's not obvious at first glance.

Source: [VSCode Docs, Troubleshoot AI in Visual Studio Code](https://code.visualstudio.com/docs/copilot/troubleshooting)

### 4.5 Chat Debug View: request-by-request inspection

While Agent Debug Log gives the aggregate view, the **Chat Debug View** shows the detail of each individual request: complete system prompt sent to the model, user prompt, included context (files, snippets, history), and model response. It's the "mechanical" view of the IDE, equivalent to opening the network inspector in a browser.

It serves mostly to audit **what's being sent to the model on each turn**. Sometimes what's being sent is much more than the dev imagined (entire files when a fragment was enough, complete history when it could be summarized).

**How to open:** Menu `(...)`, **"Show Chat Debug View"** or Command Palette, **"Developer: Show Chat Debug View"**

Source: [Medium, GitHub Copilot Token Usage Explained, Simform Engineering, May 2026](https://medium.com/simform-engineering/github-copilot-token-usage-explained-with-practical-cost-control-03062b15ecb0)

### 4.6 Token usage visibility for BYOK (VSCode 1.120)

Before VSCode version 1.120, when a dev used their own API key (BYOK, Bring Your Own Key), the token usage indicator in Copilot always showed 0%, which generated the false impression of "zero consumption". Version 1.120 fixes that accounting: now the indicator correctly reflects real use when BYOK is used, so devs can monitor their own cost even when paying Anthropic/OpenAI directly.

### 4.7 Audit active MCPs

A cheap and underestimated optimization: each active MCP adds definition tokens to the base context of each request. The numbers matter when there are many active MCPs.

The typical arithmetic: **10 MCPs × 500 tokens average per definition = 5,000 tokens of overhead per request**. Multiplied by 100,000 requests per month with Sonnet ($3/MTok input), that's **$1,500/month just in MCPs that aren't being used**. And that's without counting that tool search partially mitigates it, but doesn't fully eliminate it.

**Action:** make a pass periodically (monthly ideally) through the active MCPs and deactivate those the team doesn't use. If an MCP is only needed for a specific task, activate it on demand and deactivate it afterward.

**Where to manage active MCPs.** In VSCode, Command Palette, "MCP: Show Installed Servers". You'll see a list with each MCP and a toggle to individually activate/deactivate them.

---

## 5. Level 1: context architecture <a id="5-level-1"></a>

While Level 0 was all configuration on closed products, Level 1 enters the territory where you really win: **the design of the context you send to the model in each request**. Three optimizations, all in Layer 1:

1. **Prompt caching.** Pay 10% for repeated content.
2. **History management.** Don't re-send 200K tokens when 10K is enough.
3. **RAG and output budgets.** Send only what's relevant and limit the length of the response.

### 5.1 Prompt caching: the highest-impact individual optimization

Prompt caching is the single highest-impact optimization in applications that reuse context. The idea is simple: if a block of content repeats between calls (a long system prompt, a knowledge base, a style guide, the first N turns of a session), it can be marked so the provider caches it and charges only 10% of the price on subsequent calls.

#### How caching economics work

Cost breaks down like this:

| Operation | Cost |
|-----------|-------|
| Cache write TTL 5 min | 1.25x base |
| Cache write TTL 1 hour | 2.0x base |
| Cache read | 0.10x base (90% discount) |

**How to read this table:** the first time you send a cacheable block, you pay a little more (1.25x if you want it to last 5 minutes, 2x if you want it to last an hour). From then on, every time you reuse that block, you pay only 10% of its normal cost. If you reuse it twice within the window, you're already saving. If you reuse it a hundred times, the savings are huge.

The break-even is **a single hit**. Any app with a stable system prompt or RAG is leaving money on the table if it doesn't activate it.

#### Caching when using configurable tools (Claude Code, Copilot, Claude Cowork)

If you use tools like Claude Code, Claude Cowork or Copilot Chat, **the caching is done by the tool for you**. You don't have to activate anything. Your `CLAUDE.md`, your system prompt, the files you load, all go to cache automatically. VSCode has published that its prompt caching reuse rate inside the editor is **93%**.

What you can do when you're on a configurable tool is **measure the effect** (with OpenTelemetry, see Section 9) and **favor cache with good practices**:

- Keep the structure of your prompts stable. Changing the order of instructions invalidates the cache.
- Don't close and open files unnecessarily. Every time you change the set of open files, you might be invalidating portions of the cache.
- A long conversation in the same context caches well; jumping between projects resets the count.
- If Claude Code is acting slow or consuming a lot, open a new session instead of continuing last Monday's.

#### Caching when consuming direct Anthropic API

Here you do activate it yourself, and there are **two modes**: automatic and explicit. The difference has to do with how much control you want over where the cache breakpoint is placed.

##### Automatic caching: the "you start in 30 seconds" mode

Anthropic added automatic caching to its API in April 2026. The idea: instead of manually marking which block to cache, you put a single top-level field on the request and the system places the breakpoint on its own, on the last cacheable block.

**Where this snippet goes:** in your application code, in the file where you initialize the Anthropic client and build each `messages.create()`. Typically a service like `services/llm_client.py`.

> 🔧 **Layer 1** (app code) · Anthropic Python SDK · Application developers calling `api.anthropic.com` directly

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    cache_control={"type": "ephemeral"},  # automatic caching, top-level
    system="You are an internal Visma assistant. ...long system prompt...",
    messages=[{"role": "user", "content": user_message}]
)
```

**What this parameter does.** Anthropic receives the request and automatically places the cache breakpoint on the last cacheable block (the system prompt in this case). On subsequent calls with the same content, the system prompt is served from cache at 10% of cost.

**When automatic is appropriate.** It's the recommended mode to start. If you're just incorporating caching into your app and want to see the savings quickly without having to decide what to cache, this mode gets you to 80% of the benefit with zero design decisions.

##### Explicit caching: the "fine control" mode

When you need more granular caching (multiple cacheable blocks with different TTLs, control over exactly what's cached and what's not), you move to explicit. You mark each block you want to cache with `cache_control` directly. Up to 4 breakpoints per request.

> 🔧 **Layer 1** (app code) · Anthropic Python SDK · Application developers who need fine control of caching

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
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

**What it does:**
- The first block of the system prompt (the assistant instructions) has its own `cache_control`. It will be cached as an independent block.
- The second block (the knowledge base) has another `cache_control`. Also cached as an independent block.
- If in future calls the instructions change but the knowledge base doesn't (typical case when you iterate on the system prompt), the second block keeps being served from cache even though the first one invalidates.

**When explicit is appropriate:**
- When your prompt has parts that change frequently and parts that don't. You want the parts that don't change to keep being served from cache even when the others invalidate.
- When some blocks are huge (a 50K-token knowledge base) and others small (a short system prompt). Caching both separately gives you more resilience.

**Extended TTL (1 hour):** if your calls are distributed in windows longer than 5 minutes, you can extend the TTL:

```python
"cache_control": {"type": "ephemeral", "ttl": "1h"}
```

The cache write with TTL of 1h costs 2x base (instead of 1.25x), but cache read is still 10%. If your app makes calls in windows of 30 to 60 minutes, extended TTL is clearly advantageous. If calls concentrate in bursts of minutes, the default TTL of 5 min is enough.

##### How to verify caching is working

This is the part most devs don't do and that's why they don't know if their caching works. In the response of any Anthropic API call, the `usage` object brings you:

```
"usage": {
    "input_tokens": 21,
    "cache_creation_input_tokens": 188086,
    "cache_read_input_tokens": 0,
    "output_tokens": 393
}
```

**How to read this:**
- `input_tokens`: new tokens in this request, outside cache. In this call, only the user message (21 tokens).
- `cache_creation_input_tokens`: tokens that were written to cache in this call. The first time you send a cacheable block, all its tokens go here.
- `cache_read_input_tokens`: tokens that were read from cache (at 10% of price). In the first call of the session it's at zero. In the second call with the same prefix, it should equal what was `cache_creation` the first time.
- `output_tokens`: what the model generated.

**The healthy pattern:** first call, high cache_creation, cache_read at zero. Second call (with the same prefix), cache_creation at zero or low, high cache_read. Third call, same as the second. And so on.

**If you never see cache_read different from zero**, the cache isn't activating. Common causes:
- The cacheable content is changing between requests (typically because there's a timestamp or session ID at the beginning of the prompt).
- The calls are separated by more than 5 minutes (with default TTL) and the cache expired.
- There's a bug in your app sending the prompt without the `cache_control`.

A good practice is to log `cache_read_input_tokens / (cache_read_input_tokens + input_tokens)` as a "cache hit ratio" metric in your dashboards (Section 9). If that metric drops, something broke.

##### Caching in Bedrock and Vertex (same principle, same syntax)

If you consume Claude via AWS Bedrock or Google Vertex, the syntax is the same. Anthropic maintains parity between its direct API and hyperscalers. Bedrock supports `cache_control: {"type": "ephemeral"}` just like direct API.

> 🔧 **Layer 1** (app code) · Anthropic SDK via Bedrock · App developers with EU compliance or AWS-only

```python
import boto3, json
client = boto3.client("bedrock-runtime")

body = {
    "anthropic_version": "bedrock-2023-05-31",
    "system": [{"type": "text", "text": "Reply concisely"}],
    "messages": [{"role": "user", "content": [
        {"type": "text", "text": "Describe the best way to learn programming."},
        {"type": "text", "text": "<cacheable content>",
         "cache_control": {"type": "ephemeral"}}
    ]}],
    "max_tokens": 2048
}

response = client.invoke_model(
    modelId="anthropic.claude-sonnet-4-6-20260101-v1:0",
    body=json.dumps(body)
)
```

The official documentation of each hyperscaler has the detail per model, because not all models cache everything. For Vertex the syntax is equivalent, using the Google SDK.

#### When it applies with most impact

- RAG apps with a large stable knowledge base.
- Assistants with complex system prompts (instructions, examples, tone guides, tools).
- Multi-turn agents where the first N turns stay the same.

**Typical impact case:** a RAG application with system prompt + knowledge base of ~50K tokens, called 1,000 times a day. Without caching, 50M tokens/day are processed just for setup. With caching, those 50K are charged once at the full price (cache write) and then ~999 times at 10%. The net measured savings is **84%** over the total bill for that app.

Source: [Finout, 2026](https://www.finout.io/blog/anthropic-api-pricing), [Official Anthropic Prompt Caching Docs](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)

### 5.2 History management

In agentic and chat applications the chronic problem, as we saw in Section 1.3, is that context accumulates turn by turn. There are three strategies to avoid each call ending up processing 200K tokens when 10K was enough. Each strategy has a different trade-off between memory fidelity and cost.

**Where these snippets go:** in the code that orchestrates calls to the model. Typically a module like `services/conversation_manager.py` that receives the message array, applies some strategy, and passes the processed array to the Anthropic client.

> 🔧 **Layer 1** (app code) · Python helpers over the SDK · Application developers who design agentic session management

**Strategy A: sliding window.** The simplest: keep only the last N turns and discard the old. It's fast, predictable and useful when the conversation doesn't require long-term memory (task-specific assistants, transactional chats).

```python
def get_windowed_history(messages, window_size=10):
    return messages[-window_size:] if len(messages) > window_size else messages
```

**When it's appropriate:** transactional chats, assistants that solve a question and move on.

**The risk:** if something important was mentioned at turn 3 and the window is 10, at turn 13 the model "forgets" it. For many cases that's acceptable; for others (support assistants, long debugging sessions) it's not.

**Strategy B: progressive summarization.** When the conversation grows, you take everything old (everything except the last N recent turns) and summarize it with a cheap model (Haiku ideally). The summary replaces the history in the next call. Preserves long-term memory at low cost.

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

**Note the choice of Haiku for the summary.** Using the flagship model to summarize would be throwing money away. Summarizing is a simple task where Haiku performs the same at 1/3 the price.

**When it's appropriate:** assistants that need to remember the general context of a long session (support, debugging, planning) without paying the cost of re-sending everything.

**Strategy C: Compaction API (beta January 2026, ZDR-eligible).** This is the cleanest option and the recommended one for production agents. Instead of implementing your own summarization logic, Anthropic provides a beta API that takes care of compaction automatically when context exceeds a threshold. You define the threshold, the system decides when and how to compact.

```python
response = client.beta.messages.create(
    betas=["compact-2026-01-12"],
    model="claude-sonnet-4-6",
    max_tokens=4096,
    messages=messages,
    context_management={"edits": [{"type": "compact_20260112", "context_token_threshold": 50_000}]}
)
```

**What it does:** when context exceeds 50K tokens, Anthropic automatically compacts the old part of the history using an optimized process that retains relevant information and discards noise.

**When it's appropriate:** production agents where you don't want to maintain your own compression logic. It's ZDR-eligible (Zero Data Retention), so it doesn't add compliance complications.

### 5.3 RAG and output budgets

These are two complementary practices that almost all teams in production end up adopting. Worth understanding each separately.

> 🔧 **Layer 1** (app code) · RAG pipeline + budget dictionary · Application developers designing the retrieval flow and caps per task type

**RAG (Retrieval-Augmented Generation).** Instead of putting an entire knowledge base in the system prompt and paying for it every time, you index it in a vector store and retrieve only the relevant fragments for each query. The model receives only what it needs to respond, not the whole corpus.

```python
chunks = vector_store.search(query=user_query, top_k=5)
context = "\n\n".join([c.text[:500] for c in chunks])
```

**What happens.** Instead of sending 50K tokens of knowledge base to the model, you send the 5 most relevant fragments (~2,500 tokens). The model responds with the same quality because the rest of the corpus wasn't pertinent to that specific query.

The typical impact is high: **RAG reduces tokens by up to 70%** compared to "context stuffing" (putting all documentation in the system prompt). The operational trick is fine-tuning `top_k` and max length per chunk: 5 chunks of 500 characters is enough for most cases; raise them only if the model demonstrates a lack of context.

Source: [Koombea, 2026](https://ai.koombea.com/blog/llm-cost-optimization)

**Output budgets.** As we saw in Section 1.1, output tokens cost 5x more than input. The operational consequence is that **limiting the length of the response has a disproportionate impact on cost**. The standard way to do it is not to use a single `max_tokens` for the whole app, but to calibrate the budget according to task type:

```python
OUTPUT_BUDGETS = {
    "classification": 50,
    "extraction": 200,
    "summarization": 500,
    "code_review": 1000,
    "code_generation": 2000,
    "architecture": 3000
}
```

**How to apply it.** When you build each `messages.create()`, instead of passing a generic `max_tokens=4096`, you pass the budget corresponding to the task type: `max_tokens=OUTPUT_BUDGETS["classification"]` for classifications, etc.

The idea is that a binary classification never needs 1,000 output tokens (probably 5 is enough), an entity extraction shouldn't exceed 200, and only tasks that justify long answers (code generation, architecture design) get big budgets. This prevents the model from "expanding" when it's not necessary, without losing capability when it is.

Combined with RAG, the complete flow becomes: bounded retrieval, small context, output budget bounded to task type. Result: predictable bill an order of magnitude smaller.

---

## 6. Level 2: model selection and assignment <a id="6-level-2"></a>

This level resolves a question that seems trivial but is rarely addressed with rigor: **which model do you use for each task?** The default answer in most teams is "the flagship for everything", and it's exactly the mistake that inflates the bill.

The price difference between models from the same provider is huge (Haiku $1/$5 vs Opus $5/$25, that is 5x), and for many tasks Haiku performs the same or almost the same as Opus. Assigning the correct model per task type is probably the highest cost/effort ratio adjustment in the document.

> 🔧 **Layer 1** (app code) or **Layer 4** (in gateway if centralized) · Assignment dictionary + classifier router · Application developers, or platform if moved to shared gateway

The strategy is simple: keep a dictionary of which task type goes to which model, and a cheap classifier (Haiku) that decides for each incoming request which type it belongs to. The classifier adds a marginal cost (50 output tokens at Haiku price, that is fractions of a cent per request) but allows correctly routing all subsequent traffic.

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
               "JUST the name.",
        messages=[{"role": "user", "content": user_input}]
    )
    task_type = result.content[0].text.strip().lower()
    return task_type, MODEL_ASSIGNMENT.get(task_type, "claude-sonnet-4-6")
```

**How to read the dictionary:**
- **Haiku** for mechanical tasks: classification, entity extraction, data formatting, file navigation, simple Q&A. They are tasks where the quality difference between models is marginal and the savings is 5x.
- **Sonnet** for the bulk of development work: code generation and review, summaries, general chat, debugging. It's the price/quality sweet spot and should be the default when the task can't be clearly classified as simple or very complex.
- **Opus** only for the truly complex: architecture design, multi-step reasoning, coordination between agents. If your application spends the bulk of the budget on Opus, there's a high probability you're over-qualifying tasks.

**A note on the fallback:** the last line (`MODEL_ASSIGNMENT.get(task_type, "claude-sonnet-4-6")`) makes it so if the classifier returns something unexpected, it falls back to Sonnet by default. It's the safe value: when in doubt, Sonnet. Never fall back to Opus by default, that should always be an explicit and justified decision.

**Implementation at scale:** When there are several services, this dictionary and the routing logic are exactly what's worth moving to the internal wrapper or the gateway (Section 2.4). That way the "assignment table" is a single one, governable and auditable.

---

## 7. Level 3: automatic model routing <a id="7-level-3"></a>

Level 2 solves assignment with explicit rules: a hand-made dictionary that maps task types to models. It works very well when categories are clear and the domain is stable.

Level 3 takes the idea one step further: **let a router automatically learn which requests require the flagship model and which can be resolved by a cheaper model**, without needing to define the rules by hand. It's what's known as "model routing" or "cascading" in the literature, and the published results are notable.

### 7.1 RouteLLM (ICLR 2025, UC Berkeley/LMSYS)

RouteLLM is an open source system published at ICLR 2025 by UC Berkeley and LMSYS. The central idea: train a router (a small classifier) that, for each new request, decides whether the "strong model" (flagship) or the "weak model" (a cheaper one) solves it. The router learns from previous human comparisons which prompt characteristics predict that the cheap model will be sufficient.

The published results: **95% of GPT-4 quality** using just **26% of the calls** routed to the strong model. That is, the remaining 74% is solved with the cheap model with no measurable loss of quality. The **measured net savings is 48%**, and with augmentation techniques it can reach **75%**.

Source: [LMSYS ICLR 2025](https://www.lmsys.org/blog/2024-07-01-routellm/)

> 🔧 **Layer 4** (gateway / routing layer) · UC Berkeley/LMSYS open source · Platform or infra team that places a layer between services and providers

```python
from routellm.controller import Controller
client = Controller(
    routers=["mf"], strong_model="anthropic/claude-opus-4-7",
    weak_model="anthropic/claude-haiku-4-5", config={"mf": {"threshold": 0.3}}
)
response = client.chat.completions.create(model="router", messages=[...])
```

**The key parameter is the threshold.** It defines how aggressive the router is. A low threshold (0.1) sends more things to the strong model (more quality, less savings). A high threshold (0.5+) sends more things to the weak model (more savings, quality risk). The paper recommends calibrating it by measuring quality on a representative subset of your traffic.

**When RouteLLM is appropriate:** when you already have sufficient volume (tens of thousands of requests/day) and the Level 2 heuristic falls short because your traffic is more diverse than the categories you could define. It's a natural step-up over explicit assignment: from the hand-made dictionary to the trained model.

### 7.2 Enterprise AI Gateways

Beyond RouteLLM as a specific solution, in enterprise production **AI gateways** dominate: a network layer all LLM calls go through that centralizes routing, retries, fallbacks, per-team budgets, guardrails and observability. It's the equivalent of a traditional API gateway, but specific for LLMs.

> 🔧 **Layer 4** (shared gateway) · Self-hosted LiteLLM / managed Portkey · Platform or infra team. The code of consumer services does NOT change, the gateway intercepts

```python
# LiteLLM: self-hosted, budget controls
from litellm import Router
router = Router(
    model_list=[
        {"model_name": "prod", "litellm_params": {"model": "claude-sonnet-4-6"}, "tpm": 100000},
        {"model_name": "prod", "litellm_params": {"model": "azure/claude-sonnet-4-6"}, "tpm": 100000}
    ],
    budget_manager={"type": "redis", "redis_url": os.getenv("REDIS_URL")}
)

# Portkey: managed, compliance
from portkey_ai import Portkey
client = Portkey(api_key="PORTKEY_API_KEY", virtual_key="ANTHROPIC_VIRTUAL_KEY")
response = client.chat.completions.create(
    model="claude-sonnet-4-6", messages=messages,
    metadata={"team": "engineering", "feature": "code-review"}
)
```

There are three main players in this space and it's worth understanding how they differ:

| Gateway | Type | Best for |
|---------|------|------------|
| LiteLLM | Open source | Total control, zero fees |
| Portkey | Managed | Compliance, guardrails, SOC2 |
| OpenRouter | SaaS | Fast experimentation |

**LiteLLM** is self-hosted (it's open source), doesn't add fees to token cost, and gives total control. Has built-in budget manager with Redis, multi-provider routing, automatic fallbacks. It's the typical choice for teams that want a gateway without adding another vendor to the stack.

**Portkey** is managed (SaaS) and specializes in compliance: SOC2, guardrails, virtual keys (separate credentials per team without exposing the real API key), complete audit logs. Charges additional fee on traffic. It's the typical choice for companies with serious compliance requirements that value not operating a gateway.

**OpenRouter** is the option for experimentation: huge catalog of models (100+ providers), single API key, pays the cost of each provider plus a small margin. Useful for POCs and for discovering which new model is worth trying, not so much for enterprise production due to compliance and observability issues.

**Operational rule:** if the team is platform and doesn't want to add vendors, LiteLLM. If compliance is the priority, Portkey. OpenRouter rarely ends up being the production choice at scale, but it's excellent for the discovery phase.

---

## 8. Level 4: cost infrastructure and governance <a id="8-level-4"></a>

This level gathers three adjustments that have in common not being prompt or model optimizations, but **infrastructure and process decisions** that change the cost structure. Each has its logic:

1. **Batch API.** For non-realtime jobs, half price with no quality loss.
2. **DeepSeek via Azure.** For massive cheap tasks with European compliance.
3. **Stoppers and observability.** So cost doesn't overflow without warning.

### 8.1 Batch API (50% discount, identical quality)

A good portion of LLM consumption in companies doesn't need to respond in real time. Processing 50,000 documents to extract entities, generating embeddings for a corpus, evaluating thousands of responses against a rubric, doing massive classification of historical tickets: all those cases can wait minutes or hours to return results.

For those cases, Anthropic (and OpenAI) offer Batch API: calls are processed asynchronously when capacity is available, with an SLA of up to 24 hours, **at 50% of normal price**. There's no quality loss: it's exactly the same model, just with a different latency channel.

> 🔧 **Layer 1** (app code) · Anthropic Batches API SDK · Application developers processing non-realtime jobs (ETL, evaluations, mass generation)

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

Note an additional optimization in the snippet: the `cache_control: {"type": "ephemeral"}` applied to the system prompt. When you combine Batch + caching, the discounts accumulate multiplicatively: 50% from batch over 10% from cache read. **Documented result: up to 95% savings** compared to sending each doc separately in realtime mode without cache. Source: [PECollective, 2026](https://pecollective.com/tools/anthropic-api-pricing/)

**When it applies:**
- Nightly ETL pipelines.
- Massive model evaluation (eval harnesses).
- Synthetic dataset generation.
- Historical data reprocessing.

**When it does NOT apply:** anything that has the user waiting for a response on screen. Because the minimum latency is minutes, not seconds.

### 8.2 DeepSeek via Azure

This is one of the few cases in this document where the hyperscaler choice is correct **for model reasons, not general procurement** (see Section 3 for the general rule).

> 🔧 **Layer 1** (app code) · OpenAI SDK pointing to Azure endpoint · Application developers processing massive cheap tasks with EU compliance

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

**Procurement context:** DeepSeek V4-Flash is approximately **15x cheaper than Sonnet** in nominal price (~$0.14/MTok input vs $3.00 for Sonnet), with European compliance ensured by being served by Azure. That is, it gives you reasonable quality for mechanical tasks at a price close to the cheapest models on the market.

Direct DeepSeek access (`api.deepseek.com`) **is not viable for companies with EU data** because it routes all requests through Chinese servers, which clashes head-on with any serious GDPR or data residency requirement.

Azure adds a markup of **20 to 35% over the direct price** ([DeployBase, 2026](https://deploybase.ai/articles/deepseek-v3-pricing)), but **that markup is not general overhead, it's a compliance premium**: data stays in the Azure region you chose (EU if needed), the contract is with Microsoft, and European compliance is covered. Even with that 20 to 35% on top, DeepSeek via Azure still costs approximately **1/15 of Sonnet**, so for massive classification, structured extraction or offline batch processing, it's still the best price/compliance ratio on the market.

**Critical note:** This is one of the few cases where the hyperscaler is the correct choice for model reasons, not general procurement. For Claude or GPT, the same calculation doesn't apply (see Section 3 on nominal pricing parity). The rule is: **if the model itself isn't accessible directly with compliance, the hyperscaler is worth the markup; if the model is already available directly from the vendor with compliance, the hyperscaler adds overhead without adding value.**

### 8.3 Stoppers and cost observability

Here it's worth dismantling a myth first before explaining the how. The mental image of "they're going to overcharge you without warning, you're going to get an exploded bill at month-end" is exaggerated. **By default, no serious provider blows up your bill.** What does happen is that each tool has its own behavior when reaching the limit, and it's worth knowing each one's to decide if you want to change it.

#### How each tool behaves by default

All platforms have at least one of these three braking mechanisms before the bill grows out of control:

1. **Provider rate limits.** A hard ceiling of requests or tokens per minute that the vendor applies by default. You don't configure it. It stops you before any explosion, returning a "rate limit exceeded" type error.
2. **Plan quota.** What you paid for gives you X consumption. When it runs out, it returns "limit reached" and you have to wait for the reset (monthly typically) or pay more.
3. **Configurable spending limit.** You define how much you're willing to spend on top of the plan. If you leave it at zero (the reasonable default), there's no overage. If you raise it, then there's risk controlled by you.

The following table summarizes the real behavior per tool. Note that **what we called "stopper" in the "Has stopper by default" row doesn't mean "it cuts you off without options", it means "it doesn't accept consumption beyond the plan/quota unless you configure otherwise"**.

| Tool | Will it blow up your bill by default? | What happens when you reach the limit |
|---|---|---|
| **Claude Pro / Team / Enterprise (Desktop, Code, Cowork)** | No. | Cuts you off cleanly when the plan quota runs out. No overage. You wait for the monthly reset or upgrade. |
| **Cursor** | No. | Plan cap. To consume beyond, you have to activate "additional usage" explicitly from org settings. |
| **GitHub Copilot (new billing June 2026)** | No, if properly configured. | The seat brings AI Credits included. When they run out, depends on the **spending limit** the organization admin configured: if at zero (reasonable default), cuts off; if at X, consumes up to that cap and then cuts off. |
| **Anthropic direct API** | No. | Each workspace has a configurable **monthly spend limit**. While you don't touch it, there's a default value. When you reach it, returns error. Per-minute rate limits stop any runaway before. |
| **OpenAI direct API** | No. | Same model. Settings → Limits dashboard brings a **monthly budget** and configurable alerts. |
| **Azure OpenAI / AWS Bedrock** | Depends on how you configure it. | The bill goes to the general cloud account. It's not "open tap" (the model rate limits and subscription quotas stop you before), but you could end up paying more than expected if you didn't set up **budget alerts** in Azure Cost Management or AWS Budgets. They need to be configured explicitly. |

#### Where to configure each thing

| Tool | Setting to touch | Path |
|---|---|---|
| **GitHub Copilot** | Spending limit per organization | Organization admin in GitHub, Billing & Plans, "spending limit" |
| **Cursor** | Additional usage | Org settings, deactivate overage |
| **Anthropic API** | Workspace spend limit | Anthropic console, Limits, Workspace spend limit |
| **OpenAI API** | Monthly budget | OpenAI Dashboard, Settings, Limits, Monthly budget |
| **Azure OpenAI** | Budget alert | Azure Cost Management, Budgets, alerts at X% of budget |
| **AWS Bedrock** | Budget alert | AWS Budgets, alerts at X% of budget |

#### The operational conclusion

The correct message is not "they're going to overcharge you without you noticing". It's:

- **By default, serious tools stop you when you reach the limit.** You'll see "limit reached" or "rate limit exceeded" before the bill overflows.
- **If you want the tool to keep working beyond the plan**, you choose to activate overage or raise the spending limit. That's a conscious decision, not an accident.
- **Extra care applies in Azure/Bedrock**: since the bill goes to the general cloud account, it's worth always configuring budget alerts. Not so they cut off (they don't), but so they warn you before spending goes outside what was budgeted.
- **The stopper is not a solution, it's a fire alarm.** If your bill reaches the cap, something broke before. The question isn't "when will they cut us off?", it's "who's watching the trajectory and warning before the cap gets close?".

For each tool your BU uses, the operational exercise is:
1. Go to the corresponding admin console and look at how the spending limit/budget is configured today.
2. Confirm that the default is "don't consume beyond the plan without explicit authorization" or consciously decide to raise the cap with a value that makes sense for your budget.
3. Configure alerts at 50%, 80% and 100% so someone watches the trajectory without waiting for the cutoff.
4. Document who's responsible for reviewing those caps monthly.

#### Custom budget monitor (for apps with direct API)

If you build apps that call the API directly, in addition to the vendor's spending limit you can instrument a **budget monitor** in your own code that tracks spending per user/team and alerts progressively.

> 🔧 **Layer 1** (app code) or **Layer 4** (centralized gateway) · Python helper over the SDK or gateway middleware · Application developers or platform

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

**Three important points about this pattern:**

1. **Thresholds are graduated.** At 50% you warn so the user has visibility; at 80% so they act; at 100% so they escalate or stop. Warning only at 100% is too late.
2. **`alerted` avoids spam.** Once a threshold is crossed, no more notifications. Users disable notifications that hit them a hundred times a day.
3. **Where you place it matters.** If you put it in each service, each team has its own monitor (drift, inconsistent data). If you put it in the shared gateway (Layer 4), a single implementation governs all enterprise traffic. The larger the organization, the more attractive the gateway option.

At enterprise scale, this basic pattern is complemented with OTel (Section 9) to have unified metrics, traces and logs, and dashboards that show consumption per team, per model, per feature.

---

## 9. Level 5: observability with OpenTelemetry and personal governance <a id="9-otel"></a>

If the previous levels are about **saving** tokens, this level is about **knowing what's happening**. Without observability, all optimizations are blind bets: you can't claim caching is working, you don't know which team spends more, you can't detect cost regressions when they're introduced.

OpenTelemetry (OTel) has become the de facto standard for AI observability. It has three advantages over proprietary alternatives: it's vendor-neutral (data doesn't get trapped in a SaaS), has native support in the CLIs of the main providers (Claude Code, Copilot) and integrates with any backend (Grafana, Prometheus, Jaeger, Langfuse, etc.).

This section has two audiences: **individuals** who want to see their own consumption (Section 9.6) and **platforms** that want to observe the whole company (Sections 9.1 to 9.5). Both share the same technology, only scale changes.

### 9.1 OTel in Claude Code (CLI)

Claude Code exposes complete OTel telemetry just by activating environment variables. No need to install anything extra or modify the CLI.

**Where to set these variables.** You can put them in two ways:
- **Local to your shell:** add them to your `~/.zshrc` or `~/.bashrc` (Mac/Linux) or to user environment variables in Windows. They stay active in any terminal you open from then on.
- **Enterprise via MDM:** an admin distributes them automatically to all devs (see Section 9.5).

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

**What each variable does:**
- `CLAUDE_CODE_ENABLE_TELEMETRY=1` activates the telemetry subsystem. Without this, the others do nothing.
- `OTEL_METRICS_EXPORTER=otlp`, `OTEL_LOGS_EXPORTER=otlp`, `OTEL_TRACES_EXPORTER=otlp` tell Claude Code to export metrics, logs and traces using OTLP protocol (the OpenTelemetry standard).
- `OTEL_EXPORTER_OTLP_ENDPOINT` is the address of the backend the data is sent to. `localhost:4317` means there's a backend running on your own machine (typically a container, see Section 9.3). For enterprise, this points to an internal collector (`collector.internal:4317`).
- `OTEL_EXPORTER_OTLP_PROTOCOL=grpc` defines the transport format. gRPC is the most efficient; you can also use `http/protobuf` if gRPC has issues.
- `OTEL_METRIC_EXPORT_INTERVAL=10000` exports metrics every 10 seconds (10000 ms). Below this you overload the backend; above you lose granularity.
- `OTEL_RESOURCE_ATTRIBUTES` tags the data. It's the most useful of all for enterprise: it lets you filter by `team.id` and `department` in the dashboard, comparing consumption between teams. When applied via MDM, this variable is automatically set according to the team the dev belongs to.

**What metrics you'll see.** Claude Code emits six types of metrics, with textual names (these are the real names you'll search in Grafana or Prometheus):

- `claude_code_token_usage`: broken down into `input`, `output`, `cache_creation` and `cache_read`. The key metric to know how much repeated context you're re-sending.
- `claude_code_cost_usage`: cost in USD per API call.
- `claude_code_session_count`: number of sessions you opened.
- `claude_code_active_time_total`: active Claude Code time.
- `claude_code_lines_of_code_count`: lines of code added and removed.
- `claude_code_code_edit_tool_decision`: accept/reject of suggestions, broken down by language.

### 9.2 OTel in Copilot Chat

For Copilot, activation is by settings, simpler than in Claude Code. It has two modes of exporting: to an OTLP endpoint (a server or container that receives metrics) or **directly to a local JSONL file on your own machine**. For individual use the local file is the simplest and what we'll use in Section 9.6.

**Local file mode (recommended for personal use):**

```json
{
    "github.copilot.chat.otel.enabled": true,
    "github.copilot.chat.otel.exporterType": "file",
    "github.copilot.chat.otel.outfile": "/Users/your-user/.copilot-otel.jsonl"
}
```

With this, each interaction with Copilot Chat is written as a JSON line to the file. Nothing leaves your machine. The file fills itself as you use Copilot normally.

**OTLP endpoint mode (to connect to a container or backend):**

```json
{
    "github.copilot.chat.otel.enabled": true,
    "github.copilot.chat.otel.exporterType": "otlp",
    "github.copilot.chat.otel.otlpEndpoint": "http://localhost:4317"
}
```

**Attributes it exports.** Copilot Chat follows the **GenAI Semantic Conventions** of OpenTelemetry, an open standard for naming LLM telemetry attributes. The main ones you'll see:
- `gen_ai.request.model`: which model was invoked.
- `gen_ai.provider.name`: which provider (anthropic, openai, etc.).
- `gen_ai.tool.name`: which tool was executed.
- `copilot_chat.edit.source`: where an edit came from.
- Token counts and durations per each span.

**Important note:** OTel monitoring comes **off by default** in Copilot Chat. It has to be activated explicitly. It's fine that it's that way because it gives you control of what's exported and to where, especially important for privacy: by default only metadata (tokens, model, durations) is exported, not the content of prompts or responses. If you want to capture complete content:

```json
{ "github.copilot.chat.otel.captureContent": true }
```

And vendor-side clarification: "Content capture can include sensitive information such as code, file contents, and user prompts". Handle with care.

Both Claude Code and Copilot emit to the OTLP endpoint, so you can have **a single backend that consolidates observability of all the team's AI products**, regardless of which tool each dev is using.

### 9.3 Local stack in 30 seconds

To experiment and validate capture, no complex infrastructure needs to be set up. Microsoft's Aspire Dashboard is a container that starts a complete OTLP backend (traces, metrics, logs) with a navigable UI, all in a single `docker run`.

> 🔧 **Layer 2** (dev workstation) · Aspire Dashboard Docker container · Individual dev for local use without cloud account

```bash
docker run --rm -d -p 18888:18888 -p 4317:18889 --name aspire-dashboard \
  mcr.microsoft.com/dotnet/aspire-dashboard:latest
```

**What this command does:**
- `--rm` deletes the container when you stop it (no old containers accumulate).
- `-d` runs it in the background (detached).
- `-p 18888:18888` maps port 18888 (web UI) to your machine.
- `-p 4317:18889` maps port 4317 (OTLP endpoint) to your machine.
- `--name aspire-dashboard` gives it a name for easy stop later (`docker stop aspire-dashboard`).
- The last line is Microsoft's official image, maintained and free.

**How you use it:**
1. Run the command.
2. Open http://localhost:18888 in the browser.
3. Set the environment variables from Section 9.1 with `OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317`.
4. Use Claude Code normally. Each request you make will appear in the dashboard in real time.

It's the fastest way for a dev to locally test "what it looks like" in AI observability before investing in production infrastructure.

### 9.4 Production stack (Grafana + Prometheus)

For enterprise, the most common stack is OTel Collector + Prometheus for metrics + Grafana for dashboards. The three are open source, widely adopted, and typically orchestrated with docker-compose.

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

Once up, Claude Code/Copilot traces and metrics flow to the collector (port 4317), Prometheus collects them (with 90 days of retention in this example), and Grafana visualizes them. Typical dashboards for AI cost observability include: tokens per team/day, cost per feature, cache hit/miss ratio, distribution of models per task, latencies per endpoint, error rate.

### 9.5 Centralized configuration (MDM)

At enterprise scale you can't ask each developer to configure their shell with OTel variables. The standard way to do it is to distribute the configuration via MDM (Intune for Windows, Jamf for Mac). A single push policy applies to all devs and stays governed centrally.

> 🔧 **Layer 3** (enterprise managed settings) · JSON distributable via MDM (Intune, Jamf) · Developer platform admin

```json
{ "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1", "OTEL_METRICS_EXPORTER": "otlp",
    "OTEL_LOGS_EXPORTER": "otlp", "OTEL_EXPORTER_OTLP_PROTOCOL": "grpc",
    "OTEL_EXPORTER_OTLP_ENDPOINT": "http://collector.internal:4317",
    "OTEL_EXPORTER_OTLP_HEADERS": "Authorization=Bearer <token>"
}}
```

With this, **all developers in the group are automatically instrumented**, data travels to an internal collector authenticated with bearer token, and from day one you have real visibility of how AI is being used across the organization. It's the foundation that enables optimization decisions from previous levels: without enterprise observability, decisions are bets.

### 9.6 Personal governance: your own dashboard in 10 minutes, all on your machine

Sections 9.1 to 9.5 are designed for platform. This section is different: it's for **any person who uses AI every day** and wants to see their own consumption, without waiting for the platform team to set up infrastructure, without paying anything extra, without uploading metrics to any external service.

The idea: with a small Python script and a static HTML, in 10 minutes any dev can have their personal dashboard that answers:

- How many sessions did I open this week?
- How much repeated context am I re-sending (cache hit ratio)?
- What models am I using, in what proportion?
- How many tokens were input vs output?
- How much does it cost me each day?

#### Path A (recommended): Python script + local static HTML

This is the lightest option and the one that respects the "nothing leaves my machine" principle. The conceptual flow is simple:

1. **Activate export to local file** in the tools (environment variables for Claude Code, JSON setting for Copilot Chat). The file fills itself as you use the tools normally.
2. **Run the Python script** once a week (or whenever you want a snapshot). It reads the file, computes aggregations, writes a `dashboard.html` in the same folder.
3. **Open the HTML in the browser.** See the charts. Look. Adjust your use if needed.

No docker. No running server. No cloud account. Zero euros. The script reads from a plain text file and writes to an HTML file, that's it.

##### Step-by-step setup

**Step 1. Activate export in Claude Code.**

In your `~/.zshrc` (Mac/Linux) or environment variables (Windows):

```bash
export CLAUDE_CODE_ENABLE_TELEMETRY=1
export OTEL_LOGS_EXPORTER=otlp
export OTEL_METRICS_EXPORTER=otlp
export OTEL_EXPORTER_OTLP_PROTOCOL=http/protobuf
# Claude Code doesn't yet support direct file export;
# for personal use the simplest is to spin up Aspire Dashboard locally
# (see Path B) or use the claude --debug command that logs to stderr.
```

> Note: Claude Code at the time of writing this doc doesn't expose an "export to file" mode as direct as Copilot's. The cleanest alternative for personal use is to use Path B (local Aspire) or run Claude Code with `--debug` and redirect stderr to a file. For Copilot Chat, Path A works straight.

**Step 2. Activate export in Copilot Chat.**

In VSCode `settings.json`:

```json
{
    "github.copilot.chat.otel.enabled": true,
    "github.copilot.chat.otel.exporterType": "file",
    "github.copilot.chat.otel.outfile": "/Users/your-user/.copilot-otel.jsonl"
}
```

Replace `/Users/your-user/` with your real home (`$HOME` on Mac/Linux, `%USERPROFILE%` on Windows). From then on, every time you use Copilot Chat, a JSON line will be written to the file. Without you having to do anything.

**Step 3. The Python script that builds the dashboard.**

Save it as `dashboard.py` in any folder (ideally the same one as the JSONL or a separate project):

```python
#!/usr/bin/env python3
"""
Personal Copilot Chat / Claude Code consumption dashboard.
Reads the JSONL exported by the tools and generates a static HTML
with charts. Zero external dependencies (just stdlib + Chart.js via CDN).

Usage: python dashboard.py /path/to/copilot-otel.jsonl
"""
import json
import sys
import os
from collections import defaultdict
from datetime import datetime
from pathlib import Path

# --- Reference prices (USD per million tokens, May 2026) ---
PRICES = {
    "claude-haiku-4-5":    {"input": 1.00, "output": 5.00,  "cache_read": 0.10},
    "claude-sonnet-4-6":   {"input": 3.00, "output": 15.00, "cache_read": 0.30},
    "claude-opus-4-7":     {"input": 5.00, "output": 25.00, "cache_read": 0.50},
    "gpt-5-nano":          {"input": 0.05, "output": 0.40,  "cache_read": 0.00},
    "gpt-5.4":             {"input": 2.50, "output": 15.00, "cache_read": 0.25},
    "gpt-5.5":             {"input": 5.00, "output": 30.00, "cache_read": 0.50},
}

def calc_cost(model, input_t, output_t, cache_read_t=0):
    """Calculate cost in USD for a given call."""
    p = PRICES.get(model.lower(), PRICES["claude-sonnet-4-6"])  # reasonable fallback
    return (input_t * p["input"] + output_t * p["output"] + cache_read_t * p["cache_read"]) / 1_000_000

def parse_jsonl(path):
    """Read the JSONL and return a list of normalized events."""
    events = []
    with open(path, "r", encoding="utf-8") as f:
        for line in f:
            line = line.strip()
            if not line:
                continue
            try:
                evt = json.loads(line)
                # GenAI Semantic Conventions use these attributes:
                attrs = evt.get("attributes", {})
                model = attrs.get("gen_ai.request.model", "unknown")
                input_t = int(attrs.get("gen_ai.usage.input_tokens", 0))
                output_t = int(attrs.get("gen_ai.usage.output_tokens", 0))
                cache_read_t = int(attrs.get("gen_ai.usage.cache_read_input_tokens", 0))
                # Timestamp can come in several formats
                ts_str = evt.get("timestamp") or evt.get("time") or attrs.get("timestamp")
                if ts_str:
                    try:
                        ts = datetime.fromisoformat(ts_str.replace("Z", "+00:00"))
                    except Exception:
                        ts = datetime.now()
                else:
                    ts = datetime.now()
                events.append({
                    "ts": ts,
                    "day": ts.strftime("%Y-%m-%d"),
                    "model": model,
                    "input_tokens": input_t,
                    "output_tokens": output_t,
                    "cache_read_tokens": cache_read_t,
                    "cost_usd": calc_cost(model, input_t, output_t, cache_read_t),
                })
            except (json.JSONDecodeError, KeyError, ValueError):
                continue
    return events

def aggregate(events):
    """Compute aggregations for the dashboard."""
    sessions_per_day = defaultdict(int)
    tokens_per_day = defaultdict(lambda: {"input": 0, "output": 0, "cache_read": 0})
    cost_per_day = defaultdict(float)
    tokens_per_model = defaultdict(int)
    total_input = total_output = total_cache_read = 0

    for e in events:
        sessions_per_day[e["day"]] += 1
        tokens_per_day[e["day"]]["input"] += e["input_tokens"]
        tokens_per_day[e["day"]]["output"] += e["output_tokens"]
        tokens_per_day[e["day"]]["cache_read"] += e["cache_read_tokens"]
        cost_per_day[e["day"]] += e["cost_usd"]
        tokens_per_model[e["model"]] += e["input_tokens"] + e["output_tokens"]
        total_input += e["input_tokens"]
        total_output += e["output_tokens"]
        total_cache_read += e["cache_read_tokens"]

    cache_hit_ratio = (total_cache_read / (total_cache_read + total_input)) if (total_cache_read + total_input) > 0 else 0
    return {
        "sessions_per_day": dict(sorted(sessions_per_day.items())),
        "tokens_per_day": dict(sorted(tokens_per_day.items())),
        "cost_per_day": dict(sorted(cost_per_day.items())),
        "tokens_per_model": dict(tokens_per_model),
        "totals": {
            "input": total_input, "output": total_output, "cache_read": total_cache_read,
            "cost": sum(cost_per_day.values()),
            "cache_hit_ratio": cache_hit_ratio,
        },
    }

HTML_TEMPLATE = """<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>My AI consumption</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<style>
  body { font-family: -apple-system, system-ui, sans-serif; max-width: 1100px; margin: 2em auto; padding: 0 1em; color: #222; }
  h1 { border-bottom: 2px solid #333; padding-bottom: .3em; }
  .totals { display: grid; grid-template-columns: repeat(4, 1fr); gap: 1em; margin: 2em 0; }
  .card { background: #f5f5f5; padding: 1em; border-radius: 6px; text-align: center; }
  .card .v { font-size: 1.8em; font-weight: bold; color: #1a5d99; }
  .card .l { font-size: .9em; color: #666; }
  .chart-row { display: grid; grid-template-columns: 1fr 1fr; gap: 2em; margin: 2em 0; }
  .chart-full { margin: 2em 0; }
  canvas { max-height: 320px; }
</style>
</head>
<body>
<h1>My AI consumption</h1>
<p>Generated: __GENERATED__. Events analyzed: <strong>__EVENT_COUNT__</strong>.</p>

<div class="totals">
  <div class="card"><div class="v">__TOTAL_INPUT__</div><div class="l">input tokens</div></div>
  <div class="card"><div class="v">__TOTAL_OUTPUT__</div><div class="l">output tokens</div></div>
  <div class="card"><div class="v">__CACHE_HIT__%</div><div class="l">cache hit ratio</div></div>
  <div class="card"><div class="v">$__TOTAL_COST__</div><div class="l">total estimated cost</div></div>
</div>

<div class="chart-full"><h2>Sessions per day</h2><canvas id="sessions"></canvas></div>
<div class="chart-row">
  <div><h2>Cost per day (USD)</h2><canvas id="cost"></canvas></div>
  <div><h2>Models used (total tokens)</h2><canvas id="models"></canvas></div>
</div>
<div class="chart-full"><h2>Input vs output tokens per day</h2><canvas id="tokens"></canvas></div>

<script>
const DATA = __DATA_JSON__;

new Chart(document.getElementById('sessions'), {
  type: 'bar',
  data: { labels: Object.keys(DATA.sessions_per_day), datasets: [{ label: 'Sessions', data: Object.values(DATA.sessions_per_day), backgroundColor: '#1a5d99' }] },
  options: { responsive: true }
});

new Chart(document.getElementById('cost'), {
  type: 'line',
  data: { labels: Object.keys(DATA.cost_per_day), datasets: [{ label: 'USD', data: Object.values(DATA.cost_per_day), borderColor: '#c0392b', fill: false, tension: 0.2 }] },
  options: { responsive: true }
});

new Chart(document.getElementById('models'), {
  type: 'doughnut',
  data: { labels: Object.keys(DATA.tokens_per_model), datasets: [{ data: Object.values(DATA.tokens_per_model), backgroundColor: ['#1a5d99','#c0392b','#27ae60','#f39c12','#8e44ad','#16a085'] }] },
  options: { responsive: true }
});

const days = Object.keys(DATA.tokens_per_day);
new Chart(document.getElementById('tokens'), {
  type: 'bar',
  data: {
    labels: days,
    datasets: [
      { label: 'Input', data: days.map(d => DATA.tokens_per_day[d].input), backgroundColor: '#1a5d99' },
      { label: 'Output', data: days.map(d => DATA.tokens_per_day[d].output), backgroundColor: '#c0392b' },
      { label: 'Cache read', data: days.map(d => DATA.tokens_per_day[d].cache_read), backgroundColor: '#27ae60' }
    ]
  },
  options: { responsive: true, scales: { x: { stacked: true }, y: { stacked: true } } }
});
</script>
</body>
</html>
"""

def render_html(agg, event_count, out_path):
    """Generate the static HTML."""
    t = agg["totals"]
    html = (HTML_TEMPLATE
        .replace("__GENERATED__", datetime.now().strftime("%Y-%m-%d %H:%M"))
        .replace("__EVENT_COUNT__", str(event_count))
        .replace("__TOTAL_INPUT__", f"{t['input']:,}")
        .replace("__TOTAL_OUTPUT__", f"{t['output']:,}")
        .replace("__CACHE_HIT__", f"{t['cache_hit_ratio']*100:.0f}")
        .replace("__TOTAL_COST__", f"{t['cost']:.2f}")
        .replace("__DATA_JSON__", json.dumps(agg))
    )
    with open(out_path, "w", encoding="utf-8") as f:
        f.write(html)

def main():
    if len(sys.argv) < 2:
        print("Usage: python dashboard.py /path/to/copilot-otel.jsonl")
        sys.exit(1)
    jsonl_path = Path(sys.argv[1]).expanduser()
    if not jsonl_path.exists():
        print(f"Doesn't exist: {jsonl_path}")
        sys.exit(1)
    events = parse_jsonl(jsonl_path)
    if not events:
        print("The file doesn't have parseable events yet. Use Copilot Chat a bit more and run again.")
        sys.exit(0)
    agg = aggregate(events)
    out_path = jsonl_path.parent / "dashboard.html"
    render_html(agg, len(events), out_path)
    print(f"Dashboard generated: {out_path}")
    print(f"Open with: open {out_path}  (Mac)  or  xdg-open {out_path}  (Linux)  or  start {out_path}  (Windows)")

if __name__ == "__main__":
    main()
```

**Step 4. Run the script.**

After using Copilot a couple of days so the JSONL has data:

```bash
python dashboard.py ~/.copilot-otel.jsonl
```

This generates a `dashboard.html` in the same folder as the JSONL. Open it with double click or from the terminal. Done: chart of sessions per day, cost per day, models used, input vs output tokens.

##### Why this approach

- **Zero infrastructure.** No running container, no open port, no service to maintain. If you restart the machine, nothing breaks.
- **Real privacy.** Metrics live in a text file on your disk. If you want to delete everything, `rm ~/.copilot-otel.jsonl`. If you want to see what's there, `cat`. If you want to copy it to another machine, you copy it.
- **Modifiable.** The script is 150 lines of Python without dependencies. If you want to add a new metric, open the file and edit. If you want to change the HTML colors, same.
- **Reproducible.** Another team dev can run the same script with their own JSONL and get their own dashboard. Each with their data, without sharing anything.

##### Limitations

- It's a snapshot, not a real-time dashboard. To see it updated you have to run the script again.
- Costs are estimates based on the script's PRICES table. If Anthropic or OpenAI change prices, the dictionary has to be updated.
- For Claude Code, until Anthropic exposes a direct file export equivalent to Copilot's, the practical alternative is to use Path B (local Aspire).

#### Path B (alternative): local Aspire Dashboard

If you prefer a richer UI in real time, and don't mind having a container running, use the local Aspire Dashboard (same as Section 9.3). Same `docker run`, the Claude Code and Copilot variables point to `localhost:4317`, and you watch everything from http://localhost:18888. It's still all local, nothing leaves your machine.

Trade-off: requires Docker installed, there's a running container while you use it, and data lives in the container (as long as you don't delete it).

#### Secondary mention: Grafana Cloud (free tier)

If at some point the local dashboard falls short (you want trends across months, share with a colleague, get email alerts when last month's cost crosses a certain threshold), the next step is Grafana Cloud. It has a free tier that includes OTLP endpoint, metrics database and web dashboards. It works by pointing environment variables to its endpoint instead of localhost. The downside is that metrics leave your machine toward Grafana. For pure personal use, Path A is the option of least friction and greatest privacy.

#### Why it matters that each person measures themselves

This isn't a classic FinOps exercise. It's something more direct: **if you can't see your own consumption, you can't improve it.** It's the same logic as an athlete with their tracker, or a dev with their profiler. Not measuring isn't neutral, it's blindness.

And for a company, collective governance isn't built from above asking BUs for reports. It's built from below, with each person who understands their own use and adjusts. The tech lead who has a team dashboard relies on data that exists because the people on the team measured first. The BU that reports reasonable consumption to finance does so because the devs measured before. Without the first step, the rest is budget theater.

Ten minutes of setup. Zero euros of cost. Personal governance questions answered forever.

---

## 10. Level 6: alternative lower-cost providers <a id="10-alternatives"></a>

To close the picture: in addition to optimizing Claude/GPT, there's a universe of alternative providers that for certain cases are radically cheaper. The price range is huge: **prices vary up to 625x between the most expensive and the cheapest model on the market**. Most teams pay between 4x and 30x more than necessary for tasks where a cheap model performs the same.

The strategy isn't "replace everything with the cheapest": it's **identifying the tasks where an alternative model gives the same result** and routing those tasks there, keeping Claude/GPT for the work that justifies their price.

### Groq: LPU hardware, extreme speed

Groq has custom hardware (LPUs, Language Processing Units) that serves open source models with sub-100ms latencies. It's the choice for tasks that need instant response: inline classification, autocomplete, content moderation, message routing.

> 🔧 **Layer 1** (app code) · OpenAI SDK with alternative base_url · Application developers who need sub-100ms latency on simple tasks

```python
from openai import OpenAI
client = OpenAI(api_key=os.getenv("GROQ_API_KEY"), base_url="https://api.groq.com/openai/v1")
response = client.chat.completions.create(model="llama-4-scout-17b-16e-instruct",
    messages=[{"role": "user", "content": text}], max_tokens=50)
```

Llama 4 via Groq costs approximately $0.11/MTok input. It's between **4 and 10 times cheaper than GPT-4o** for equivalent tasks, with sub-100ms latencies (versus 500ms to 1s for traditional providers). Note the integration pattern: the OpenAI SDK works by pointing to the Groq URL, no need to learn a new SDK, just change the `base_url`.

### Together AI: 100+ models

Together AI hosts more than 100 open source models with enterprise SLA and fine-tuning support. It's the choice for high-volume batch processing and for cases where you need a specific open source model not available on other platforms.

> 🔧 **Layer 1** (app code) · OpenAI SDK with alternative base_url · Application developers who need high-volume batch with fine-tuning

```python
from openai import OpenAI
client = OpenAI(api_key=os.getenv("TOGETHER_API_KEY"), base_url="https://api.together.xyz/v1")
response = client.chat.completions.create(
    model="meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8",
    messages=messages, max_tokens=OUTPUT_BUDGETS.get(task_type, "summarization"))
```

Same pattern as Groq: OpenAI SDK pointing to another endpoint. Together's plus is the diversity of available models and managed fine-tuning support.

### Mistral: the only major European provider

Mistral is based in France, native GDPR, data in the EU. It's the only **direct major European** model provider alternative: without going through a hyperscaler, without procurement overhead, with European compliance ensured by direct contract.

> 🔧 **Layer 1** (app code) · OpenAI SDK with alternative base_url · Application developers who need native GDPR without going through hyperscaler

```python
from openai import OpenAI
client = OpenAI(api_key=os.getenv("MISTRAL_API_KEY"), base_url="https://api.mistral.ai/v1")
response = client.chat.completions.create(model="mistral-small-latest",
    messages=messages, max_tokens=200)
```

For European companies that need to minimize American dependencies and maintain data residency within the EU without paying hyperscaler overhead, Mistral is the first option to evaluate.

### Decision guide

This table summarizes which provider is appropriate for which task. The operational rule: **each task goes to the provider that best balances quality, price and compliance for that specific task**. There's no universal "winning" provider.

| Task | Provider | Reason |
|-------|----------|--------|
| Complex reasoning | Claude Opus / GPT-5 | Irreplaceable quality |
| General code | Claude Sonnet | Best balance |
| Mass classification | Groq (Llama 4) | 4 to 10x cheaper, sub-100ms |
| Offline batch | Together AI / Fireworks | Low price with SLA |
| EU data residency | Mistral | Only major European |
| EU compliance + minimum cost | DeepSeek via Azure | Compliance + 15x cheaper |

**A note on self-hosting:** running open source models on your own GPUs (with vLLM for example) can be tempting and seems like maximum control. The documented practical rule: it's only economically justified from **$5K to $10K/month of sustained consumption and 70%+ GPU utilization**. Below that, operational cost (GPUs, observability, scaling, fallbacks, on-call) exceeds nominal savings. Source: [morphllm.com](https://www.morphllm.com/llm-api). For the vast majority of teams, managed providers are cheaper in real TCO.

---

## 11. Verified metrics reference <a id="11-metrics"></a>

Document closing: all the metrics used throughout the sections, with their source and URL so any number can be independently verified. Serves as an appendix for negotiations, internal presentations, leadership proposals or when someone questions a figure.

| Metric | Value | Source | URL |
|---------|-------|--------|-----|
| Re-sent context = % bill | 62% | LeanOps 2026 | https://leanopstech.com/blog/agentic-ai-cost-runaway-token-budget-2026/ |
| Tokens turn 1 vs turn 50 | 5K to 200K | Redblink | https://redblink.com/ai-token-cost-optimization/ |
| Output vs input multiplier | 4 to 6x | Redis 2026 | https://redis.io/blog/llm-token-optimization-speed-up-apps/ |
| Cache reads discount | 90% | TokenOptimize.dev | https://www.tokenoptimize.dev/guides/llm-token-optimization-strategies |
| RAG app savings with caching | 88 to 95% | Finout | https://www.finout.io/blog/anthropic-api-pricing |
| VSCode prompt caching reuse | 93% | VS Magazine | https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx |
| Prompt caching automatic vs explicit | Documented | Anthropic official | https://platform.claude.com/docs/en/build-with-claude/prompt-caching |
| Prompt caching break-even | 1 hit | MetaCTO | https://www.metacto.com/blogs/anthropic-api-pricing-a-full-breakdown-of-costs-and-integration |
| RAG token reduction | up to 70% | Koombea | https://ai.koombea.com/blog/llm-cost-optimization |
| RouteLLM quality/calls | 95%/26% | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| RouteLLM savings vs baseline | 48% | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| RouteLLM with augmentation | 75% | LMSYS ICLR 2025 | https://www.lmsys.org/blog/2024-07-01-routellm/ |
| Tool search VSCode | up to 20% | VS Magazine | https://visualstudiomagazine.com/articles/2026/04/30/vs-code-curbs-token-use-ahead-of-copilots-controversial-usage-based-billing-switch.aspx |
| Copilot Auto Mode | 10% discount | GitHub Changelog | https://github.blog/changelog/2026-04-17-github-copilot-cli-now-supports-copilot-auto-model-selection/ |
| Copilot top users = spend | 10 to 15% = 60 to 70% | Synapx | https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/ |
| Agentic cost | ~3.5x flat fee | Synapx | https://www.synapx.com/github-copilot-usage-based-billing-executive-guide/ |
| Batch API | 50% discount | Anthropic | https://www.finout.io/blog/anthropic-api-pricing |
| Batch + caching | up to 95% | PECollective | https://pecollective.com/tools/anthropic-api-pricing/ |
| Thinking omitted = savings | None | CheckThat.ai | https://checkthat.ai/brands/anthropic/pricing |
| Thinking 2000t + 500t output | 5x more expensive | PECollective | https://pecollective.com/tools/anthropic-api-pricing/ |
| Opus 4.7 tokenizer overhead | up to 35% more tokens | Finout | https://www.finout.io/blog/anthropic-api-pricing |
| Azure EA alone | 15 to 25% | Microsoft Negotiations | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| Azure EA + MACC | 23 to 28% | Microsoft Negotiations | https://microsoftnegotiations.com/blog/github-copilot-enterprise-licensing |
| **Azure OpenAI overhead vs direct OpenAI** | **15 to 40% (avg 22%)** | **TokenMix** | **https://tokenmix.ai/blog/azure-openai-cost** |
| **Azure OpenAI overhead vs direct OpenAI** | **15 to 40%** | **Inference.net** | **https://inference.net/content/azure-openai-pricing-explained/** |
| **Azure OpenAI overhead vs direct OpenAI** | **20 to 40%** | **CloudZero** | **https://www.cloudzero.com/blog/azure-openai-pricing/** |
| **Bedrock Claude overhead vs direct Anthropic** | **20 to 35% effective** | **TokenMix Bedrock** | **https://tokenmix.ai/blog/aws-bedrock-pricing** |
| **Bedrock regional endpoints premium** | **+10% over global** | **Official Anthropic docs** | **https://platform.claude.com/docs/en/about-claude/pricing** |
| **Foundry Claude credits exclusion** | **Azure credits don't apply** | **Microsoft Q&A** | **https://learn.microsoft.com/en-us/answers/questions/5851352** |
| **Documented Foundry trap case** | **~$13K USD/month one startup** | **AZ365.ai March 2026** | **https://az365.ai/blog/claude-on-azure-the-marketplace-billing-trap/** |
| **Pricing parity Claude Bedrock/Vertex/Foundry** | **Identical to direct** | **Official Anthropic docs** | **https://platform.claude.com/docs/en/build-with-claude/claude-in-microsoft-foundry** |
| DeepSeek via Azure markup | +20 to 35% | DeployBase | https://deploybase.ai/articles/deepseek-v3-pricing |
| DeepSeek V4-Flash vs Sonnet | ~15x cheaper | 2026 prices | https://techjacksolutions.com/ai-tools/deepseek/deepseek-pricing/ |
| AI % IT spend | up to 50% | Deloitte 2026 | https://redblink.com/ai-token-cost-optimization/ |
| Groq vs GPT-4o | 4 to 10x cheaper | ToolHalla 2026 | https://toolhalla.ai/blog/groq-vs-together-vs-fireworks-2026 |
| Self-hosting break-even | $5K to $10K/month | morphllm.com | https://www.morphllm.com/llm-api |
| Official Anthropic pricing May 2026 | Sonnet $3/$15, Opus $5/$25, Haiku $1/$5 | Anthropic | https://platform.claude.com/docs/en/about-claude/pricing |
| Official OpenAI pricing 2026 | GPT-5.4 $2.50/$15, GPT-5.5 $5/$30, Nano $0.05/$0.40 | OpenAI / Finout | https://openai.com/api/pricing/ |
| GenAI Semantic Conventions OTel | Open standard | OpenTelemetry official | https://opentelemetry.io/docs/specs/semconv/gen-ai/ |
| Claude Code OTel observability | Documented | Anthropic official | https://code.claude.com/docs/en/agent-sdk/observability |
| Copilot Chat OTel monitoring | Documented | VSCode official | https://code.visualstudio.com/docs/copilot/guides/monitoring-agents |
