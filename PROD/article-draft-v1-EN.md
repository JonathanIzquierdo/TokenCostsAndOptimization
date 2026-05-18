# The Money Lost in Every AI Conversation

*How tokens became the new engineering efficiency metric — and what to do about it*

---

## Opening

There's a number that, when you hear it for the first time, doesn't seem real.

**62% of what your company pays for AI** is not for the work the model does. It's context that was already processed, being sent again, in every request, every conversation turn, every agent call. Not because it's necessary. Because nobody configured it not to.

This wasn't estimated by anyone. It was measured by LeanOps in an audit of 30 engineering teams running agentic AI in production between March and May 2026. More than half the bill is repeated noise.

Most of the team's intuition is that we pay for what the model *produces* — the responses, the generated code, the analyses. That intuition is wrong. We pay for everything we *tell* it before it produces anything — and in most production systems, we tell it the same things over and over.

This is not a problem of which model you're using. It's an architecture problem. And it has a solution.

---

## Section 1 — The invisible problem: how costs grow without anyone noticing

### The conversation that looks flat but isn't

Imagine a working session with an agent. It starts simple: you give it context, ask for something, it responds. Ten minutes later you ask something related. Twenty minutes later, something else. An hour in, you're still in the same session.

What you don't see is what's happening underneath. Every new turn includes the entire previous history. Not just the last message — everything. The agent needs that history to maintain coherence. But you pay for every token of that history in every request.

A 20-turn conversation can consume between 5,000 and 10,000 tokens when 500 to 1,000 tokens of recent context would be enough. The rest is history the model already processed, already "knows", and you're paying it to process again.

In short sessions, this is manageable. But in agentic workflows — where an agent works for hours, calls tools, receives results, iterates — the effect compounds exponentially. A session that starts at 5,000 tokens per call can reach 200,000 tokens per call by turn 50. The model's work didn't change. The cost did.

### The multiplier nobody configures

There's another factor that makes everything worse: output tokens cost between 4 and 6 times more than input tokens.

In models like Claude Sonnet 4.6, input costs $3 per million tokens. Output costs $15 per million. That's not a marginal difference — it's a 5x multiplier.

Most teams set prompts carefully. Very few set `max_tokens` explicitly in their workflows. When you don't, the model generates what it considers appropriate for the task. On simple tasks — a classification, an extraction, a yes/no answer — that can be 10 to 50 times more tokens than necessary.

An app without `max_tokens` configured, where the model responds with 800 tokens when it needed 200, has a 4x higher output bill. For the same result.

### The hidden cost of extended thinking

Models with extended reasoning — Claude Opus with adaptive thinking, OpenAI's o1/o3 — have a behavior many teams don't know in detail: their *thinking* tokens are billed as output tokens.

When Claude Opus 4.7 solves something complex, it can generate thousands of internal reasoning tokens before producing the response. Those tokens are billed at $25 per million.

The most common mistake: using `display: "omitted"` believing it reduces the cost. It doesn't. Omitting the thinking display reduces perceived latency, but the tokens are still generated and still billed. A response with 500 visible output tokens and 2,000 thinking tokens costs 5 times more than the same response without thinking enabled.

This doesn't mean extended thinking is bad. For architectural analysis or complex debugging, it may be exactly what you need. But applying it to simple classifications or boilerplate completions is a cost multiplier that goes completely unnoticed if you don't know to look for it.

### The Copilot effect: the 10% that drives 60%

In enterprise Copilot deployments, there's a consistent pattern: the top 10 to 15% of users concentrates between 60 and 70% of advanced feature consumption.

These are the developers who use chat intensively, run agentic sessions, do code review with Copilot. They're also the most productive on the team — and with the new billing model, they're the ones who will exhaust the included credits first.

For teams with intensive agentic workflows, estimates point to costs around £52 per developer per month — approximately 3.5 times the previous flat fee. Not for the whole team: for the intensive user cohort. But if that cohort represents 15% of a team of 200 people, their impact on the total bill is greater than the entire rest of the team combined.

---

## Section 2 — The change that makes all of this urgent now

For years, the cost of AI in development tools was predictable. You paid a flat fee per seat. The most intensive developer cost the same as the one who barely used it. Tokens were an implementation detail, not a business metric.

That changed on June 1, 2026.

### Copilot is no longer a flat subscription

GitHub Copilot migrated to usage-based billing. Each plan includes credits equivalent to the seat price — $19 in AI Credits for Business, $39 for Enterprise. One AI Credit equals $0.01. When the included credits run out, every additional token has a direct price.

Credits are pooled at the enterprise level. If someone doesn't use theirs, others consume them automatically. What sounds good in theory — collective efficiency — means in practice that the intensive 10-15% cohort can consume the entire pool before the rest of the team has a chance to use it.

GitHub is offering a transition period with additional credits for the first three months: $30 extra for Business, $70 extra for Enterprise. This cushions the initial impact. But starting in September, the real economics kick in.

### What remains free (and what doesn't)

Inline completions and code suggestions do not consume AI Credits. They remain unlimited on all paid plans. If your team uses Copilot primarily for autocomplete, the impact is smaller.

What does consume credits: chat in the editor, chat on GitHub.com, Copilot Workspace, Cloud Agents, and agentic code review. And that last one has an important detail: it runs on GitHub Actions, meaning it consumes both AI Credits and GitHub Actions minutes. Double billing that many teams don't anticipate.

### The agentic multiplier

The billing change coincides with a change in how Copilot is used. The product evolved from an autocomplete assistant to an agentic platform that can handle long, multi-step sessions iterating over entire repositories. That agentic use is the default for the most advanced users — and it brings significantly higher compute demands.

A developer spending the day doing code completions generates predictable and relatively low consumption. The same developer using Copilot Workspace to plan features, refactor modules, and generate automated tests can generate 10 to 20 times more tokens in the same period. Without doing anything "wrong" — they're simply using the tool for what it was designed for.

### The scale of the problem

Deloitte documented in 2026 that in some enterprise firms, AI consumes up to 50% of total IT spending. IDC predicts Global 1000 companies will underestimate their AI infrastructure costs by 30% by 2027.

These aren't numbers from startups experimenting with APIs. These are organizations with mature processes, controlled budgets, who still ended up with AI bills they didn't anticipate. The reason is always the same: tokens are invisible until they appear in the billing.

The good news is that there are concrete optimization layers available today. Some take five minutes. Others require architecture changes. But all of them move the bill.

---

## Section 3 — The optimization layers

There's no single lever that solves the problem. There's a sequence of layers, each more technical than the last, each with greater potential impact. You can apply just the first ones and already notice a difference. You can apply all of them and radically change the economics of your AI usage.

---

### Layer 0 — What you can do today, in five minutes, without changing any architecture

These are settings. No new code, no architecture approvals, no sprints needed.

**Enable Auto Mode in Copilot**

In the Copilot model selector in VSCode, there's an "Auto" option that most of the team ignores because it sounds vague. It isn't: with Auto enabled, Copilot dynamically selects the most efficient model for each task — routing between GPT-5.4, GPT-5.3-Codex, Claude Sonnet 4.6 and Haiku 4.5 based on request complexity.

The immediate benefit is a 10% discount on the model multiplier. The structural benefit is that simple tasks go to simple models, and only complex tasks use the expensive ones. Zero additional configuration.

**Enable `chat.tools.compressOutput.enabled` in VSCode**

Add this to your `settings.json`:

```json
{
  "chat.tools.compressOutput.enabled": true
}
```

This VSCode 1.120 feature post-processes terminal command output before sending it to the model as context. It collapses unchanged diff hunks, discards complete lockfile diffs, reduces `ls -l` to filenames, removes `npm install` progress bars and deprecation warnings.

The output of a `git diff` in an active repository can have thousands of lines. Without compression, all of that goes to the model. With compression, only the relevant changes go through.

**Tool search — deferred MCPs by default**

VSCode 1.118 introduced a separation of the agent toolset: approximately 30 core tools are always available (covering 88% of real cases), and the rest are loaded on demand when the model explicitly needs them.

For Anthropic models this is active by default. For GPT models in Copilot:

```json
{
  "github.copilot.chat.responsesApi.toolSearchTool.enabled": true
}
```

Impact: up to 20% token savings per request, simply because the schemas of tools you never use stop taking up context space.

---

### Layer 1 — Context architecture

**The 90% discount almost nobody uses: prompt caching**

Prompt caching is probably the most powerful cost lever available today, and most teams either don't have it enabled or use it incorrectly.

The mechanics are simple: the static content of your prompts — system prompt, tool definitions, knowledge base, examples — is processed once and cached. Subsequent calls that include that same prefix don't reprocess it. Anthropic cache reads cost 10% of the base input price — a 90% discount on that portion of the context.

A RAG app with a 50,000-token knowledge base in the system prompt, queried 1,000 times a day, pays the full cost of those 50,000 tokens in every request without caching. With caching, it pays 10% in the vast majority of those requests. Savings on that single portion of the system: between 88% and 95%.

Break-even: a single cache hit with a 5-minute TTL.

**Conversation history: sliding window, not full history**

Instead of sending the complete conversation in every turn, maintain only the last N relevant turns plus a compressed summary of earlier context. The exact implementation depends on the framework, but the principle is universal: the model doesn't need to remember every detail of every turn — it needs the relevant context for the current task.

Combined with Anthropic's Compaction API (available in beta since January 2026), this can be fully automated. The API automatically summarizes the history when context approaches the configured limit, using semantic understanding — significantly better than naïve truncation. The system prompt stays cached through multiple compactions, and the API is eligible for Zero Data Retention, making it viable for regulated environments.

**RAG instead of context stuffing**

Sending entire documents to the context because "it might be relevant" is one of the most expensive patterns in production. RAG can reduce context token usage by up to 70% by retrieving only the specifically relevant fragments for the current query.

---

### Layer 2 — Correct model selection

This is the layer with the most impact on cost per request, and the easiest to ignore when everything "works".

The price difference between model tiers is dramatic: Haiku is 5x cheaper than Sonnet and 25x cheaper than Opus. Most agentic frameworks use a single model for everything — and that model is usually Sonnet or Opus because it "gives better results". What they don't measure is that for 70% of workflow tasks, Haiku gives exactly the same results at a fraction of the cost.

The correct assignment logic:
- **Haiku:** classification, structured data extraction, formatting, file navigation, and anything that's essentially pattern matching
- **Sonnet:** chat, summarization, general code generation, document analysis, and most daily work
- **Opus:** complex architectural analysis, multi-step reasoning where output quality impacts important decisions, agent coordination

**The always-active MCPs problem**

Every active MCP adds its complete schema to the context on every request — even if you never use that tool in that session. With 10 active MCPs and 500-token schemas each, you're adding 5,000 tokens of overhead per request before doing anything.

At $3/MTok input in Sonnet, on 100,000 monthly requests: $1,500 monthly cost for tools nobody asked for.

---

### Layer 3 — Intelligent routing

Intelligent routing automates model selection. Instead of a human deciding which model to use for which task, a system decides dynamically per request.

- **Auto Mode + OpenRouter:** zero configuration, 10% discount, routing across 300+ models
- **RouteLLM (open source, UC Berkeley/LMSYS):** 95% GPT-4 quality with 26% of calls — 48% cheaper than random baseline. With data augmentation: 75% cost reduction. Drop-in replacement for the OpenAI client
- **AI Gateways (LiteLLM, Portkey):** the most important argument isn't routing — it's protection. Without a gateway, a single runaway loop can burn the entire API budget overnight

---

### Layer 4 — Infrastructure levers

**Batch API: 50% discount with no quality change**

For any pipeline where the response doesn't need to be real-time, Batch API is free money. 50% discount on both input and output tokens. Quality is identical to synchronous processing. Combined with prompt caching: a cached batch request can cost as little as 5% of a standard uncached request.

**DeepSeek via Azure: the world's cheapest model with European compliance**

DeepSeek V4-Flash costs $0.14 per million input tokens and $0.28 per million output tokens direct. For high-volume, low-complexity tasks — bulk classification, data extraction, offline summarization — it's 35 to 100 times cheaper than frontier models.

The problem: direct DeepSeek API routes all requests through Chinese servers. Azure AI Foundry integrates DeepSeek models with a 20–35% markup over official rates. That markup is easily justified: data stays within the Azure compliance boundary, no transmission to China, European data residency maintained.

Even with the Azure markup, DeepSeek V4-Flash costs approximately $0.19 per million input tokens — around 15x cheaper than Sonnet for the same high-volume tasks.

**Microsoft EA negotiation**

For organizations with Microsoft Enterprise Agreements, the discount on Copilot isn't fixed — it's negotiated. Documented range: 15–25% with EA alone, 23–28% for organizations with both EA and Azure MACC commitments. Requires framing the conversation with Microsoft's account team in the context of EA renewal, not negotiating Copilot in isolation.

---

## Closing — From individual practice to team culture

Everything described here can be applied by one person alone this week. Auto Mode, compressOutput, prompt caching — these are individual decisions with immediate impact.

But the underlying problem isn't solved with individual settings. It's solved with collective visibility.

The 10–15% of users concentrating 60–70% of spend aren't doing anything wrong. They're using the tools for what they were designed for. The problem is that nobody sees that consumption until the bill arrives. And when the bill arrives, the conversation becomes reactive: cuts, restrictions, limitations.

The alternative is building proactive governance: token consumption observability per user and per team, alerts at 50%, 80%, and 100% of the monthly credit, model selection policies per task type, and per-team budgets with automatic controls.

Companies that build this infrastructure now will reuse it for every new AI service they adopt. Those that treat it as a one-time billing change will rebuild it from scratch every time.

The goal isn't to spend less. It's to spend well.

---

## What to do this week

**Today (5 minutes each):**
- Switch the Copilot model selector to "Auto" in VSCode
- Add `chat.tools.compressOutput.enabled: true` to `settings.json`
- Verify you're on VSCode 1.118 or higher

**This week:**
- Review which MCPs you have active — disable the ones you don't use frequently
- Identify which tasks go to Sonnet/Opus that could go to Haiku
- Enable the billing preview in GitHub Billing Overview

**This month:**
- Implement prompt caching in any system with long system prompts
- Evaluate Batch API for any pipeline that doesn't need real-time responses
- Map the team's top consumers and understand which workflows drive that consumption

**Medium term:**
- Evaluate RouteLLM or an AI gateway for agentic systems in production
- Negotiate the Microsoft EA at the next renewal, including Copilot in the MACC context
- Explore DeepSeek via Azure for high-volume workloads

---

*For a deep dive on each topic with technical details, code examples, and step-by-step implementation guides, see `PROD/technical-docs-draft-v1-EN.md`*
