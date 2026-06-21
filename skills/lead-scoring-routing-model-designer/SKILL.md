---
name: lead-scoring-routing-model-designer
description: >
  Turns raw CRM fields, product-event data, SQL definitions, and territory rules into a
  complete, opinionated lead-scoring and routing system — fast. Produces three linked
  artifacts: (1) a scored rubric mapping demographic/firmographic/behavioral/product-usage
  signals to point values with a PQL threshold; (2) a routing decision tree that routes
  MQL → SDR queue, PQL → AE direct, unqualified → nurture, and handles territory/round-robin
  logic; (3) lifecycle-stage definitions with entry/exit criteria and SLA clock start times.
  The whole system is grounded in the POCUS (Product-Led + Outbound Scoring) framework so
  sales touches high-intent accounts first, not highest-form-fills. Use when the user says
  "lead scoring," "PQL model," "lead routing," "MQL definition," "lifecycle stages," "build
  our scoring rubric," "score leads in HubSpot/Salesforce," "which leads go to sales," or
  "territory routing logic."
---

# Lead Scoring & Routing Model Designer

Most scoring models break because they reward form fills, not buying intent. This skill builds
a **POCUS model** (Product-Led + Outbound-Cued Scoring): product events carry more weight than
demographics, and routing logic is deterministic — no "assign to whoever's free" ambiguity.

Three outputs, always produced together: **rubric → routing tree → lifecycle stage definitions**.
They're useless in isolation; a rubric without routing leaves leads in a queue; routing without
stage definitions makes SLA enforcement impossible.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads ICP, offer tier structure, and any named-customer
  proof so rubric weights and routing thresholds are calibrated to the actual customer profile.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` +
  that brand's `brand.md` directly; if none exists, ask the user for ICP (company size, industry,
  tech stack), offer tiers (free/trial/paid thresholds), and the product events that correlate
  with closed-won before proceeding.
- **`icp-persona-builder`** — if ICP is thin or missing, call to enrich before weighting
  firmographic signals.
- **`segmentation-rfm-strategy-builder`** — for B2C or eCommerce contexts, compose its RFM
  segment output as the behavioral-signal input to the rubric instead of product events.
- **`lifecycle-journey-mapper`** — after the rubric is done, call to verify that lifecycle-stage
  definitions produced here align with the broader journey map already in place.
- **`hubspot-sequence-workflow-builder`** — downstream: convert the routing decision tree into
  HubSpot enrollment triggers and workflow steps.
- **`automation-workflow-designer-debugger`** — downstream: convert the decision tree into any
  other CRM/automation platform's workflow logic.

---

## How a run works

```
Step 0  Brand + ICP context   ──► call brand-brain (or fallback)
Step 1  Signal inventory       ──► enumerate available CRM fields + product events
Step 2  Build rubric           ──► POCUS weight table + MQL/PQL thresholds
Step 3  Routing decision tree  ──► deterministic path per score band + territory rules
Step 4  Lifecycle stage defs   ──► entry/exit criteria + SLA clocks
Step 5  Validation gate        ──► score one known closed-won + one known churned; calibrate
Step 6  Deliver artifacts      ──► save to ./lead-scoring/<brand-slug>-model.md
```

---

## Step 0 — Brand & ICP context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Extract:
- ICP firmographics (company size bands, industries, tech stack signals)
- Offer tiers and self-serve vs. sales-assisted thresholds
- Any known product milestones that correlate with conversion (e.g., "invited a teammate,"
  "ran first campaign," "connected CRM")
- Banned words and voice rules (used when writing the routing-SLA communication templates)

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` +
that brand's `brand.md` directly; if none exists, ask the user for ICP (company size, industry,
tech stack), offer tiers, and the 3–5 product events that best predict closed-won before proceeding.

---

## Step 1 — Signal inventory

Ask for (or infer from context) the available signal categories. Do not assume fields exist; mark
any used-but-unconfirmed signal `[verify field exists]`.

| Category | Example signals |
|---|---|
| Demographic / contact | Title seniority, buyer role (decision-maker / influencer / user) |
| Firmographic | Employee count band, revenue band, industry, tech stack (e.g. Shopify, Salesforce) |
| Behavioral / engagement | Email opens/clicks (decay-weighted), ad click → page visit, form type, webinar attended |
| Product usage (PQL signals) | Feature X activated, team invite sent, integration connected, usage > N events in 7d, upgrade-modal viewed |
| Negative / fit-disqualifiers | Free-forever plan only, competitor employee, student/personal email domain, geography OOS |

If product events are unavailable (early-stage), note it; default to behavioral + firmographic
only and flag the model as "pre-PQL — revisit when instrumentation ships."

---

## Step 2 — POCUS Scoring Rubric

### Weighting philosophy (POCUS)

Product signals > behavioral signals > firmographic signals > demographic signals.
Negative signals (disqualifiers) always override positives — a −50 cap pulls any lead below MQL
regardless of their positive score.

**Default weight bands** (adjust based on brand's ICP and product maturity):

| Signal tier | Weight range | Rationale |
|---|---|---|
| Core PQL event (e.g., "invited teammate") | +20 to +30 | Highest conversion-correlation |
| Secondary PQL event (e.g., "feature activated") | +10 to +15 | Strong but not sufficient alone |
| Behavioral — high intent (pricing page, demo request) | +15 to +20 | Shows evaluation intent |
| Behavioral — medium intent (webinar, case study) | +5 to +10 | Nurture signal |
| Firmographic fit — strong (ICP size + industry match) | +10 to +15 | Base qualification |
| Firmographic fit — partial | +5 | |
| Demographic — decision-maker title | +10 | |
| Demographic — influencer title | +5 | |
| Negative — personal email domain | −20 | |
| Negative — student / competitor / wrong industry | −50 (cap) | Hard disqualifier |
| Engagement decay | −1 per week of inactivity | Keeps recency in the score |

**Output format for the rubric:**

```
## Scoring Rubric — [Brand slug]

### Positive signals
| Signal | Field / Event | Points | Notes |
|---|---|---|---|
| ... | ... | ... | ... |

### Negative signals (disqualifiers)
| Signal | Field / Event | Points | Notes |

### Thresholds
| Stage | Score range | Definition |
|---|---|---|
| Unqualified | < [X] | Below fit/intent floor |
| MQL | [X]–[Y] | Marketing-qualified; routed to SDR |
| PQL | ≥ [Z] + ≥1 core PQL event | Product-qualified; AE direct |
| Disqualified | Any −50 cap | Route to long-nurture; do not touch |
```

Threshold defaults: MQL ≥ 40, PQL ≥ 60 + ≥1 core PQL event. Adjust based on funnel volume
targets the user provides; if none provided, use defaults and flag for calibration.

---

## Step 3 — Routing Decision Tree

Routing is deterministic. No "round-robin by default" without territory logic — that creates
SLA variance. Every branch ends in an explicit owner type, queue, or sequence.

```
Lead scored
│
├─ Hard disqualifier (−50 cap)?
│   └─► Long-nurture sequence (no SDR touch); flag for quarterly re-score
│
├─ PQL (≥ threshold + ≥1 core event)?
│   ├─ Matches named-account / ABM target list?
│   │   └─► Assign to named-AE; Slack alert within 5 min [verify SLA]; high-priority
│   ├─ Territory match?
│   │   └─► Route to territory AE; 4-hour SLA [verify]
│   └─ No territory rule?
│       └─► Round-robin AE pool; 4-hour SLA [verify]
│
├─ MQL (≥ threshold, no core PQL event)?
│   ├─ Enterprise segment (firmographic size ≥ [N] employees [verify])?
│   │   └─► SDR outbound queue; 24-hour first-touch SLA [verify]
│   ├─ SMB / self-serve segment?
│   │   └─► Automated SDR sequence (call `hubspot-sequence-workflow-builder` to build)
│   └─ Inbound demo-request form?
│       └─► AE direct even at MQL score (form is implicit PQL)
│
└─ Unqualified?
    └─► Lead-nurture drip (call `lead-nurture-drip-builder`); re-score on engagement trigger
```

Territory rules: list them explicitly. If unavailable, document the placeholder:
`[Add territory matrix: geo/segment/named accounts → owner]`

---

## Step 4 — Lifecycle Stage Definitions

Link stage definitions to the routing tree. Each stage needs: entry criteria, exit criteria,
and who owns the SLA clock.

| Stage | Entry criteria | Exit criteria | SLA owner | Notes |
|---|---|---|---|---|
| Subscriber | Email opt-in, score = 0 | Any positive signal breaks 10 pts | Marketing | Long-term nurture |
| Lead | Score ≥ 10, no disqualifier | Reaches MQL threshold | Marketing | Automated nurture eligible |
| MQL | Score ≥ [X] | SDR accepts (→ SAL) or rejects (→ Recycled) | SDR manager | SLA clock starts on assign |
| SAL (Sales-Accepted Lead) | SDR accepts, books meeting | AE qualifies (→ SQL) or recycles | SDR | 48h to first meeting [verify] |
| SQL | AE confirms fit; BANT or MEDDIC pass | Opp created | AE | Stage moves to pipeline |
| PQL | ≥ PQL threshold | AE accepts direct | AE | Bypass SDR entirely |
| Recycled | SDR/AE rejects; reason logged | Re-nurture for 30d then re-score | Marketing | Require reject reason in CRM |
| Disqualified | Hard disqualifier flag | Manual review only | RevOps | Never auto-reassign |

Note: if the brand uses MEDDIC (enterprise) vs. BANT (SMB), note which framework governs SQL
qualification and flag it for SDR training alignment.

---

## Step 5 — Validation Gate

Before delivering, run a fast sanity check using real data if available, or prompt the user:

1. **Closed-won test.** Pick 3 recent closed-won accounts. Run their known signals through the
   rubric. Do they all score PQL or MQL? If not, the thresholds are too high — recalibrate.
2. **Churned/lost test.** Pick 3 churned accounts. Do they score below MQL or have a disqualifier?
   If they score PQL, the model is miscalibrated — a key negative signal is missing.
3. **Volume sanity.** Estimate: at current lead volume, how many PQLs/month does this produce?
   If the number exceeds AE capacity, raise the PQL threshold. If it's zero, lower it.

State validation results explicitly. If real data isn't provided, flag each threshold as
`[not validated — run against closed-won/churned sample before going live]`.

---

## Step 6 — Deliver artifacts

Save to `./lead-scoring/<brand-slug>-model.md` (create the directory if needed). The file contains:
1. Scoring rubric table
2. Routing decision tree (ASCII or mermaid syntax — ask user preference)
3. Lifecycle stage definitions table
4. Validation notes
5. Implementation notes for the downstream CRM (HubSpot/Salesforce/custom); call
   `hubspot-sequence-workflow-builder` or `automation-workflow-designer-debugger` on request.

Inline quick-wins: summarize the top 3 highest-weight signals and the single most common
routing path so an SDR manager can brief the team in 5 minutes without reading the full doc.

---

## Principles (Non-Negotiable)

- **POCUS order:** product events > behavior > firmographics > demographics. Never invert.
- **Brand-brain first.** No rubric before ICP is confirmed; a scoring model calibrated to the
  wrong customer profile is worse than no model.
- **Deterministic routing.** Every scored lead has exactly one next step. "It depends" is not
  a routing rule — resolve it or document the tie-breaker explicitly.
- **Disqualifiers cap, not offset.** A −50 flag cannot be overridden by positive signals.
- **Validation before launch.** Test against closed-won and churned samples; flag all thresholds
  as unvalidated until that test runs.
- **Fields must exist.** Never design scoring around CRM fields that haven't been confirmed live;
  mark every unconfirmed field `[verify field exists]`.
- **Honest about coverage.** If product instrumentation is missing, say so and scope the model
  to available signals; don't fake PQL scoring with proxy signals without naming the gap.

---

## What Not to Do

- Don't weight email opens heavily — they're inflated by Apple MPP and bot clicks; use CTOR or
  link-click as the behavioral signal instead.
- Don't use a single MQL threshold for enterprise and SMB — company size alone changes what a
  "qualified" score means; segment the thresholds.
- Don't round-robin PQLs — they're high-intent and time-sensitive; route them to a named AE or
  the fastest-to-respond pool, not the next person in line.
- Don't produce a rubric without lifecycle definitions — scoring without stage transitions means
  no SLA enforcement and no feedback loop to marketing.
- Don't reimplement ICP derivation — call `icp-persona-builder` if ICP is missing; don't re-ask
  the same questions brand-brain already answered.
- Don't invent product-event names — list only confirmed events or mark them `[verify field exists]`.

---

## Quality Checklist (self-review before delivering)

- `brand-brain` called and ICP + product-event context loaded (or fallback executed)?
- Every CRM field and product event marked `[verify field exists]` if not confirmed by user?
- POCUS weight order respected: product signals carry more weight than demographics?
- Disqualifier cap (−50) present and non-overridable?
- Routing tree has no "TBD" terminal nodes — every path ends in an explicit owner type?
- Lifecycle stage table includes entry criteria, exit criteria, and SLA clock owner for every stage?
- Validation gate run (or flagged as unvalidated) with closed-won and churned sample logic stated?
- Artifacts saved to `./lead-scoring/<brand-slug>-model.md`?
- Top-3 highest-weight signals and primary routing path surfaced as quick-wins for the SDR manager?
