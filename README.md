# digistay-assignment

DigiStay Founder's Office — assessment submission.

**Taking the last 30% of hotel-software support off people.** A design for a
hospitality-SaaS support agent.

## The short version

Don't build an agent. Fix the measurement, keep the runtime already in place, and
build the four things the vendor will never build for you.

Three findings drove that recommendation:

1. **The current "70% resolved" is probably not a resolution rate.** A conversation
   counts as resolved if the customer confirms the answer was good *or simply exits
   without asking for anything more*. The canonical hotel failure — the customer who
   gives up in chat and phones the front desk at 23:00 — is scored as a success.
2. **The number moves on its own.** Constrained conversations stopped counting as
   "involved" when the metric definitions changed, so the same operation can report a
   different number next month with nothing having changed.
3. **Pricing dictates the architecture.** A resolution is billed per outcome, and so is
   a procedure handoff — but a workspace-level escalation rule is not billable at all.
   So every never-automate category is implemented as an escalation rule, not a
   procedure.

The first deliverable is a re-baseline, not a model.

## Files

| File | What it is |
|---|---|
| `index.html` | **Main submission.** Self-contained, no build step, no dependencies. Three views: Document, Diagram (flow / map / blast-radius / swimlanes), Tools note. Light and dark themes, mobile and print friendly. |
| `deliverable.md` | The same document as plain markdown, for reading in a mail client or diffing. |
| `redteam-scenarios.md` | Supporting material: the nine stress-test scenarios run against the design, and the five that changed it. |

## Viewing it

Open `index.html` in any browser. It runs entirely from the filesystem — nothing to
install, nothing to build.

To serve it locally instead:

```bash
python -m http.server 8000
# then visit http://localhost:8000
```

## What the design says

- **14 query categories**, one deterministic rule evaluated every turn, and seven
  things never automated however capable the technology becomes.
- **Four components built**, four bought, and a clear list of refusals.
- **Five rollout phases** from re-baseline to capped writes, each with a stated level
  of customer exposure.
- **Pre-committed withdrawal triggers** with thresholds, and a scale-conditional
  build/buy answer that changes the recommendation under roughly 2,000 conversations
  a month.
- **Assumptions and gaps stated explicitly** rather than hidden — including what could
  not be verified and why.

## Note on sources

The built-in `web_search` tool failed for the entire working session, so all research
was done by fetching primary sources directly: vendor documentation, the EU AI Act
text, OWASP, and PCI SSC. The full list of what was verified, and what was not, is in
the Tools note tab and in §9 of the document.
