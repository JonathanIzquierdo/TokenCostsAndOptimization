Choosing the right Claude plan
Following the group agreement with Anthropic announced earlier this year, volume discounts are now available to all Visma companies. This article breaks down Claude plan options, costs, and our recommendation.
Feb 4, 2026: Claude Code & Anthropic APIs now available in Visma

Feb 24, 2026: Update on Anthropic/Claude Group Agreement

Anthropic offers two commercial plans for organizations: Teams and Enterprise. Both use commercial terms, which is recommended for any business use of Claude, to ensure operating within the Anthropic terms of agreement.

Consumer plan risks. Free, Pro, and Max are consumer plans not intended for commercial software development. Companies doing so are technically operating outside the terms of their agreement with Anthropic.
Three options are available to Visma companies. Here's what each means in practice:

Option 1 — Teams Premium
$20 or $100/month, no annual commitment, up to 150 seats.

Must be purchased by each BU independently, each user gets usage included. No central management or group commitment possible. Recommended for engineers and development teams under ~150 seats.

Legally cleared for commercial use, though the terms are less favourable to Visma than the Enterprise plan available under Options 2 and 3.
The most significant differences concern:

a) Anthropic's security commitments,
b) Anthropic's right to unilaterally amend the terms,
c) the liability cap, and
d) Anthropic's obligation to notify Visma of suspected or confirmed confidentiality breaches.
You can find a more in-depth analysis of the legal differences here.

Recommended for engineers and development teams under ~150 seats. Not recommended for roles handling sensitive data (e.g. finance, M&A).

How to get started? Order yourself from Anthropic's website.


Option 2 — Enterprise, decentralized
$20/seat/month fixed (min 10) + consumption at API token rates. 12-month commitment required. Each company manages its own tenant.

Group agreement discounts apply per company, but volumes aren't aggregated at the Visma group level, so discounts are lower than what the central tenant achieves. Each company handles its own governance, provisioning, and Anthropic relationship independently.

To waive the fixed seat price, a commitment of $ 50k / year is necessary.

The added legal protections compared to Option 1 matters especially if you process sensitive or financial data, handle customer data under strict contractual obligations, or plan to embed Claude into complex workflows where unilateral term changes by Anthropic could create compliance/legal risks. See the full Vendor Risk Assessment of the Enterprise terms here.

Choose this if your company needs full control over integrations, data isolation requirements. Otherwise, Option 3 gives you the same controls at better economics.

How to get started? Directly through Anthropic, contact Mia Allgaier, miaallgaier@anthropic.com.


Option 3 — Enterprise, centralized
Common tenant: $0/seat/month + consumption at API token rates with group discounts. 12-month commitment required

Companies needing data isolation and configuration flexibility get their own separate tenant (same as Option 2). Everyone else joins the Visma common tenant — pooled management, lower overhead, and seat fees fully waived because the group has already crossed the volume threshold.

Legal terms and protections are the same as under Option 2.

The common tenant benefits from consolidated volume that individual companies might not achieve alone.

Consumption costs are tracked per company and reinvoiced accordingly.

Visma recommendation: join the common tenant. It's the most cost-efficient path for the vast majority of companies, without giving up Enterprise-grade compliance controls.

How to get started? Create a ticket at Visma Employee support


Can you mix options?
Options 1 and 3 can be used together. A common pattern: individual engineers or small teams on Teams Premium, while larger engineering orgs or platform teams join the Enterprise centralized tenant.



Enterprise plan pricing & discounts
These discounts were negotiated by HG Capital with Anthropic and are now available to all Visma companies under the group agreement (Options 2 and 3). They apply to new agreements, at renewal, or when making the required commitments — not retroactively.

Enterprise Seat (Chat+Cowork+Code) fee

$20/seat/month (list)
$14/seat/month with 20-seat minimum
$0/seat/month with >$50k annual API commitment

API token usage discounts
2% on >$100k–$500k annual spend
5% on >$500k–$1M
9% on >$1M–$5M

Additional considerations
Consuming via AWS, GCP, or Azure instead of Anthropic directly.

Visma has enterprise agreements with AWS, GCP, and Azure. Routing Claude consumption through these platforms consolidates spend into existing cloud commitments, at the same rates and discounts as the Visma group agreement with Anthropic.

Claude Code continues to work exactly the same way for engineers — same commands, same workflows. Usage data that normally appears in Anthropic's admin console is no longer available.
Additional infrastructure setup is required. For large teams or high-volume API usage the cost saving likely justifies it; for smaller teams the overhead may not be worth it.

Note: Claude is available as a managed service on AWS Bedrock and Azure AI Foundry (currently in preview). On GCP there is no native Vertex AI integration — consumption goes through the direct Anthropic API, billed via GCP.
