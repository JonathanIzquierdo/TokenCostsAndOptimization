# Tokens, the bill, and a conversation we had been postponing

On June 1st GitHub Copilot changes its billing model. At Visma it's one of the most widely adopted AI vendors, so for many BUs this is the first month where the question "how much are we consuming in tokens?" stops being theoretical. An article to understand the change, the impact, and where to start optimizing.

---

## Why this article, and why now

On June 1st GitHub Copilot stops charging a flat fee and moves to usage-based billing. Each seat still costs the same, but now it includes AI Credits: while you're inside those credits there's no extra charge; when they run out, you pay by consumption. It's a change that hits Visma directly because Copilot is one of the most widely adopted AI vendors internally: a significant portion of our AI consumption today goes through it.

That's the trigger. But it's worth saying from the start: **the rest of the stack already worked this way**. Cursor has been charging per plan with quotas and overage for months. Claude (Desktop, Code, Cowork) the same. The direct APIs from Anthropic, OpenAI, Azure OpenAI, Bedrock have always been pay-per-token from day one, never had flat fees. The only thing that changes on June 1st is Copilot. Everything else was already there.

And there's the uncomfortable part. The question "how much does each token we consume cost us?" should have been bouncing around in our heads for a while. If it wasn't, it's because Copilot's flat fee was blocking our view of the part of the stack where we consume the most, and because in the rest (Claude, Cursor, APIs) consumption was still small or buried inside larger budgets nobody was auditing.

That period is ending. And it's ending in the worst possible way: with the change of the most-used vendor, right when nobody at Visma is particularly trained to think of token consumption as a user skill. Because that's what it is. **Using AI well isn't just prompt engineering, it's knowing how to consume fewer tokens to get the same result.** And that skill, just like cloud FinOps 10 years ago, is built with habit, not with a magical tool.

Seen this way, **this event is the right excuse for a conversation we already owed ourselves.** June 1st doesn't invent the problem, it makes it visible. And while the problem was invisible, nobody had reasons to build the capability. Now they do.

This article tells the story: what's happening, why it matters, what optimizations exist, where they live. It's the "to understand" reading. When some point catches you and you want to dig into the how (the snippets, the exact settings, the commands to configure), all of that lives in the **project's technical documentation** (Link to technical document). This text tells you what to look for; the technical one tells you how to do it.

There's an idea that's going to come up several times: spend well, not spend less. We're not trying to shrink AI. We're trying to make sure every euro put into AI is working, not heating up air.

---

## What's happening with the money (and nobody is looking)

Before talking about what to do, it's worth showing where the problem is.

Across 30 teams surveyed between March and May 2026, **62% of the AI bill is not for model work**. It's for sending the same context, over and over. Of every 100 euros a company puts into AI, 62 are literally the same system prompt, the same project documentation, the same history, going back and forth to the model. It's as if a courier charged you every time it moved a box and you made it pass by the same corner 50 times because you forgot to ask it to leave the box at the destination.

The concrete part: a typical session starts at 5,000 tokens. By turn 50 it's at 200,000. Nobody sees it. There's no sign that says "you're at 40x the cost of when you started". The bill arrives at month-end and it's attributed to "more AI usage".

Until now this didn't hit us directly because in Copilot we paid flat (Copilot Business $19/seat, Enterprise $39/seat) and in the rest of the tools consumption wasn't yet enough for anyone to raise a hand. That changes on June 1st. And the operational question, for any dev, team or BU, becomes one: how much of your bill is real work, and how much is the courier going in circles?

---

## The spectrum: closed products, configurable tools, direct API

Before moving on, a conceptual pause that matters for people at all technical levels. It's the most important distinction in the article.

There's a widespread idea that says the AI world is divided in two: tools (black boxes) and APIs (transparent). That division was true a year and a half ago. Not today. What we have is a spectrum of three levels, and understanding where each thing sits changes the cost and control analysis significantly.

**Closed products.** Chat-style applications without instrumentation designed for the user: ChatGPT consumer version, Gemini, Claude.ai on Free/Pro plans. The only thing you see is the chat, the response, some monthly quota from the plan. You don't have a model picker, you can't modify parameters, there's no telemetry that can be exported. Any serious optimization depends on what the vendor decides to expose, which is typically nothing.

**Configurable tools.** Products that do allow significant control: Copilot inside VSCode (the most common case at Visma), Claude Code, Cursor, Claude Cowork on Team or Enterprise plans. Here you can choose the model, activate telemetry, configure settings that move the cost (Auto Mode, output compression, MCP control), monitor consumption from the IDE itself. The caching is done by the tool for you. You don't have access to the deepest options but you do have a good amount of accessible optimization.

**Direct API.** You write the code that calls the model: Anthropic direct API, OpenAI API, AWS Bedrock, Google Vertex, Azure OpenAI. Total control: exact prompt, context, model, parameters like temperature/top_p/top_k, explicit caching, per-call metrics, per-token costs. It's where the biggest discounts live (90% caching, 50% batch) but it requires building.

A BU can be at all three levels simultaneously: configurable tools for devs to use Copilot, API for a product feature that calls an LLM under the hood, and maybe some informal team using ChatGPT consumer. That's not a problem, it's reality. What matters is knowing **where each case sits** and not assuming that anything that's not an API is a closed product.

The simple rule for designing new implementations: **configurable tools for human use** (a dev writing code, someone exploring ideas), **API for programmatic use** (a product feature that calls a model with no human on the other side). Closed products are a problem mainly when they're used for cases that warrant something else: if your team is using ChatGPT consumer for tasks that require observability, governance or repeatability, there's something to review there.

---

## The objections I've heard three times this week

Leading this conversation inside Visma left me with a collection of responses. They're worth visiting, because the questions are the same across all BUs.

### "I use Cursor or Claude Code, this doesn't affect me"

It does affect you, just for a different reason. June 1st doesn't change Cursor or Claude, those were already on plans with quotas and overage. What changes is that **before, nobody was watching consumption anywhere in the stack**, because Copilot was flat and the rest was still small. Now that the biggest line of the budget (Copilot) moves to usage-based, BUs are going to start watching everything. And when they start watching Cursor or Claude they'll find exactly the same vices: eternal sessions, repeated context, expensive models for small tasks.

The concrete part for you: the techniques that move the needle are the same. Keep sessions short, open a new one when the conversation changed topics, remove what the model doesn't need to see, choose the smallest model that works for the task. In Cursor that translates to using Auto mode and not forcing Claude Opus for everything. In Claude Code, to taking care of the project context file and not letting it grow without restraint.

### "How does prompt caching work and what is it for?"

Here's one of the biggest levers in the system. The idea is simple: **the same content sent twice doesn't cost twice**. Anthropic charges cached tokens at 10% of the normal price, a 90% discount on reads. The first send (cache write) costs a bit more, 25% extra, but as soon as that same content is read from cache once, you're already saving. From the second hit onward, the savings multiply.

What does this mean in practice?

- If you use **Claude Code or Claude Desktop**, you don't have to activate anything. The tool caches for you. Your project context file, the system prompt, the files you load, all go to cache automatically. What you can do is keep your prompts and the structure of open files stable to favor hits; every time you change too much, you invalidate cache.
- If you use **Copilot in VSCode**, same. The tool has a published cache reuse rate of 93% inside the editor. It's working even if you don't see it.
- If you **build with direct API**, you activate it yourself. There's an automatic mode (a single field in your request and that's it) and an explicit mode with fine control over what to cache and what not.

A typical app with a stable system prompt and high volume, with caching well activated, can reach **88 to 95% savings** compared to the price without cache. That's the difference between having a reasonable bill and one that hurts.

The implementation detail (how to activate each mode, how to verify the cache is hitting, what happens in Bedrock and Vertex) is in the project's technical documentation, section 5.1 (Link to technical document).

### "My session isn't that long"

That's what I thought too. Then I looked.

A normal 20-turn conversation drags along between **5,000 and 10,000 unnecessary tokens** that could have been left out. That's per conversation. Multiply by the number of conversations per day. Multiply by the team. Multiply by the Visma companies.

And a technical detail that gets underestimated a lot: **output costs 4 to 6x more than input**. It's not a minor detail. When you ask a model to summarize everything we've talked about so far, the summary comes out very expensive. When you ask it to rewrite an entire file instead of doing a diff, also. The asymmetry is in every decision.

### "I have extended thinking hidden from the display. It's not shown, it doesn't count"

It counts. It counts as output tokens. The fact that it's not shown doesn't mean it's not billed. Hiding the reasoning saves you perceptual latency, not money.

A call with 2,000 tokens of thinking and 500 of output costs **5x more** than one without thinking. If you activated extended thinking because "it always gives better answers" and never turned it off, this affects you. Thinking is expensive. Deciding when it's worth it is the prompter's job, not the model's.

### "We have a Microsoft Enterprise Agreement, we already have a discount"

We do. The EA (Enterprise Agreement, the corporate master contract with Microsoft) negotiates a **15 to 25% discount**, and with MACC (Microsoft Azure Consumption Commitment, the multi-year spend commitment that enables additional discounts) it goes up to **23 to 28%**. But that discount applies to Azure consumption, not to the direct API. And Azure over OpenAI direct has an overhead of **15 to 40% in TCO** (Total Cost of Ownership, the real total cost summing support, egress, monitoring and everything that doesn't show on the pricing page).

Let's do the math. A 25% discount on a service that has 30% overhead over the direct alternative. The "discount" disappears. It's not that the EA doesn't help, it does, but not as automatic savings. It helps as compliance, data residency and bill consolidation. And that, for European companies with GDPR on top like Visma's, **is exactly what we're buying.** We pay the hyperscaler for legal certainty, not for price.

The extreme case is DeepSeek via Azure: **+35% markup** over the direct price, but the direct price isn't an option because the servers are in China. Here the EA discount buys compliance, not savings. It's fine that it's that way, **but it needs to be called by its name.**

### "This will solve itself with cheaper models"

Partly, yes. But there's already a technique that delivers **95% of GPT-4's quality with 26% of the calls to the strong model**. It's called RouteLLM. With data augmentation, it drops to **14% of calls to the strong model and 75% savings**.

**37% of enterprises already run 5+ models in production**. IDC projects **70% for 2028**. The average savings from intelligent model routing is **20 to 80% of OpEx**.

This is already here. It's not the future. The question is not "will cheaper models come?", it's "are we using the cheap ones that already exist for the tasks that don't need the expensive ones?".

---

## "But before charging me, won't it warn me? Doesn't it block on its own?"

This is the most important question in the article, and the one most people asked me. The short answer, after reviewing it carefully, is: **by default they don't blow up your bill. But the difference between "they don't blow it up" and "your budget covers it" depends on how you configure each tool.**

There's a myth here worth dismantling first. The mental image of "they charge without limit until you get a scare at month-end" is exaggerated. Reality is more nuanced. Almost all tools have one of these three mechanisms:

1. **Provider rate limit.** A hard ceiling of requests or tokens per minute that the vendor applies by default. You don't configure it. It stops you before any explosion, returning a "rate limit exceeded" type error.
2. **Plan quota.** What you paid for gives you X consumption. When it runs out, it returns "limit reached" and you have to wait for the reset (monthly typically) or pay more.
3. **Configurable spending limit.** You define how much you're willing to spend on top of the plan. If you leave it at zero (the reasonable default), there's no overage. If you raise it, then there's risk controlled by you.

**How each tool behaves by default:**

- **Claude Pro / Team / Enterprise (Desktop, Code, Cowork):** cuts you off cleanly when the plan quota runs out. No overage. You wait for the reset or upgrade.
- **Cursor:** same model. Plan cap. If you want overage you have to activate it explicitly from org settings.
- **GitHub Copilot under the new billing:** brings AI Credits included in the seat. When they run out, what happens depends on the organization's **spending limit**. By default it usually comes at zero (cuts off and that's it). If the admin raised it, it consumes up to that cap and then cuts off.
- **Anthropic direct API:** your workspace has a configurable **monthly spend limit** from the console. While you don't touch it, there's a default value (not infinite). When you hit it, it returns an error. Apart from that, the per-minute rate limits stop any runaway before the bill grows.
- **OpenAI direct API:** same model. Settings → Limits dashboard brings a **monthly budget** and configurable alerts.
- **Azure OpenAI / AWS Bedrock:** this is where more attention is needed. The bill goes to the general cloud account. It's not "open tap" (the model rate limits and subscription quotas stop you before), but you can end up paying more than expected if you didn't configure **budget alerts** in Azure Cost Management or AWS Budgets. The good practice is to always configure them.

**The operational truth then is this:** no serious provider bills you silently without warning. What does happen is that **the default cap may not match your real budget**. That's why the exercise is to review, not to panic.

What needs to be done is **not a uniform recipe**. Each Visma BU has its mix of providers, its plans, its corporate agreements. It's an exercise of reviewing and configuring:

- **Review your specific situation.** What tools do you use? Under what plan? Is billing centralized in an organization or does each team pay separately? That information determines where you need to go to look, and sometimes reveals that the conversation is with IT or the organization's admin, not something you touch directly.
- **Confirm the behavior in your case.** In each console, look: does the plan cut off only when the quota runs out or does it allow overage? Is there a spending limit configured? Are there budget alerts? The reasonable default is: hard plan quota, no overage. If you want overage so you don't run out of service mid-month, activate it with an explicit cap that makes sense for your budget.
- **Review again in a month.** Tools change settings, plans get updated, contracts get renegotiated.

This article doesn't come to tell you "they're going to wreck your bill". It comes to tell you the opposite: **use your tokens well so what you already pay is enough**. And if in some case you decide to open overage because you don't want to run out of service in a critical week, let it be a conscious decision, with a cap, knowing how much you're willing to put in. The tap isn't open by itself, but you can open it by accident if you don't look.

Where to find each setting on each platform (console paths, budget alert formats, recommended values) is in the project's technical documentation, section 8.3 (Link to technical document).

---

## Personal governance: how to measure yourself

Here comes the concrete part of the article. Because this entire conversation about the bill, BUs and companies starts at something much smaller: **every person who uses AI every day can, and should, see their own consumption.** Not the team's. Not the BU's. Their own.

I'm not proposing it as an obligation or as the only way. I'm proposing it as **the fastest way to start understanding what you consume**.

If you don't know how many sessions you opened this week, how much repeated context you sent, what models you triggered, how many input vs output tokens (which cost 4 to 6x more), then you're not doing governance, you're guessing. And the good news is that **measuring yourself in 2026 takes ten minutes of setup and zero euros**, if you use the right tools.

### Configurable tools are no longer "blind"

Before getting into the how, an important correction to something that gets said a lot. IDE tools are not closed products. They are tools with real configuration: you pay per plan, not per token, but **they expose OpenTelemetry out of the box**. What changes is who instruments: you instead of the vendor.

Tools that do expose OpenTelemetry today:

- **Claude Code:** official support. Five environment variables in your shell and it starts exporting metrics and events to any OTLP backend, including a local file on your own machine.
- **Claude Cowork (Claude Desktop) on Team and Enterprise plans** from version 1.1.4173 onwards.
- **GitHub Copilot Chat:** native settings in VSCode. Exports traces, metrics and events following GenAI Semantic Conventions (an open standard from OpenTelemetry itself). Also supports exporting to a local file.
- **Codex CLI (OpenAI):** exports structured logs and OTel metrics.

The one that still doesn't expose it natively is **Cursor**. It has Admin Dashboard with its own API and CSV exports in Enterprise, but no OTel out-of-the-box. If you need it urgently, there are community hooks (cursor-otel-hook, CursorLens as proxy) that close the gap.

The updated rule: before adopting or going deeper into an AI tool, ask if it exposes OTel. If it does, you can have real observability without needing to migrate anything to a direct API. If it doesn't, your visibility will depend on the vendor's dashboard.

### Your personal dashboard in ten minutes, all on your machine

Here comes the concrete part. Any dev who uses Claude Code or Copilot can set up their own consumption dashboard in ten minutes, **without asking IT for anything, without paying anything extra, without uploading metrics to any external service, without opening a ticket**.

The idea is this. Both Claude Code and Copilot Chat can export their telemetry to a local file in JSONL format (JSON Lines, a format where each line is an independent JSON object, standard for logs and telemetry). Each line of the file is an event (a session, a request, tokens consumed, model used, estimated cost). With that you already have everything you need to answer the personal governance questions: how many sessions per day, how much repeated context, what models, how many input vs output tokens, how much it costs you.

The only thing missing is something that reads that file and shows it to you nicely. This is **a small Python script that reads the JSONL, computes the aggregations, and generates a static HTML with charts**. You open the HTML in the browser, you look. Zero docker, zero cloud account, zero running server, zero sharing metrics with anyone.

The flow, conceptually:

1. Set the environment variables (Claude Code) or the JSON setting (Copilot Chat) so they export to a local file.
2. Use the tools normally for a week. The file fills itself, without you having to do anything.
3. Run the Python script (one command: `python dashboard.py`) that reads the file, does the math, and writes a `dashboard.html` in the same folder.
4. Open the HTML in the browser. See the consumption of the last period.
5. Repeat when you want an updated snapshot.

The most interesting thing about this: the metrics aren't approximations, they're exact data taken from the CLI itself. And they never leave your machine. The JSONL file lives on your disk, the script reads it locally, the HTML is generated locally, you look at it locally.

If this caught you, the exact how is in the project's technical documentation, section 9.6 (Link to technical document): the textual environment variables for Claude Code, the JSON settings for Copilot Chat, the complete Python script, and the HTML template with the charts to answer each of the personal governance questions.

### When it's not for you, it's for your app

The whole section above is for you using AI as a dev. **When a BU builds a product feature that calls LLMs under the hood**, there's no vendor that exports for you: you have to instrument your own code. That's where direct API with your own OTel is the only path.

The good news is that Anthropic, OpenAI and the big providers already publish OTel specs following GenAI Semantic Conventions, so you don't have to invent names. A feature with five LLM endpoints, well instrumented, gives you: consumption per endpoint, per model, per end user, per product feature. That's what later turns a conversation from "the bill went up" into "feature X went up because endpoint Z doesn't have caching".

### Why it matters that each person measures themselves

This isn't a FinOps exercise. It's something more direct: **if you can't see your own consumption, you can't improve it.** It's the same logic as an athlete with their tracker, or a dev with their profiler. Not measuring isn't neutral, it's blindness.

And for Visma, collective governance isn't built from above asking BUs for reports. It's built from below, with each person who understands their own use and adjusts. The tech lead who has a team dashboard relies on data that exists because the people on the team measured first. The BU that reports reasonable consumption to finance does so because the devs measured before. Without the first step, the rest is budget theater.

Ten minutes of setup. Zero euros of cost. The personal governance questions answered forever. If the only thing you take away from this article is setting up your own dashboard, it was worth it.

---

## The parallel that helps think about it

In the 2010s, when AWS was just maturing, a pattern repeated across many companies: someone would spin up an EC2 instance to try something, forget to shut it down, and at month-end someone would ask "what's this $4,000 in a region we don't even use?". The first cloud bills were a shock not because the cloud was expensive, but because the unit of cost had changed and nobody had updated their habits.

Compute moved from "I bought a server, I amortize it over 3 years" to "I pay per CPU-hour while it's on". The same team, with the same workloads, could spend 5x more or 5x less depending on whether someone turned off the instance on Friday.

Tokens are at that phase now. We used to pay flat per seat (at least with the most-used vendor), now we pay per use. The bill will have big variances by habit, not by workload. And as in the AWS case, the difference between those who see the change coming and those who see it later on the dashboard isn't 10%, it's multiples.

The good part: we already know how this movie ends. The companies that survived the cloud transition weren't the ones that went back to on-prem. They were the ones that learned to tag instances, to turn off what they didn't use, to choose the right tier for each workload, to instrument everything with metrics, to set budgets and alerts. **Spend well, not spend less.**

Visma's companies already went through that transition with cloud. There's no need to reinvent. The practices are the same, the asset to watch is new.

---

## What can be done right now, by level of effort

Three levels. Any Visma BU can start with the first this same week. Each level is covered in detail in the project's technical documentation.

**Zero effort (Monday morning).** Activate Auto Mode in Copilot. Activate terminal output compression in VSCode. Activate tool search for MCPs. In Cursor, make sure you're on Auto. In Claude Code, check that the project context file doesn't have 500 lines. Review and configure spending limits where applicable in your case. This is the floor. Combined estimated savings: **15 to 25%** without touching workflows. All the exact settings are in section 4 of the technical doc (Link to technical document).

**Medium effort (a sprint).** Each dev sets up their personal dashboard with the local Python script (Claude Code or Copilot Chat, depending on what they use more). If a BU has an app or feature that calls a model API, activate prompt caching. Migrate asynchronous workflows (reports, nightly classifications, embeddings, prompt evaluation in CI) to Batch API: **50% discount with no quality difference**, combinable with caching for **up to 95% savings**. Instrument the calls with OpenTelemetry following GenAI Semantic Conventions. Estimated savings: **60 to 90%** in the cost of that specific app. How to activate each thing: sections 5 and 9 of the technical doc (Link to technical document).

**Big effort (a quarter, architecture decision).** Multi-model routing: classification to Haiku, code and standard reasoning to Sonnet, complex decisions to Opus, batch and experiments to DeepSeek direct (where there's no sensitive data issue) or via Azure (where there is). RouteLLM or LiteLLM or Portkey as gateway. **Reported savings: 20 to 80% of OpEx**, depending on the current mix. This isn't for every BU, but for those that already have serious AI load in production it's the big lever. Sections 6, 7 and 8 of the technical doc cover the details (Link to technical document).

---

## What no vendor can fix for you

These three things depend on each Visma company, and on each person within it. I list them in order of impact.

**Deciding what task goes to what model.** If everything goes to the most expensive model, everything costs like the most expensive. The bill isn't set by the provider, it's set by the routing. It's an architecture decision, not a tool one.

**Cutting context.** The 62% of garbage in the bill doesn't go away with a setting. It goes away by deciding what information to send and what not, what history to compress, when to open a new session instead of continuing Monday's. Context engineering reduces costs **between 60% and 80%**. It's a skill, not a click.

**Looking at the bill, starting with your own.** Sounds trivial and it's the most important. Look at your own, look at the team's, look at the BU's, look at the monthly trajectory. **IDC projects that Global 1000 companies are underestimating their AI costs by 30% toward 2027.** That's exactly what happens when nobody looks. If there's no one with the question "what's moving it this month?", the answer is always going to be "more AI use", and that's not actionable.

---

## It's not a recipe. It's a moment.

I want to be clear about something before the actionable steps: nothing of what I wrote here is "the way" to work with AI. I'm not saying everyone has to have a Grafana dashboard, or that everyone has to migrate features to direct API, or that everyone has to do multi-model routing on Friday. I'm saying something else.

**This is the way to understand. And understanding, today, is what needs to be done.**

We had a couple of very generous years. New tools appeared every month, models were getting better and better, prices were dropping, and flat plans let us experiment without thinking twice. If you activated extended thinking "just in case", if you opened 40 sessions this week, if you left an agent running all weekend, the cost of learning from that was low. The flat bill arrived, we all kept going.

That period is ending. Not all at once, not for everyone, not on June 1st. But it's ending. And the window to learn cheap, to experiment, measure, get it wrong, adjust, without it hurting, **is exactly now**.

Those who take advantage of this window will come out the other side understanding how AI consumption is governed. They'll know what setting moves what number. They'll have intuition. They'll be the ones who in six months, when this is company policy across all Visma BUs, already have the answer. **Those who let it pass will have to learn the same thing, but with the bill running and less margin to experiment.** It's the same curve, different emotional context.

That's why this article. Not to tell you what to do, but to show you where to start learning, and to let you know that the moment to do it is this one. Becoming good at this, the best, even, isn't magic, it's going through a learning curve. The curve is there. The timing too. The only thing missing is deciding to step in.

---

## What to do this week, by role

**If you're a developer.** Monday: activate Auto Mode, output compression and tool search in VSCode. Tuesday afternoon: set up your personal OpenTelemetry dashboard with the local script. Ten minutes, zero euros, it'll change your view. Wednesday: with the dashboard already running, look at a long session of yours and observe how much context it's dragging, what models you used, how many tokens were output, what caching hit rate you have. Friday: if your team has a feature with its own API, open the ticket to activate caching and to instrument the calls with OTel.

**If you're a tech lead.** Monday: review what tools your BU uses and where spending limits apply. Confirm none is operating with default = "no cap" without knowing it. Tuesday: tell the team how to set up the personal dashboard. Collective governance starts with the individual one. Wednesday: spend an hour with the team to decide at what point of the spectrum (closed products, configurable tools, direct API) the different use cases sit. Friday: if you have something on API without OpenTelemetry, put it in the roadmap for the next sprint.

**If you make budget decisions in a BU.** Monday: ask for the dashboard of use of the last 90 days per tool. Wednesday: identify who is the named person responsible for watching the monthly trajectory. If nobody, assign it. Friday: schedule the conversation with finance to show them the June change before they ask you about it in July.

**If you're looking at Visma from above.** This is on someone. The AI bills at Visma's companies are going to change shape this year. The BUs that see the change coming will come out well-positioned. Those that don't will be explaining in July why the AI tools line went up 3x.

**Spend well, not spend less.** The June 1st change is just the event that forced us to look. What matters is the capability that gets built afterward: users who understand what they consume, teams that monitor what they pay, BUs that choose between configurable tools and direct API with criteria. That capability isn't brought by Copilot, or Cursor, or Anthropic. It's brought by the people who dare to understand what they're consuming. And it starts, literally, with each dev who sets up their dashboard one Tuesday afternoon. Not to do everything differently. To understand what they're already doing. That's the curve worth going through now, while it's still cheap.

---

## To go deeper

Each of the technical points mentioned in this article (prompt caching, OpenTelemetry, Batch API, multi-model routing, spending limits per vendor, personal dashboards with local script, enterprise observability) is covered in detail in the **project's technical documentation** (Link to technical document). There you'll find the exact snippets, the textual environment variables, the IDE settings, the complete Python script for the personal dashboard, the procurement decision matrices, and the complete reference of verified metrics with their original sources.

The idea is that this article helps you understand **what's happening, why, and where the optimization points are**. The technical documentation helps you **implement them in your specific case**. The two readings are complementary: one without the other is incomplete.
