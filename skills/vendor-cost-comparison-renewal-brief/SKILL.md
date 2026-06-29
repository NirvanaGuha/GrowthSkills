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
  conversation. Structured as a TCO-first renewal workflow — total cost of ownership first, then
  fit and risk, then negotiation leverage — and driven by an explicit TCO-delta decision rule
  (keep under 15%, renegotiate 15–40%, switch over 40%) for marketing and growth team software.
  Output saves to ./vendors/. Use
  when the user says "vendor renewal," "compare SaaS tools," "should we keep or cancel,"
  "renewal is coming up," "get a better deal," "contract review," "vendor negotiation prep,"
  "tool consolidation," or hands over quotes, invoices, or a stack of contracts and asks
  what to do.
---

# Vendor Cost Comparison & Renewal Brief

A renewal conversation is a negotiation. Win it by showing up with numbers, not intuition. This skill structures everything from raw quotes and invoices to a signed-off brief — annualized TCO table, feature-gap flags, risk scoring, and a keep/renegotiate/switch recommendation — in one pass. Output is ready to share with finance or take into the vendor call.

The decision turns on one number: the **annualized TCO delta** between the current vendor and the best credible alternative, expressed as a percentage of current spend. That delta drives the verdict — **keep under 15%, renegotiate at 15–40%, switch above 40%** — gated by feature blockers and switching cost. Everything before the verdict (the TCO model, the fit-and-risk pass, the leverage read) exists to make that one number trustworthy and to decide whether the gate overrides it. This is a house decision rule, not a vendor-supplied or analyst-branded framework; the thresholds are starting defaults, calibrated to the brand's stage and budget lens.

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

## Step 2 — Total Cost of Ownership (TCO)

The TCO concept — counting the full lifetime cost of a system, not just its sticker price — was popularized by Gartner in the late 1980s. We use the idea, not a branded multi-stage model.

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

The headline output of this step is the **TCO delta** — the number the verdict in Step 5 runs on:

```
TCO delta % = (Current annual TCO − Best alternative annual TCO) / Current annual TCO × 100
```

A positive delta means an alternative is cheaper all-in; negative means the current vendor is already the cheap option. Compute it against the *best credible* alternative (quoted or trialed — not a theoretical option). Carry this single figure forward; it is the spine of the recommendation.

---

## Step 3 — Fit & Risk Scoring (Fit + Risk axes)

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

Run the verdict off the **TCO delta** from Step 2, then apply two gates that can override it:

| TCO delta (best alt vs. current) | Base verdict | Override gates |
|---|---|---|
| **< 15%** | Keep | A hard-blocker feature gap on the current vendor → Renegotiate or Switch regardless of cost. |
| **15–40%** | Renegotiate | If the alternative has *no* migration friction and *no* relationship hook to pull → Switch. |
| **> 40%** | Switch | Switch only if migration cost is quantified and manageable; if migration is unknown or larger than ~1 yr of the savings → Renegotiate and re-cost. |

Gate logic, in order:
1. **Feature gate** — a hard-blocker gap (Missing on a must-have) on any option removes it from contention before cost is weighed. A blocker on the *current* vendor forces at least a Renegotiate.
2. **Switching-cost gate** — a >40% delta does not earn a Switch verdict unless migration cost is quantified. An unquantified migration cost caps the verdict at Renegotiate.

The thresholds are house defaults. Tighten them for a cash-constrained early-stage brand (a 15% delta may be worth chasing) or loosen them for a stable team where switching churn is expensive — state the adjustment if you make one.

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
- **One number drives the verdict.** The TCO delta against the best credible alternative is the spine. Compute it, then let the 15% / 40% thresholds and the two gates decide — don't reverse-engineer the math to fit a verdict you already wanted.
- **Gates beat dollars.** A hard feature blocker or an unquantified migration cost overrides the cost delta. A cheaper tool you can't actually switch to, or that can't do the job, is not the cheaper option.
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
- TCO delta % computed against the best *credible* alternative and carried into the verdict?
- Verdict matches the decision table (< 15% Keep · 15–40% Renegotiate · > 40% Switch), with any feature-gate or switching-cost-gate override stated explicitly?
- Any threshold adjustment for brand stage / budget lens called out rather than applied silently?
- Verdict is single and committed (Keep / Renegotiate / Switch) with a one-sentence rationale?
- Executive brief is self-contained and shareable with a non-technical finance stakeholder?
- Negotiation talking points are calibrated to actual leverage — no invented walk-aways?
- Artifacts saved to `./vendors/`; path confirmed to user?
