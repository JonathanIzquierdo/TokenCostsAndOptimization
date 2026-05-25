# Tokens, the bill, and a budget decision (short-align version)

## The stake

**62% of the AI bill is repeated context**, the same system prompt and the same history traveling back and forth to the model. On June 1st GitHub Copilot, one of the most adopted vendors at Visma, moves from flat fee to usage-based billing, and that turns those 62% into something billable that wasn't visible before. **For the most intensive usage patterns (agentic, long sessions), the cost can reach up to ~3.5x the flat fee we were paying until now**, while for normal autocomplete usage it will probably stay within the seat's AI Credits.

This is not an article about tightening the belt on AI. **It's about every euro you put in actually working, not heating up air. Spend well, not spend less.**

---

## The 3 actions that move the needle this quarter

One line of what, one of why, one of how to start. The full development is in (Link to technical document).

**1. Activate prompt caching.** Anthropic charges cached content at 10% of the normal price, a 90% discount on each read. It's the single highest-impact optimization: an app with a stable system prompt and high volume can save 88 to 95% over the price without cache. *How to start:* if you use Claude Code or Copilot in VSCode, they do it for you, don't touch anything (documented cache reuse rate of 93% in VSCode). If your team has a product feature that calls the API directly, activate `cache_control` automatic on the system prompt this week.

**2. Assign the correct model per task.** The flagship (Claude Opus, GPT-5.5) costs up to 100x more than the small models (Haiku, GPT-5 Nano). For classification, extraction, file navigation or simple Q&A, Haiku performs the same at 1/5 the price. *How to start:* in Copilot activate **Auto Mode** (10% automatic discount, the model is chosen for you). On features with direct API, maintain a `MODEL_ASSIGNMENT` dictionary that routes by task type.

**3. Configure spending limits BEFORE June 1st.** No serious provider blows up your bill by default, but the default cap may not match your real budget. *How to start:* go to the admin console of each tool your team uses (Copilot, Cursor, Anthropic, OpenAI, Azure, Bedrock), check what's configured today, and set alerts at 50%, 80% and 100% of the month's budget. If no one is designated to watch the trajectory, assign someone.

---

## What this means for you, by role

### If you are a developer

**Why it matters to you:** the bill is no longer someone else's, part of it is attributed to individual sessions. Your long sessions, the agents you left running over the weekend, the thinking activated "just in case", all of that now has a measurable cost. And at the same time, **measuring your own consumption takes ten minutes** and requires no permissions.

**This week:**
- Monday: activate Auto Mode + compressOutput + tool search in VSCode. 15 to 25% savings without touching your workflow. Exact details in section 4 of the technical doc (Link to technical document).
- Tuesday afternoon: set up your personal dashboard with the local Python script in section 9.6 of the technical doc (Link to technical document). It reads the JSONL file Copilot exports and generates a static HTML with charts, all on your machine, without uploading metrics anywhere.
- Rest of the week: with the dashboard already running, look at one of your long sessions and observe how much repeated context you are re-sending, what models you used, how many tokens were output. Intuition is built that way.

### If you are a tech lead / engineering manager

**Why it matters to you:** heavy users (10 to 15% of the team) typically generate 60 to 70% of the spend. Without per-team visibility, that concentration is invisible and gets attributed to "more AI usage in general". When the budget shifts to usage-based, you are the one who will have to explain what moved the bill this month. Without observability, there is no concrete answer.

**The pattern to establish in your team:**

- **Basic observability:** having each dev with their personal dashboard serves two purposes at the same time. First, it helps the dev understand their use, see which sessions are expensive and optimize. Second, it helps you as a tech lead understand the team's aggregate consumption, see where it concentrates, and have concrete data for the conversation with the BU or Finance when the time comes. The good thing is that both are achieved with the same setup, without asking the dev for anything extra.

  **On how to measure, there are several ways and what matters is picking one and starting.** Native vendor dashboards (Copilot admin analytics, Cursor Admin Dashboard, Anthropic console) are available with no setup and are a good starting point if you want something running this week. CSV exports and vendor billing APIs work for monthly reports without new tooling. **In the medium term, my suggestion is OTel (OpenTelemetry, the open observability standard):** it's vendor-neutral, natively supported by Claude Code and Copilot Chat, and unifies metrics, traces and logs into a single backend. It's the most complete path and the one that scales best when more tools appear in the stack.

- **A look at the team's current usage:** map which tool is being used for which type of case, and apply the three-level spectrum (closed products / configurable tools / direct API) as a criterion to detect mismatches. If someone is leaning on consumer ChatGPT for a task that requires observability, governance or repeatability, that's a mismatch worth a conversation. If a product feature with massive programmatic usage is running without caching, that's another one. The idea is not to prescribe a single architecture, but to have visibility on what's used for what, and decide case by case if there's something to adjust.

- **A named owner** for watching the team's monthly consumption trajectory. Without a first and last name, it doesn't get watched.

**This week:** dedicate an hour with the team to do that simple mapping. What tools they use, for what cases. Write it down. You'll probably find one or two cases where there's an obvious mismatch, and that's the first conversation worth having.

### If you make budget decisions in a BU

**Why it matters to you:** your BU's "AI tools" line becomes variable starting June 1st, in a significant portion. Finance's question in July won't be "did we spend more?", it'll be "why did it go up?". The correct answer is not "because we used more AI", it's: "because such-and-such features went to production and such-and-such don't have caching activated".

**The concrete math worth doing before month-end:**
- Current Copilot seat cost, multiplied by your active population.
- Cost projection for heavy users (~10 to 15% of the team) under the new billing. If your BU has heavy users with agentic usage, the line can go up to ~3.5x for that segment. Worth knowing it before reacting.
- Separate line for product features that call direct APIs. That line is the one that scales with customer usage, not with headcount, and it's governed with engineering practices (caching, batch, routing), not with licenses.

**The budget decision:** it's not raising or lowering the general AI tools budget. It's **splitting that budget into two categories** (developer usage = scaling with headcount; programmatic usage = scaling with product) and assigning each a different governance practice. If both are in the same bucket, any cost conversation gets contaminated because you can't tell what moved what.

---

## The numbers that matter

Verified data that backs the decisions above. Sources are in the technical document (Link to technical document), reference section.

- **62% of the typical AI bill = re-sent context**, not new information. Measured across 30 teams surveyed in early 2026.
- **Output costs 4 to 6x more than input.** Limiting `max_tokens` by task type has disproportionate impact.
- **Cache reads = 10% of the normal price**, 90% discount. Break-even on the first hit.
- **VSCode prompt caching reuse rate = 93%** inside the editor.
- **Copilot Auto Mode = 10% automatic discount.** Settings without sacrificing quality for standard tasks.
- **EA + MACC discount = 23 to 28%**, but Azure has a 15 to 40% TCO overhead. **The discount is not automatic savings**: in typical scenarios it offsets the overhead but doesn't beat it. The EA is justified by compliance and procurement, not by price.
- **Heavy users (10 to 15% of the team) = 60 to 70% of the spend.** The distribution is always asymmetric.
- **Agentic usage under usage-based billing = up to ~3.5x the flat fee** we were paying before for that specific pattern. For normal autocomplete usage, much less.

---

## The moment

We had a couple of very generous years. New tools appeared every month, models were getting better and better, prices were dropping, and flat plans let us experiment without thinking twice. If you activated extended thinking "just in case", if you opened 40 sessions this week, if you left an agent running all weekend, the cost of learning from that was low.

That period is ending. Not all at once, not for everyone, not on June 1st. But it's ending. And the window to learn cheap, to experiment, measure, get it wrong, adjust, without it hurting, **is exactly now**.

Those who take advantage of this window will come out the other side understanding how AI consumption is governed: what setting moves what number, what team pattern scales, what conversation to have with Finance. They'll be the ones who in six months, when this is company policy, already have the answer. **Those who let it pass will have to learn the same thing, but with the bill running and less margin to experiment.**

---

## If you want to go deeper

The technical detail of everything above, with snippets, exact settings, the full Python script for the personal dashboard, and the procurement decision matrix, is in the project's technical document (Link to technical document). The extended version of the article, with the most frequent objections and the parallel to the cloud transition, is in (Link to extended article).

If your team or BU identifies a case where token consumption optimization can move the bill measurably, **the internal Accelerator can support the implementation**. It's a program that helps engineering teams modernize their practices and become AI-native, and in the coming months a specific module on token consumption optimization is being incorporated. **Self-service teaching modules** are also being prepared so teams can distribute the knowledge internally without depending on one-to-one sessions.

---

**Spend well, not spend less.** The June 1st change is just the event that forced us to look. What matters is the capability that gets built afterward: developers who understand what they consume, tech leads who have a pattern for their team, BUs that split the seat budget from the programmatic features budget. That capability isn't brought by Copilot, or Cursor, or Anthropic. It's brought by the people who dare to understand what they're consuming.
