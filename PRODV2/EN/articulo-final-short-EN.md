# Tokens, the bill, and a conversation we had been postponing (short version)

On June 1st GitHub Copilot changes its billing model. At Visma it's one of the most widely adopted AI vendors, so for many BUs this is the first month where the question "how much are we consuming in tokens?" stops being theoretical. This article covers what changes, what doesn't, and where to start optimizing. The full version, with all objections and implementation detail, lives in `articulo-final.md` (Link to extended article) and the technical documentation in `doc-tecnico-final.md` (Link to technical document).

---

## Why this article, and why now

On June 1st Copilot stops charging flat and moves to usage-based billing. Each seat still costs the same, but now it includes AI Credits: while you're inside those credits there's no extra charge; when they run out, you pay by consumption.

But worth saying from the start: **the rest of the stack already worked this way**. Cursor has been charging per plan with quotas and overage for months. Claude (Desktop, Code, Cowork) the same. Direct APIs from Anthropic, OpenAI, Azure OpenAI, Bedrock have always been pay-per-token from day one. The only thing that changes on June 1st is Copilot.

The question "how much does each token we consume cost us?" should have been bouncing around in our heads for a while. If it wasn't, it's because Copilot's flat fee blocked our view of the part of the stack where we consume the most, and because in the rest consumption was still small or buried inside larger budgets nobody was auditing. That period is ending.

**Using AI well isn't just prompt engineering, it's knowing how to consume fewer tokens to get the same result.** That skill, just like cloud FinOps 10 years ago, is built with habit, not with a magical tool. June 1st doesn't invent the problem, it makes it visible.

Idea that's going to come up several times: **spend well, not spend less**. We're not trying to shrink AI. We're trying to make sure every euro put into AI is working, not heating up air.

---

## What's happening with the money (and nobody is looking)

Across 30 teams surveyed between March and May 2026, **62% of the AI bill is not for model work**. It's for sending the same context, over and over. Of every 100 euros a company puts into AI, 62 are literally the same system prompt, the same project documentation, the same history, going back and forth to the model. It's as if a courier charged you every time it moved a box and you made it pass by the same corner 50 times because you forgot to ask it to leave the box at the destination.

The concrete part: a typical session starts at 5,000 tokens. By turn 50 it's at 200,000. Nobody sees it. There's no sign that says "you're at 40x the cost of when you started". The bill arrives at month-end and it's attributed to "more AI usage".

Until now this didn't hit us directly because in Copilot we paid flat and in the rest of the tools consumption wasn't yet enough for anyone to raise a hand. That changes on June 1st. And the operational question, for any dev, team or BU, becomes one: how much of your bill is real work, and how much is the courier going in circles?

---

## The spectrum: closed products, configurable tools, direct API

It's the most important distinction in the article. A widespread idea says the AI world is divided in two: tools (black boxes) and APIs (transparent). That division was true a year and a half ago. Not today. We have a spectrum of three levels.

**Closed products.** Chat-style applications without instrumentation designed for the user: ChatGPT consumer version, Gemini, Claude.ai on Free/Pro plans. The only thing you see is the chat, the response, some monthly quota. You don't have a model picker, you can't modify parameters, there's no telemetry that can be exported.

**Configurable tools.** Products that do allow significant control: Copilot inside VSCode (the most common case at Visma), Claude Code, Cursor, Claude Cowork on Team or Enterprise plans. Here you can choose the model, activate telemetry, configure settings that move the cost (Auto Mode, output compression, MCP control), monitor consumption from the IDE itself. The caching is done by the tool for you.

**Direct API.** You write the code that calls the model: Anthropic direct API, OpenAI API, AWS Bedrock, Google Vertex, Azure OpenAI. Total control: exact prompt, context, model, parameters, explicit caching, per-call metrics. It's where the biggest discounts live (90% caching, 50% batch) but it requires building.

A BU can be at all three levels simultaneously. That's not a problem, it's reality. The simple rule for designing new implementations: **configurable tools for human use** (a dev writing code), **API for programmatic use** (a product feature that calls a model with no human on the other side).

---

## Three things worth knowing before touching anything

These are the most frequent objections I heard this week leading this conversation. If you understand these three, you already have 80% of the value of this article.

### Prompt caching: the biggest lever in the system

The same content sent twice doesn't cost twice. Anthropic charges cached tokens at 10% of the normal price, **a 90% discount on reads**. The first send costs 25% extra, but as soon as that same content is read from cache once, you're already saving.

How does it apply in each case?
- **Claude Code or Claude Desktop:** the tool caches for you, you don't have to activate anything.
- **Copilot in VSCode:** same, with published cache reuse rate of 93%.
- **Direct API:** you activate it yourself, there's automatic mode and explicit mode with fine control.

A typical app with a stable system prompt and high volume, with caching well activated, can reach **88 to 95% savings** compared to the price without cache. That's the difference between having a reasonable bill and one that hurts.

### Output costs 4 to 6x more than input

Technical detail that gets underestimated a lot. When you ask a model to summarize everything we've talked about so far, the summary comes out very expensive. When you ask it to rewrite an entire file instead of doing a diff, also. The asymmetry is in every decision.

And a particular case: if you have extended thinking activated, **those tokens are billed as output even if you don't see them**. Hiding the reasoning in the display saves you perceptual latency, not money. A call with 2,000 tokens of thinking and 500 of output costs 5x more than one without thinking. If you activated it "just in case" and never turned it off, this affects you.

### The EA discount is not automatic savings

The EA (Enterprise Agreement with Microsoft) negotiates a **15 to 25% discount**, and with MACC (Microsoft Azure Consumption Commitment) it goes up to **23 to 28%**. But that discount applies to Azure consumption, not to direct API. And Azure over OpenAI direct has an overhead of **15 to 40% in TCO** (Total Cost of Ownership).

Let's do the math. A 25% discount on a service that has 30% overhead. The "discount" disappears. The EA helps, but not as automatic savings: it helps as compliance, data residency and bill consolidation. And that, for European companies with GDPR on top, is exactly what we're buying. **We pay the hyperscaler for legal certainty, not for price.** It's fine that it's that way, but it needs to be called by its name.

---

## "But before charging me, won't it warn me?"

Short answer: **by default they don't blow up your bill**. But the difference between "they don't blow it up" and "your budget covers it" depends on how you configure each tool.

The mental image of "they charge without limit until you get a scare at month-end" is exaggerated. Almost all tools have provider rate limits, plan quotas, or configurable spending limits. The behavior varies:

- **Claude Pro / Team / Enterprise:** cuts off cleanly when the quota runs out. No overage.
- **Cursor:** plan cap. If you want overage you have to activate it explicitly.
- **Copilot under the new billing:** AI Credits included, then it depends on the spending limit the organization configured.
- **Anthropic API / OpenAI API:** workspace has configurable monthly spend limit, with a default value. When you hit it, it returns an error.
- **Azure OpenAI / AWS Bedrock:** here more attention is needed. The bill goes to the general cloud account. It's not an open tap (rate limits stop you before), but you should always configure budget alerts.

**The operational truth:** no serious provider bills you silently. What does happen is that the default cap may not match your real budget. That's why the exercise is to review, not to panic. This article doesn't come to tell you "they're going to wreck your bill". It comes to tell you the opposite: **use your tokens well so what you already pay is enough**. And if you decide to open overage, let it be a conscious decision, with a cap.

Where to find each setting on each platform is in the project's technical documentation, section 8.3 (Link to technical document).

---

## Personal governance: how to measure yourself

This entire conversation starts at something much smaller: **every person who uses AI every day can, and should, see their own consumption**. Not the team's. Not the BU's. Their own. If you don't know how many sessions you opened this week, how much repeated context you sent, what models you triggered, you're not doing governance, you're guessing.

The good news: measuring yourself takes ten minutes of setup and zero euros. Both Claude Code and Copilot Chat can export their telemetry to a local file. With a small Python script that reads that file and generates a static HTML with charts, you have your personal dashboard running, **all on your machine, without uploading metrics to any external service**.

The exact how (environment variables, settings, complete script, HTML template) is in the project's technical documentation, section 9.6 (Link to technical document).

For Visma, collective governance isn't built from above asking BUs for reports. It's built from below, with each person who understands their own use and adjusts. The tech lead who has a team dashboard relies on data that exists because the people on the team measured first. The BU that reports reasonable consumption to finance does so because the devs measured before. **Without the first step, the rest is budget theater**.

---

## It's not a recipe. It's a moment.

Nothing of what I wrote here is "the way" to work with AI. I'm not saying everyone has to have a dashboard, or migrate features to direct API, or do multi-model routing on Friday.

**This is the way to understand. And understanding, today, is what needs to be done.**

We had a couple of very generous years. New tools appeared every month, models were getting better, prices were dropping, and flat plans let us experiment without thinking twice. If you activated extended thinking "just in case", if you opened 40 sessions this week, if you left an agent running all weekend, the cost of learning from that was low. The flat bill arrived, we all kept going.

That period is ending. Not all at once, not for everyone, not on June 1st. But it's ending. And the window to learn cheap, to experiment, measure, get it wrong, adjust, without it hurting, **is exactly now**.

Those who take advantage of this window will come out the other side understanding how AI consumption is governed. They'll know what setting moves what number. They'll have intuition. They'll be the ones who in six months, when this is company policy, already have the answer. **Those who let it pass will have to learn the same thing, but with the bill running and less margin to experiment.** It's the same curve, different emotional context.

That's why this article. Not to tell you what to do, but to show you where to start learning, and to let you know that the moment is this one. Becoming good at this isn't magic, it's going through a learning curve. The curve is there. The timing too. The only thing missing is deciding to step in.

---

## Where to start this week

Three steps by level of effort. Anyone can start with the first today.

**Zero effort, today.** Activate Auto Mode in Copilot (10% automatic discount). Activate terminal output compression in VSCode. Activate tool search for MCPs. In Cursor, make sure you're on Auto. Review and configure spending limits where applicable in your case. Combined estimated savings: **15 to 25%** without touching workflows.

**A sprint.** Set up your personal dashboard with the local script. If your team has an app that calls a model API, activate prompt caching. Migrate asynchronous workflows to Batch API: 50% discount, combinable with caching for up to 95% savings. Estimated savings: **60 to 90%** in that app's cost.

**A quarter.** Multi-model routing: classification to Haiku, code to Sonnet, complex decisions to Opus, cheap batch to DeepSeek via Azure. RouteLLM or LiteLLM as gateway. **Reported savings: 20 to 80% of OpEx**.

The detail of each step, with exact settings, is in the project's technical documentation (Link to technical document).

---

**Spend well, not spend less.** The June 1st change is just the event that forced us to look. What matters is the capability that gets built afterward: users who understand what they consume, teams that monitor what they pay, BUs that choose between configurable tools and direct API with criteria. That capability isn't brought by Copilot, or Cursor, or Anthropic. It's brought by the people who dare to understand what they're consuming. And it starts, literally, with each dev who sets up their dashboard one Tuesday afternoon.
