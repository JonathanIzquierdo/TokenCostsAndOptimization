Getting paid per tool call: what happens when your MCP server has a wallet
Coinbase ships x402. Visa launches Trusted Agent Protocol. Stripe introduces the Agentic Commerce Suite. Mastercard rolls out Agent Pay. The infrastructure for agent-to-agent payments is built. So where do public APIs fit in a world where AI agents can pay per request, with no contracts, no API keys, and no partner programs?
A while back I wrote about building an MCP server around e-conomic's public APIs. A small proof of concept showing that any agent can talk to a Visma product through MCP in a few hundred lines of code. The point of that post was that wrapping a public API with an MCP server is, by now, fairly trivial.

Which makes the next question the interesting one: who pays for the API-call, and how?

One thing to get out of the way first. This isn't an article about crypto. It's a piece about the first payment rail that can economically settle sub-cent, low-latency transactions. That rail happens to run on stablecoins today. If a non-crypto equivalent showed up tomorrow, the argument would be identical.

For years, the answer to "who pays for the call" has been "whoever signed the partner agreement, billed at the end of the month". That model built the SaaS ecosystem we all benefit from. But it was designed for humans and their apps, not for agents making thousands of opportunistic calls on behalf of a customer.

Coinbase recently published a small example in their x402 repo. It's worth 10 minutes of your time:

🔗 coinbase/x402 — MCP client example

💳 What is x402
x402 revives the long-dormant HTTP status code 402 Payment Required and turns it into a real protocol. The idea is straightforward:

An agent calls a tool on an MCP server.
If the tool costs money, the server responds with 402 Payment Required and a machine-readable description of the price, the network, and the accepted token.
The agent's wallet signs a payment payload and retries the call.
The server verifies, executes, settles, and returns the result along with a transaction receipt.
That's it. No API keys, no contracts, no monthly invoicing. Just a wallet, a price tag, and a request.

🔌 Example
The Coinbase example is deliberately boring, which is the best part:

📋 Available tools:
 - ping:         A free tool that returns pong
 - get_weather:  Get current weather for a city. Requires payment of $0.001.

🆓 Test 1: ping          → Response: pong           Payment made: false
💰 Test 2: get_weather   → Amount: 1000 (USDC, Base Sepolia)
                           Approving payment...
                           Response: { "city": "San Francisco", "weather": "sunny", "temperature": 65 }
                           Payment made: true
                           📦 Transaction: 0x...

A free tool and a $0.001 tool, sitting side by side on the same server. The client discovered the price, approved it, paid it, and got the answer, all inside a single callTool round trip.

💸 Why not just use Stripe?
This is the fair question, and it's where I think x402 gets interesting for reasons that have nothing to do with crypto.

Card rails were designed for humans buying things. The unit economics reflect that: a Stripe charge costs roughly 2.9% plus $0.30, settles in days, and is acknowledged in seconds. That works beautifully at a $50 average order value. It's unviable at $0.001 per API call. The minimum fee alone is three hundred times the value of the call.

You can work around this, of course. You can build a prepaid credit system, take a card on file monthly, run a metered invoice at the end of the period, reconcile usage against a ledger. Every API company that has ever tried to monetise at granular scale has built some version of this. It is months of engineering work and a permanent ongoing cost centre. And it still doesn't solve the latency problem: the payment lives around the request, not inside it.

x402 is different in two ways that actually matter:

Settlement lives inside the request. On a rail like Base, finality is a couple of seconds and the marginal cost of a transfer is a fraction of a cent. That means the payment can happen during the tool call, not as a separate monthly reconciliation job. The economic event and the technical event collapse into the same round trip.
No permanent percentage-taker. There is no Stripe-equivalent in the middle skimming 2.9% off every call. The protocol settles peer-to-peer between the client's wallet and yours. For a $0.001 tool, this is the difference between "viable business" and "mathematically impossible".
Honest caveat: if x402 takes off, facilitators will emerge in the middle. Someone will offer compliance, fiat off-ramp, dispute resolution, consolidated invoicing. That's fine. The point is that the protocol doesn't mandate a middleman. Today, you can run end-to-end without one. Stripe doesn't offer that choice, and never will, because their business model depends on the opposite.

🤔 Why this matters for running a public API
Every SaaS with a public API is about to have the same conversation. It usually sounds like this:

"Agents are hitting our API 100x more than human integrations used to. They don't convert to paid seats. Our ecosystem partners are nervous. And we have no way to meter the value we're giving away."

The current toolkit is quite blunt: rate limits, tier gates, T&C changes, and, increasingly, just shutting the door. We've seen some of the bigger players in our space choose that last option in the last few months.

x402 points at a third path:

Metered, not gated. Instead of deciding who's allowed in, you price what they use. A cheap tool stays cheap. An expensive one prices itself in.
No onboarding tax. The developer (or agent) doesn't apply, wait, sign, or integrate a billing SDK. They bring a wallet. You ship a price.
Contracts as data, not PDFs. The price, the network, and the terms live in the 402 response. Machine-readable, versionable, and trivial to update.
AI training is a pricing question, not a policy question. If someone wants bulk access, you can charge accordingly, or not serve it at all. The mechanism is the same either way.
🛠️ How to get started implementing
If you have an MCP server wrapping a public API (and a lot of us increasingly do), here's where I'd start:

Pick one tool on your MCP server that's cheap to serve but has clear business value. A lookup, a validation, a small report.
Wrap it with the x402 middleware and publish it on a testnet. Price it at $0.001 to start. You're not trying to make money yet, you're trying to watch the flow.
Have an agent call it, and inspect the receipt end-to-end.
Then ask the uncomfortable questions: What would this tool cost if we priced it honestly? What's the marginal cost of one agent call vs. a human session? Who's the customer, the agent, the developer who built it, or the end user behind it all?
Those questions are the interesting ones. The plumbing is almost trivial.

⚠️ A few caveats, because they matter
It's early. x402 is a young protocol. Production deployments are rare, and the tooling, reference servers, and wallet UX are all still being shaped.
Crypto rails carry compliance baggage. For a regulated SaaS business, "we accept stablecoins" is not a five-minute legal review. That said, a stablecoin is just a settlement layer underneath. Nothing stops a facilitator from sitting in front of it and giving you a EUR-denominated invoice at the end of the month.
This complements partner programs, it doesn't replace them. Strategic integrations will always want contracts. x402 is for the long tail: the agents and developers who today can't get in, and whom you couldn't economically onboard even if they asked.
🌐 The rest of the industry is moving too
x402 isn't happening in isolation. The traditional payment networks are building their own versions of the same idea, just on card rails instead of crypto.

Visa shipped Trusted Agent Protocol (TAP), an open framework where AI agents prove their identity with cryptographically signed HTTP messages before transacting through Visa's existing network. Over 100 partners are building with it, and Visa expects millions of consumers to use agent-driven checkout by holiday 2026. Mastercard launched Agent Pay, tokenised card credentials that let agents transact on behalf of cardholders without a separate wallet or new card. All US Mastercard holders are expected to be enabled by end of 2026. Stripe went furthest: their Agentic Commerce Suite supports both traditional card payments via shared payment tokens and stablecoin settlement through the Machine Payments Protocol, built on the Tempo blockchain. They're already live with agent checkout in ChatGPT via Etsy and Shopify. Google published the Agent Payments Protocol (AP2), an open standard for agents to complete transactions securely.

The difference: Visa, Mastercard, and Stripe are solving for consumer purchases, an agent buying something on behalf of a user. x402 is solving for machine-to-machine API calls, an agent paying for data, compute, or access at sub-cent granularity. They're not competing. They're addressing different layers of the same shift.

🪪 The identity gap and how it's about to close
x402 solves the payment side, but it leaves an obvious question open: who's actually paying? A wallet address is pseudonymous by default. The server knows it got paid, but it doesn't know if the agent on the other end represents a legitimate customer, a competitor scraping data, or a bot farm burning through a stolen key.

Coinbase has been building the missing layers. AgentKit gives any AI agent a wallet and on-chain capabilities. Agentic Wallets, launched in February, add infrastructure purpose-built for autonomous spending with security guardrails. Together they handle the "how does an agent hold and spend money" problem.

The most interesting move came in March, when World, Sam Altman's identity project, integrated with AgentKit and x402. The idea is that every agent carries a cryptographic proof that a unique, verified human is behind it, using World ID. So now an API server can not only charge per call, but also verify that the caller is backed by a real person. No name, no email, no API key. Just a proof.

That combination, x402 for payment, AgentKit for wallet infra, World ID for identity, starts to look like a full stack for agent commerce. It's early, and nobody's running this in production at scale yet. But the pieces are there, and they fit together in a way that the card-network approaches don't even attempt.

🔮 What I think is actually happening
The shift from "humans using apps" to "agents calling tools" is going to force every API-owning company to have a point of view on three questions:

Who is our API for, now that a lot of the callers aren't human?
What's the unit economics of a single call, and does our pricing reflect it?
If we don't meter, what behaviour are we implicitly rewarding?
x402 doesn't answer any of those questions. But it's the first plausible piece of infrastructure I've seen that makes the answers implementable at the scale agents are about to operate at.

Worth a look. If you spin up the demo and want to compare notes, don't hesitate to reach out.
