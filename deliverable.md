# DigiStay Assignment

*Taking the last 30% of hotel-software support off people*

---

## The answer

Don't build an agent. Fix the measurement, keep the runtime you already pay for, and build the four things Intercom will never build for you.
**Your 70% is probably not a resolution rate.**
Intercom counts a conversation as resolved if the customer **confirms the answer was good** **or simply exits without asking for anything else** — the second case is an "assumed resolution." The canonical hotel failure is exactly that customer: they give up in the chat and phone the front desk at 23:00. That scores as resolved. If your 70% is an Involvement Rate rather than a Resolution Rate, it isn't a quality signal at all.
**Use escalation rules, not Procedures, for everything you never automate.**
A resolution costs **$0.99** and a Procedure handoff also costs **$0.99** — but an escalation triggered by a **workspace-level Escalation Rule is not billable at all**. So a never-automate category implemented as a Procedure costs a dollar per conversation; implemented as an escalation rule it's free. A false resolution is charged, then deducted when the customer returns — quality is a cost lever, not just a quality one.
**My first deliverable is a re-baseline, not a model.**

## Why the 30% doesn't resolve

| Cause | Owner | How hard |
| --- | --- | --- |
| **Constrained by config** — escalation rule fired, workflow intercepted, usage limit hit, human stepped in | Support ops | Not hard. Free to fix. |
| **Customer asked for a person, sounded angry, or looped.** A behavioural refusal, not a knowledge failure | Product/UX | Partly easy |
| **Language mismatch** — won't answer if the customer's language differs from the content language unless real-time translation is on | One toggle | Not hard |
| **Message in a field it can't see** — tickets filed via "Create a ticket" put the problem in a ticket attribute ; the agent only answers from messages | Support ops | Not hard, and invisible |
| **Knowledge doesn't exist** | Content | Medium |
| **Knowledge exists, no authority to act** — it can say you're owed a credit, it can't issue one. **Knowledge ≠ authority** | Engineering | **This is the expensive one** |
| **Needs live customer state** across properties, plans, integrations, billing | Engineering | Hard |
| **Content is stale** — billing, plan names, tax, OTA behaviour | Content + Product | Hard |
| **Identity and scope** — chain regional manager, night manager, agency booking for someone else | Eng + Security | **Hardest** |
| **Incident** — the PMS sync died at 22:00 and the hotel is full | Never resolved by an agent |
| **Commercial or legal** — contract, pricing, security questionnaire, DPA | Never automated |
Causes 1–5 are **not capability failures**. They're hygiene, and they're probably most of your 30%. Cause 6 is the real engineering project. Causes 10–11 aren't targets.
 **The genuinely hard parts:** property-level authorisation — a hotel group isn't a tenant, and corporate admin, regional, property manager, night manager, front desk, revenue and accountant are seven roles with different rights; account-specific reasoning spanning two systems and a business-date concept; knowing when to hand over; proving a resolution actually happened; and turning a team from answerers into exception-handlers, which no code can reverse.

## What I need to confirm

| I ask | From | What it changes |
| --- | --- | --- |
| **"Which metric is 70%, exactly?"** | Whoever built the report | Determines whether I have a quality problem or a measurement problem. Everything downstream depends on it. |
| "200 consecutive escalated conversations," split by cause — user-requested vs rules-vs-guidance | Head of Support | Sets the taxonomy, therefore the build. If our own guardrails cause most handoffs, I rewrite the rules instead of building an agent. |
| "What do you close that you shouldn't have?" | Head of Support | The 70% is contaminated. If the bot is confidently wrong, more automation makes it worse. |
| "What can an automated action legally touch?" | Product + Finance | Procedures and connectors, or nothing. Sets the never-automate list. |
| Refund/credit caps and who overrides | Finance | The line between "act" and "ask." |
| "Do we ever touch card data?" | Eng + Compliance | PCI scope, architecture, logging. |
| "Do EU chains require EU-only processing?" | Sales/Legal | Intercom now offers EU-hosted Fin. Decides whether that's sufficient. |
| SLAs by segment; do strategic accounts get a human first? | CS leadership | The audience-scoped rules, and the price of the VIP carve-out. |
**I assume** the 30% is dominated by diagnosable causes rather than capability ones — usually true, and the first thing I test. **I don't know** your volumes and channel mix, so every cost figure scales off them; nor the split of the 30% across these causes; nor whether you're PCI-scoped; nor whether you're a PMS or a channel manager, which changes what data exists to reason over. I proceed by building measurement first, keeping policy in configuration rather than code, and running a read-only shadow before any Procedure touches an account.

## What it does

Fourteen categories, one rule, four hard stops.
| Category | How I handle it |
| --- | --- |
| How-to / procedural | Answer from validated content, cite it, link the article |
| Account state lookup | Read-only connector. Render the facts; don't generate them |
| Data retrieval / export | Read connector, generate artefact, honour property scope strictly |
| Reversible account change | Auto, consequence restated after |
| Reversible but touches a live property | Auto with explicit confirmation — a wrong answer here is a sold-out hotel |
| Money within cap | Auto, restate the consequence, record it |
| **Money above cap / contract / price / SLA** | Procedure gathers everything, then pauses for human approval |
| **Incident or outage** | Don't troubleshoot. Acknowledge, page a human with blast-radius context, link a status page. **Never invent an ETA** |
| **Security, legal, procurement, DSAR/erasure** | Acknowledge and route. Nothing more |
| **Complaint, churn risk, competitor named** | Warm handover with full context, immediately |
| **Strategic / VIP account** | Human first, by policy. This costs money — I make it an explicit founder decision |
| Multi-issue | Split. Resolve what's in scope, escalate the rest with what I already gathered |
| Out of scope / spam | Sales routing or silence |
**The rule, evaluated every turn — not a one-shot classifier.** I hand over if any of: the intent is never-automate; the account is human-first; the action is above the confirmation threshold without pre-approval; the answer needs a fact that's in neither validated content nor a tool; the content that would answer it is past its TTL; the customer asked twice; confidence is below the **per-intent** threshold; the conversation exceeded N turns or K tool calls; or the customer disputes a fact the system asserted.
- **The threshold is per intent, never global.** A 10% error rate on "what are your check-in hours" is fine; the same rate on "you're eligible for a full refund" is a chargeback. Confidence is weakest where consequences are largest, so high confidence plus high blast radius still confirms.
 - **I make escalation cheap and never penalise it.** One click, no argument, no delay. If "human" is a fast way to skip the bot, the 30% never moves and you can't tell the difference. I measure "asked for a human" separately from "the agent failed."
**Never automated, however capable the technology gets.** Not because a model couldn't do it — because the error is unrecoverable and the cost lands on someone who isn't us: anything touching a **live property tonight** — room availability, check-in/out, payment capture; a hotel cannot have its inventory decided by a language model. **Money above cap** and anything committing the company on price, SLA or roadmap. **Decisions about guest personal data** — a guest cannot be "almost" deleted. **Security disclosures and signed attestations.** **Authenticating a person or granting access.** **Anything during an active incident for that tenant**, where correct behaviour is to be muted , not careful. And **any price or term not read live from a source of truth** — models remember old prices confidently.
**What the human receives — a packet, not a transcript:** one-line issue summary; account and properties in scope; **verified identity and on whose authority they may act**; every tool call already made; the specific fact still unknown and the three places most likely to hold it; the last customer message verbatim; which content version was cited and whether it was within TTL; why the agent stopped; **a machine-drafted reply to edit**; risk flags and SLA clock. This turns "customer is angry, please help" into "P2, invoice variance, Plan Pro, $2.4k ARR, identity confirmed via SSO, one $340 credit pending your approval, draft attached."

## The system

Four views of the same architecture are in the Diagram tab — flow, map, blast radius and swimlanes.
** Keep — Intercom**
Already has Procedures, first-class human-in-the-loop with Slack, and audience-matched capability enumeration via MCP. Rebuilding that is the most expensive mistake available; replacing the inbox is a 6–12 month distraction.
** Build — Action Gateway**
Vendor audiences are coarse; a hotel group needs property- and role-level scope. Read and write use different credentials, every write is idempotent with a post-condition, and this is where the parallel fan-out lives.
** Build — Document registry**
Content is a projection, not a source of truth. Owners, version scope, volatility classes, expiry, and a CI gate blocking any knowledge change that regresses the golden suite.
** Build — Evaluation harness**
Vendor tests test their agent. They can't test my blast-radius policy or my multi-tenant scope. Golden set de-identified at ingestion; red-team cases in CI permanently.
** Model — vendor's choice**
I don't choose a model, so model-selection literature is irrelevant. I spend the budget on scope and evaluation. I don't self-host — the win is in the data and the workflow.
** Not built**
A second runtime, a vector database, a replacement inbox, a fine-tuned model, and an agent that writes its own documentation. Procedures are strictly sequential — which is why the Gateway does the fan-out.
**The model is not an authorization boundary.** The single highest-severity bug in a system like this is a tool that accepts a record identifier from model output and passes it to the database unscoped. The scope comes from the authenticated session, server-side; the model may supply a filter, never the scope. In a hotel group this is the difference between a regional manager resolving another property's booking and not.

## Implementation

 **Where it runs.** AWS, one region — EU if you have EU-hosted Intercom and EU chains, since the agent's AI processing already stays in the EU there. API Gateway + Lambda on the sync path, SQS for async, Postgres for the decision log, Secrets Manager, CloudTrail, Terraform. A few thousand lines, not a platform.
 **On the way in.** I verify the webhook signature, return 200 immediately, and enqueue — never process inline, because Intercom retries slow webhooks and a retried webhook is how you get a duplicate refund. Then I claim a lock keyed on conversation + message id, and **if a teammate has already replied I drop the job, permanently, for that conversation.** A queued agent reply landing after a human has answered is the number-one embarrassment and a compliance problem.
 **How it knows the product.** Three tiers, descending trust. **One: my document registry** — every article has an owner, a version scope, a last-verified date and a volatility class (billing/tax 14 days, UI 180 days, security/integrations 30 days). Only VALIDATED docs are projected into Intercom, and an expired doc cannot be answered from. **Two: executable truth** — the product's own API and permission model, which can't go stale by definition. **Three: verified staff answers**, promoted with an expiry and approved by a human. That third tier is the compounding asset — every escalated conversation is a candidate answer.
A doc edit invalidates the cache and **re-runs the golden suite; a knowledge change that regresses it is blocked from deploying.** And "I don't have a verified answer for that" is a success — my suite scores calibrated abstention as a positive.
 **How it acts.** The Action Gateway is the only thing holding a credential. Read and write use different credentials. Every write carries an idempotency key and a post-condition assertion — "resend invoice" then asserts a delivery record exists and is timestamped after the call, which stops a timeout double-charging. Every non-trivial action **restates its consequence in one line** before executing, catching both the model's intent errors and the customer's misunderstanding. Everything is append-only audited with a compensating action recorded where possible.
**Cost — the insight that inverts the usual instinct.** The dominant cost is the **$0.99 outcome fee, not tokens**. So I never route a never-automate conversation through a Procedure; I never run "bot only" mode, because it converts every escalation into a dead end; and I treat every false resolution as a double charge. Only then do I optimise model spend.
**What I record, and what I never do.** I record intent, reason code, the names of tools called and their outcomes, content IDs and versions cited, confidence, cost, outcome type, escalation reason, resolution verdict. I **never** record card numbers, guest PII from free text, credentials, full message bodies in my own store, or unredacted prompts in any third-party log. Two specifics: the agent's memory feature is beta, on by default, and retains personal details for 180 days — so I decide that explicitly, because "facts a customer shared" may include guest details. And **I de-identify the eval set at ingestion, not at export** — if the goldens are irreversibly de-identified when written, GDPR erasure collapses to a database delete.
 **Where it fails first.** One: property-level authorisation across a chain — the most likely silent failure and the worst consequence. Two: confident staleness on billing and integration setup after a PMS release. Three: hallucinated capability. Four: double side effects on retry. Five: the metric lying — automation rising because customers gave up. Six: support resistance — if nobody writes the answers any more, the content flywheel dies. Seven: seasonal peaks.

## Getting it running

| Phase | Weeks | What I do | Exposure |
| --- | --- | --- | --- |
| **0 — Re-baseline** | 1–2 | Name the metric. Export history under both definitions. Tag every escalation and split by cause. If our own guardrails dominate, rewrite the rules instead of building an agent. Turn on language coverage; ingest ticket descriptions | Zero May be most of the 30% |
| **1 — Shadow** | 3–4 | Read-only against live traffic, nothing ever sent. Humans compare | Zero |
| **2 — Read-only answers** | 5–6 | Autonomous answers, no writes, to a 5% allowlist of low-risk accounts chosen by Support | Low |
| **3 — Procedures** | 7–10 | Top automatable flows, with human-in-the-loop on every money decision | Medium |
| **4 — Capped writes** | 11–12 | Reversible actions. 100% human review of writes for two weeks, then 5% sampling | Medium |
Then I expand by intent class, one at a time. Never a big-bang switch.
 **De-risking the first real customer message:** nobody at first, then a 1% allowlist with a human watching the queue. **Disclosure is a live legal obligation** — EU AI Act Article 50 became applicable on **2 August 2026** and requires people to be told they're interacting with an AI, at the latest at the time of first interaction. The vendor now lets you **remove** the automatic "AI Agent" label, so disclosure can silently vanish, and the Messenger intro message doesn't fire for WhatsApp, Instagram, SMS or Slack. **I leave the label on and cover every channel.** Plus a one-click "talk to a person"; no autonomous write of any new type for its first 30 days, ramping suggest-and-approve → auto-with-cap → auto; global, per-intent and per-account kill switches; hard caps on turns, tool calls and cost that force handover instead of looping; and red-team drills before launch and permanently in CI — prompt injection, a cross-property request, a fabricated outage, a false customer claim.
 **What would make me withdraw it**, pre-committed with a named owner:
- **Safety — immediately, on one occurrence:** a cross-property disclosure, an unapproved money movement, anything cardholder-related, or an autonomous statement that commits the company. Kill switch, then postmortem before restart.
 - **Trust:** autonomous CSAT more than 0.3 below the human baseline over 500 conversations, sustained two weeks; or "wrong answer" reports above 3%; or repeat-contact-in-7-days up 20% relative.
 - **Progress:** handover rate not falling after two full cycles of iteration on the same intent class. That's the hard tail, and I want the money already spent.
 - **Cost:** cost per resolution more than 50% above model. **Foundations:** validated content coverage can't be held above ~90% — at that point this is a documentation problem in an AI costume, and I say so.
 - **Governance:** a regulator or a chain's security review flags the AI component, or a data-subject request reveals logged PII that shouldn't be there.
**The rule I fix now, before I see the numbers:** if confirmed resolution turns out materially below the assumed figure, I switch the metric I optimise from automation rate to **repeat-contact-in-7-days**, and the target becomes "fewer humans overall" rather than "higher resolution rate." A customer fixed on the second contact still costs us less than one fixed on the first.
 **The following months.** Months 1–2 are operations: the reason mix shifts, the easy tail moves first, the hard tail becomes visible and is usually smaller than feared. Months 3–4 the corpus starts compounding — I watch the "unknown" rate fall, because that's the real health metric. Months 5–6 the support team changes shape: hiring and training become "when to take over and how to use a draft." That's the change most programmes underweight and it isn't reversible by code. Months 6–12 the PMS and integration connectors become the moat, and the agent's "unknowns" double as a product roadmap input. Standing risk is drift toward an unexplainable black box, which I counter with per-reply citations and a replayable decision log.

## Build, buy, delegate — and cost

** Buy and keep**
Intercom wholesale, including the model underneath it. Its **$0.99 published unit price** is itself an advantage: the specialist agent platforms all sell outcome-based but publish no list price, so Intercom's number is the only one I can model a budget against.
** Build — four things**
① **Policy & Action Gateway** — authorisation, property scope, blast-radius classification, idempotency, post-conditions, audit, parallel fan-out. ② **Document registry.** ③ **Evaluation harness.** ④ **Honest measurement.**
** Hand off**
Document authoring to Support and Product SMEs, with the agent's "unknown" queue as the work list. Refund policy to Finance — the agent consumes it, never decides it. PMS connectors to the integration team.
** Refuse outright**
Replacing Intercom. Letting a model make an authorisation decision. Any incident automation. Letting the agent author its own documentation. Training a custom model.
**I make this scale-conditional, because the answer changes with volume.** Under roughly **2,000 conversations a month** I build none of the four components — I configure Intercom and spend the engineering time only on the document registry, which is the one piece with a return at any scale. Above that line, the Gateway and the evaluation harness pay back. A 400-conversation-a-month customer does not need a platform, and recommending one would be the wrong answer.
**Procurement:** SSO, SLAs, multibrand and the extended API limit are **Expert-plan only** at $139/seat monthly against $39 Essential. If enterprise chains will demand SSO, budget for Expert. The **Pro add-on** gates Topics Explorer and CX Score — and CX Score is how I see the "assumed resolution" problem without surveys.
 **Cost.** Replace the volume, keep the arithmetic. Per 1,000 conversations at today's 70/30: 300 human-handled, ~300 billable outcomes. If 60% of the 30% is automated, 120 human-handled — and the part people miss, **every never-automate escalation is $0.00, because escalation rules aren't billable.** Savings are $0.99 per human conversation eliminated, plus the human cost behind it. Everything else is a flat monthly number in the low thousands, against a one-off build of two engineers plus a support-ops person for a quarter.
 **How I frame this for the founder:** this is not primarily a cost programme. The saving is a handful of support seats. The value is **coverage at 23:00 on a Saturday** — when a hotel with 180 rooms occupied and a broken channel sync has two choices, and both are a phone call to a person. The renewal argument, not the headcount argument, pays for this.
 **What I commit to. I don't commit to zero, and I push back on the target.**
- **6 months:** human-handled contacts down **50–60%** against a re-baselined figure.
 - **12 months:** down **~70%** — roughly 9% of all contacts reach a person.
 - **100% autonomous, permanently, on a named intent set** — how-to, account state, exports, resets. That's the number a founder can rely on: "no human is ever needed to reset a password or re-send an invoice." Checkable, and worth more than a vague percentage.
 - **The metric I report:** human contacts per 1,000 accounts per month, plus repeat-contact-in-7-days. **Not deflection** — deflection is a number you can raise by failing.
**Where the remainder goes:** a three-person specialist desk — Billing & Finance, Integrations/PMS, Trust & Compliance — with the agent as copilot, not competitor. **What I refuse to automate:** incidents, security/legal, erasure requests, and money above cap. Those stay human permanently, and the better system says so out loud at 23:00 rather than pretending otherwise.

## Assumptions and gaps

**I assume** (stated because I'll proceed, not because I checked): the 30% is dominated by diagnosable causes rather than capability ones; that your support function runs on Intercom, as described; that the product touches guest and property data.
 **Unknown and load-bearing:** conversation volumes and channel mix — every cost figure scales off them; the split of the 30% across the causes above; whether you're PCI-scoped at all; customer geography; and whether you're a PMS, a channel manager, or something between, which changes what data exists to reason over.
**What I couldn't verify, stated rather than hidden.** **PCI:** the structural reasoning holds — a refund is a merchant-balance operation, not cardholder-data processing, so scope is driven by whether a card number is ever transmitted, processed or stored, and the dominant leak paths are logs, traces, the CRM retrieval path and eval fixtures, not the chat box. **But I did not read the SAQ A document itself**, so the precise eligibility wording after the March 2025 change is unverified and a QSA must read it before anyone relies on it. If you're a service provider rather than a merchant, SAQ D is the only applicable self-assessment. **EU–US Data Privacy Framework:** I don't know its current litigation status; if it falls, EU traffic needs SCCs and a transfer impact assessment, though EU-hosted Intercom shrinks that exposure. **Model token pricing:** omitted rather than guessed; the structural levers are price-independent. **Competition:** I make no comparative price claims about any agent vendor beyond the verified fact that they publish no list price. And I reject two absence-of-evidence claims — that no public prompt-injection breach of a support bot exists, and that no public cross-tenant agent incident exists — because I couldn't verify either way.
