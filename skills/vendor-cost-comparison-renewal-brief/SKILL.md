---
name: vendor-cost-comparison-renewal-brief
description: >
  Turns vendor quotes, contracts, and usage data into a decision-ready renewal brief that a
  finance stakeholder can sign off on in one read. Accepts any combination of inputs: existing
  contract + pricing PDFs, pasted quotes, a renewal date, current utilization numbers, or a
  plain description of the vendor relationship. Produces an annualized total-cost-of-ownership
  (TCO) comparison table across current vendor and ≥1 alternative, surfaces hidden cost
  categories (overages, implementation, migration, training, lock-in penalties), flags feature
  gaps and contractual risks, scores each option on a keep / renegotiate / switch rubric, and
  assembles a concise executive brief with negotiation talking points tailored to the renewal
  conversation. Built around the Gartner Strategic Sourcing Framework (TCO → Fit → Risk →
  Leverage) adapted for marketing and growth team software. Output saves to ./vendors/. Use
  when the user says "vendor renewal," "compare SaaS tools," "should we keep or cancel,"
  "renewal is coming up," "get a better deal," "contract review," "vendor negotiation prep,"
  "tool consolidation," or hands over quotes, invoices, or a stack of contracts and asks
  what to do.
---

# Vendor Cost Comparison & Renewal Brief

A renewal conversation is a negotiation. Win it by showing up with numbers, not intuition. This skill structures everything from raw quotes and invoices to a signed-off brief — annualized TCO table, feature-gap flags, risk scoring, and a keep/renegotiate/switch recommendation — in one pass. Output is ready to share with finance or take into the vendor call.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's voice so the executive brief and negotiation talking points are on-brand. Also confirms company size, growth trajectory, and budget lens from the brand's ICP fields, which calibrate the leverage framing.
- **`competitive-intelligence-dossier`** (optional) — if alternatives research is thin, call it for a fast market scan on competing vendors.
- **`ltv-cac-payback-calculator`** (optional) — when the vendor directly affects acquisition cost or retention, call it to quantify downstream revenue impact of switching.
- **`analytical-reasoning-toolkit`** (optional) — call for a structured cost-benefit EV calculation when the decision is genuinely close.
- **`channel-roi-scorecard`** (optional) — when the vendor is a paid or analytics tool, use it to pull the channel's revenue contribution to weigh against cost.

---

## How a run works

```
Step 0  Load brand context          ──► brand-brain (voice, size, budget lens)
Step 1  Intake + clarify            ──► parse all inputs; flag gaps; ask ≤3 targeted questions
Step 2  Build TCO model             ──► annualized, all cost categories, per option
Step 3  Score on Fit + Risk         ──► feature-gap matrix, lock-in and contractual risk flags
Step 4  Leverage assessment         ──► renewal timing, alternatives availability, usage share
Step 5  Recommendation + Brief      ──► keep / renegotiate / switch + negotiation talking points
Step 6  Save artifacts              ──► ./vendors/<slug>-renewal-brief.md
```

---

## Step 0 — Load the brand

Invoke `brand-brain` (Skill tool) before producing any numbers or copy. Use the returned digest for voice (the brief must sound like the company's internal register, not generic consulting-speak) and for company-size/growth signals (a hypergrowth startup has different leverage than a stable SMB). If brand-brain is absent: read `~/.brandbrain/brands/.active` + that brand's `brand.md`, or ask the user for company stage, headcount, and budget authority in one question.

---

## Step 1 — Intake & clarification

Accept inputs in any combination:
- Contract PDF, quote PDF, invoice screenshot, pasted pricing table
- Named vendors + rough pricing ("We pay ~$2,400/yr for Vendor A")
- Usage data ("We use 40% of our seat allocation")
- Renewal date and notice-period window
- Named alternatives ("I'm looking at Vendor B and Vendor C")
- Pain points or must-have features

Parse everything. Then identify gaps. Ask **at most 3 targeted questions** in a single batch — do not back-and-forth repeatedly. Minimum required to proceed: current annual spend and renewal date. Everything else can be estimated with `[verify]` labels.

---

## Step 2 — TCO Model (Gartner Framework: Total Cost of Ownership)

Build a table covering **all cost categories**, not just license fees. For each vendor option:

| Cost category | Current vendor | Alternative A | Alternative B |
|---|---|---|---|
| License / subscription (annualized) | | | |
| Overage / usage-based charges (est.) | | | |
| Implementation / onboarding | | | |
| Integration / custom dev | | | |
| Training / enablement | | | |
| Migration cost (one-time, amortized 3yr) | | | |
| Contract exit / termination penalty | | | |
| Admin overhead (FTE hrs × loaded rate) | | | |
| **Total annual TCO** | | | |

Rules:
- Annualize everything. Amortize one-time costs over 3 years unless the user specifies.
- Mark every number you cannot confirm from input data as `[verify]`.
- Show the delta column: Alternative vs. Current ($ and %).
- If migration cost is unknown, flag it as the single biggest risk to the switch thesis.

---

## Step 3 — Fit & Risk Scoring (Framework: Fit + Risk axes)

### Feature-gap matrix

List the user's must-have and nice-to-have features. Score each vendor: Full / Partial / Missing. Flag any gap that is a hard blocker for the recommendation.

### Contractual risk flags

Check for and surface:
- Auto-renewal window (days of notice required)
- Price escalation clauses
- Data portability / export rights
- Vendor-lock mechanisms (proprietary formats, API restrictions)
- Minimum-commitment or seat-ratchet clauses
- Support SLA and penalty terms

Score overall risk: Low / Medium / High per option. A "low cost" alternative with high lock-in is not automatically a win.

---

## Step 4 — Leverage Assessment

Negotiation leverage comes from four sources. Assess each:

1. **Renewal timing** — How many days until the auto-renewal deadline? Inside 30 days is weak; 90+ days is strong.
2. **Alternatives credibility** — Are alternatives real (quoted, trialed) or theoretical? Vendors pressure-test this.
3. **Usage concentration** — What share of workflow depends on this vendor? Low usage = easy walk-away signal. High usage = be honest about switching cost.
4. **Relationship + spend tier** — Are you a named account, a volume-discount tier, or a small fish? Tier affects what the vendor will actually move on (price vs. terms vs. features).

Summarize in 2–3 sentences. This is the context for the talking points.

---

## Step 5 — Recommendation & Negotiation Brief

### Verdict

Choose one: **Keep (renew as-is) · Renegotiate · Switch**. State it in one sentence. Give the top reason.

Criteria:
- **Keep**: TCO difference < 15%, no critical feature gaps, high switching cost, short notice window.
- **Renegotiate**: Current vendor has gaps or a 15–40% TCO premium, but alternatives have real migration friction or the relationship has leverage hooks.
- **Switch**: Alternatives show > 40% TCO savings OR a hard-blocker feature gap, AND switching cost is quantifiable and manageable.

### Executive brief (shareable with finance)

```
## Vendor Renewal Brief — [Vendor Name]
Prepared: [date]  |  Renewal date: [date]  |  Notice deadline: [date]

### Summary
[2–3 sentences: current cost, recommended action, expected outcome]

### Cost comparison
[TCO table — condensed to the key rows for a non-technical reader]

### Key risks
[3 bullets max]

### Recommendation
[Keep / Renegotiate / Switch — one paragraph with rationale]

### Next steps
[3 action items with owner and date]
```

### Negotiation talking points

Write 4–6 talking points calibrated to the leverage assessment. Format:

> **Point:** [The specific ask — price reduction, extended term, feature addition, SLA improvement]
> **Anchor:** [Data or leverage source that justifies it]
> **Walk-away:** [What you will do if they decline — only include if credible]

Do not invent leverage. If the walk-away is not credible, omit it or label it as aspirational.

---

## Step 6 — Save artifacts

Write the full brief to `./vendors/<vendor-slug>-renewal-brief.md`. If a `./vendors/` directory does not exist, create it. Confirm the path in one line at the end of the run. The TCO model can be saved as a companion `./vendors/<vendor-slug>-tco.md` if the table is extensive.

---

## Principles

- **TCO before price.** Headline license cost is the least useful number. Migration, integration, and admin overhead routinely flip the decision.
- **Mark what you don't know.** Every estimated number is `[verify]`. An overconfident brief that later breaks in a finance review destroys credibility.
- **Honest leverage.** Never write a walk-away threat that the user won't actually use. Vendors remember bluffs; they erode future leverage.
- **One recommendation.** Present options clearly, then commit to a verdict. A brief that says "it depends" is not a brief.
- **Renewal window is the clock.** Surface the notice deadline immediately if the user is close to it — missing it can auto-commit another year at full price.
- **Brand voice on all shareable output.** The brief goes to internal stakeholders; it must sound like the company, not a consultant's template.

---

## What Not to Do

- Don't produce a recommendation before the TCO table is built — gut-feel comparisons that skip hidden costs mislead.
- Don't omit migration cost because it's hard to estimate — flag it explicitly; hiding it is worse.
- Don't write negotiation talking points that the user's actual leverage doesn't support.
- Don't assume the current vendor is the right answer because switching is hard — surface the real cost of inertia.
- Don't lock the recommendation to price alone; a cheap tool with a hard lock-in clause can be the most expensive option over 3 years.
- Don't write the brief in generic consulting register — brand-brain loaded the voice for a reason.

---

## Quality Checklist

- `brand-brain` called and voice + company-size context loaded before any output?
- TCO table covers all 8 cost categories; every unconfirmed number marked `[verify]`?
- Feature-gap matrix lists must-haves and flags hard blockers?
- Contractual risk flags checked (auto-renewal window, escalation clauses, data portability)?
- Leverage assessment completed before talking points written?
- Verdict is single and committed (Keep / Renegotiate / Switch) with a one-sentence rationale?
- Executive brief is self-contained and shareable with a non-technical finance stakeholder?
- Negotiation talking points are calibrated to actual leverage — no invented walk-aways?
- Artifacts saved to `./vendors/`; path confirmed to user?
