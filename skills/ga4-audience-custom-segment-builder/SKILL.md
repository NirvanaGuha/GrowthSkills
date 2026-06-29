---
name: ga4-audience-custom-segment-builder
description: >
  Converts a behavior goal into a complete, copy-paste-ready GA4 audience or segment
  configuration — covering standard audiences, sequence-based segments, and GA4 predictive
  audiences — ready to export to Google Ads or use in Explorations. Handles the full
  stack: condition logic, event parameters, time windows, membership duration, exclusion
  rules, and the platform gotchas (sampling, (not set), attribution window mismatch,
  SRM risk when using audiences in experiments, predictive eligibility thresholds).
  Produces a real GA4 Audience Builder spec (not prose advice) plus a matching
  Exploration segment config where the behavior goal calls for analysis rather than
  activation. Compose with `data-qa-measurement-gotcha-checker` for a data-quality
  gate before exporting, and with `tracking-plan-taxonomy-builder-auditor` when the
  required events are missing from the tracking plan. Use when the user says "build
  a GA4 audience," "create a segment in GA4," "retargeting audience from analytics,"
  "predictive audience," "sequence segment," "Ads audience from GA4," "users who
  did X then Y," "high-value audience," "users about to purchase / churn," or hands
  you a behavior goal and asks how to target it in Google Ads.
---

# GA4 Audience & Custom-Segment Builder

Describe the behavior you want to capture; this skill produces the exact GA4 configuration
to capture it — condition by condition, with the right scope (user / session / event),
the right time windows, and a clear export path to Google Ads or Explorations.

It does not give you advice about audiences. It gives you the spec.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's ICP, offer, funnel
  milestones, and real proof before any audience is scoped. Audience definitions must
  map to the brand's actual conversion events and business model.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active`
  + that brand's `brand.md` directly; if none exists, ask the user for the brand's primary
  conversion event name, funnel stages (e.g., free trial → paid), and any custom dimensions
  in use before proceeding.
- **`data-qa-measurement-gotcha-checker`** (compose after building) — runs a data-quality
  gate on the proposed audience conditions: flags missing events, (not set) exposure,
  sampling risk, and attribution window mismatches before the audience goes live.
- **`tracking-plan-taxonomy-builder-auditor`** (when required events are absent) — if the
  audience logic depends on events not confirmed in the tracking plan, hand off here to
  spec them out rather than building on phantom data.
- **`ga4-custom-report-exploration-builder`** (optional) — when the behavior goal is
  analysis-first (Exploration) rather than activation-first (Ads export), this sibling
  builds the matching Exploration surface; call it in parallel.
- **`segmentation-rfm-strategy-builder`** (optional) — when the goal is a behavioral
  RFM split (Champions, At-Risk, Hibernating), the RFM logic from that skill maps
  cleanly into GA4 user-scoped conditions; compose rather than re-derive.
- **`attribution-model-configurator`** (optional) — audiences built for ROAS measurement
  are sensitive to attribution model; flag and compose when the user's GA4 attribution
  setting is unknown.

---

## How a run works

```
Step 0  Brand + property context  ──► brand-brain → brand digest + GA4 property ID
Step 1  Classify the goal          ──► Standard | Sequence | Predictive | Exploration
Step 2  Resolve events & params    ──► confirm events exist; flag missing ones
Step 3  Build the spec             ──► condition logic, scope, windows, membership duration
Step 4  QA gate                    ──► data-qa-measurement-gotcha-checker (or inline if absent)
Step 5  Output + export path       ──► Audience Builder config OR Exploration segment spec
```

---

### Step 0 — Brand + property context (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). Use the returned
digest to anchor: the brand's primary conversion event (purchase, trial_start, subscribe,
lead_form_submit, etc.), funnel stage map, known custom dimensions/metrics, and ICP.
Every audience must target a real event — not an assumed one.

**Fallback if brand-brain is absent or returns no brand:** read `~/.brandbrain/brands/.active`
+ that brand's `brand.md` directly; if none exists, ask the user for the brand's primary
conversion event name, funnel stages, and any custom dimensions in use before proceeding.

Ask for the GA4 property ID if not in the brand.md — it is required for the export path.

---

### Step 1 — Classify the audience goal

Four types. Map the user's goal to the right one before building.

| Type | When to use | GA4 surface |
|---|---|---|
| **Standard audience** | Single-condition or AND/OR multi-condition targeting (e.g., "users who viewed pricing but didn't convert") | Audiences → Create audience |
| **Sequence segment** | Ordered behavior: "users who did X, then Y, within N days" | Audiences → Sequence (or Exploration free-form sequence) |
| **Predictive audience** | Forward-looking propensity: purchase probability, churn probability, revenue prediction | Audiences → Predictive audiences |
| **Exploration segment** | Analysis-first — compare cohorts in Explorations, not for Ads export | Explorations → Segment |

If the goal is ambiguous between Exploration and Ads activation, default to Audience
(Ads-ready) and note the Exploration equivalent.

---

### Step 2 — The condition framework: Scope → Window → Exclusion-Guard

This is a house working model, not an official GA4 construct — it just forces the three
decisions GA4's UI lets you skip. Apply it to every condition set: every condition must be
scoped, every audience windowed, every retargeting audience exclusion-guarded.

**Scope first.** Every condition has a scope — choose the tightest scope that answers the goal.

| Scope | Counts | Use for |
|---|---|---|
| Across all sessions | User lifetime | Loyalty tiers, LTV audiences, long-term behavior |
| Within the same session | Session | Intent signals: "did X and Y in one session" |
| Within the same event | Hit | Parameter combinations: "purchase with value > $100 AND coupon used" |

**Condition building blocks:**

```
Event conditions:      event_name [is / is not / contains / matches regex]
Parameter conditions:  parameter [operator] value  (string, number, date)
User property:         user_property [operator] value
Audience membership:   already in audience [slug]   ← for suppression / lookalike seeding
Predicted:             purchase_probability / churn_probability / revenue_prediction [top N%]
```

**Time-window rules (encode these, never skip them):**
- Membership duration default is 30 days; for eCommerce conversion audiences extend to 540 days (GA4's hard max for non-Analytics-360 properties — UI caps the field there).
- For sequence segments: set the step-completion window to ≤7 days unless there is explicit evidence of a longer consideration cycle.
- For predictive audiences: GA4 evaluates propensity daily; export freshness to Google Ads is ~24 hrs. Note this lag in the spec.

**Exclusion layer (mandatory for every audience):**
Always add an exclusion condition for users who already completed the target action
(e.g., exclude `purchase` if targeting cart-abandoners). Missing exclusions are the
single most common waste driver in retargeting spend.

---

### Step 3 — Build the spec

Produce the spec in this exact format so the user can follow it step-by-step in GA4:

```
## Audience: [Name]
Property ID:  [GA4 property ID]
Export to:    Google Ads / Exploration only
Membership duration:  [N days]

### Include conditions  (AND logic between groups; OR within a group)
Group 1 — [label]
  Scope: [across all sessions | within session | within event]
  Condition 1: event [event_name]  [operator]  [value]
  Condition 2: parameter [param]   [operator]  [value]

Group 2 — [label]  (AND with Group 1)
  ...

### Exclude conditions
  event [conversion_event]  at any point in the membership window

### Sequence (if applicable)
  Step 1: [event_name + conditions]
  ──► followed by (within [N] days)
  Step 2: [event_name + conditions]
  ──► NOT followed by (within [N] days)
  Step 3: [exclusion event]

### Predictive audience (if applicable)
  Condition: [purchase_probability / churn_probability]  top [N]%
  Eligibility note: requires ≥1,000 purchasers and ≥1,000 non-purchasers in last 28 days [verify for this property]

### Data-quality flags (pre-export)
  [List any events that need confirmation, (not set) risks, sampling warnings]
```

---

### Step 4 — Data-quality gate

Before declaring the spec ready, run **`data-qa-measurement-gotcha-checker`** or apply
these inline checks if the sibling is absent:

**GA4 / Measurement gotcha checklist (always encode):**

| Gotcha | Check |
|---|---|
| Event exists in the property | Confirm event fires (DebugView or realtime report); flag if unconfirmed as `[verify]` |
| (not set) exposure | If the condition relies on a custom dimension, flag (not set) risk — GA4 returns (not set) when the parameter fires without the dimension populated |
| Sampling threshold | Audiences built on event counts < ~1,000 users/day are sampled in standard reports; use Explorations for unsampled data quality checks |
| Attribution window mismatch | GA4 default is data-driven / 30-day click; Google Ads default may differ — call out mismatch if using audience for ROAS measurement |
| SRM risk | If this audience will be used as an experiment variant arm, note SRM (Sample Ratio Mismatch) risk from audience-based assignment |
| Predictive eligibility | Predictive audiences require minimum event volume [verify]; building them on under-trafficked properties silently returns 0 users |
| Self-referral / spam contamination | If the brand has known hostname spam (common in GA4), filter with a hostname filter condition or note the contamination risk |

---

### Step 5 — Output and export path

Deliver the full spec (Step 3 format) plus:

1. **Step-by-step UI path** so a non-technical operator can build it without guessing:
   `Admin → Audiences → New audience → [type] → [Group 1 conditions] → ...`
2. **Google Ads link** (if exporting): confirm the GA4 property is linked to Google Ads;
   note that audience population takes 24–48 hrs after creation.
3. **Exploration equivalent** (one line): if the same logic as an Exploration segment would
   yield useful pre-activation analysis, name the Exploration type (Free-form / Funnel /
   Path) to pair with `ga4-custom-report-exploration-builder`.
4. **Naming convention**: `[Brand]-[Behavior]-[Window]-[Date]` e.g.
   `PE-CartAbandoners-30d-2026Q2`. Consistent naming prevents the GA4 "audience graveyard"
   (200-audience limit is real).

Save the full audience spec to `./analytics/audiences/[slug]-audience-spec.md` when
the user asks to save it; inline by default.

---

## GA4 Predictive Audiences — what they actually are

Predictive audiences are not ML magic the user configures — GA4 trains propensity models
automatically when the property meets minimum thresholds. Three models:

| Model | Predicts | Minimum requirement |
|---|---|---|
| `purchase_probability` | Likelihood to purchase in next 7 days | ≥1,000 purchasers AND ≥1,000 non-purchasers in last 28 days [verify] |
| `churn_probability` | Likelihood to NOT be active in next 7 days | Same minimum [verify] |
| `revenue_prediction` | Expected revenue in next 28 days | Subset of purchase probability model |

**When eligibility is not met**, the audience UI shows the condition greyed out with no
population. Flag this proactively — don't let the user build a dead audience.

**Best-use pattern:** combine predictive with behavioral exclusion.
Example: `purchase_probability top 10%` AND `NOT event: purchase (last 30 days)` →
high-intent new prospects, not recent buyers.

---

## Sequence segment logic — the three-step pattern

Most retargeting sequence goals fit one of three patterns. Name the pattern before building.

**Pattern A — Funnel drop (most common):**
`Step 1: page_view [page = /pricing]  →  Step 2: NOT purchase (within 7 days)`

**Pattern B — Engagement ladder:**
`Step 1: session_start (≥3 sessions)  →  Step 2: [key_feature_event]  →  Step 3: NOT trial_start (within 14 days)`

**Pattern C — Win-back (compose with `win-back-re-engagement-campaign-builder`):**
`Step 1: purchase (30–180 days ago)  →  Step 2: NOT session_start (last 30 days)`

Sequence steps in GA4 use AND logic within a step and strict temporal ordering between
steps. The "within N days" window is set per step transition, not globally.

---

## Principles

- **Spec, not advice.** Every output is a copy-paste-ready GA4 config, not a paragraph
  about what you could theoretically do.
- **Exclusions are not optional.** Every audience ships with an exclusion condition.
  Retargeting without exclusions burns budget on existing customers.
- **Scope matters more than most people think.** "Across all sessions" vs. "within the
  same session" produces radically different audience sizes — always state and justify scope.
- **Confirm events before building.** An audience built on an event that misfires or
  never fires has zero users and zero value. Flag unconfirmed events as `[verify]`.
- **Name audiences for their future selves.** Naming convention is mandatory; a 200-slot
  ceiling with 200 untitled audiences is a real problem.
- **Predictive ≠ magic.** State eligibility thresholds; if unconfirmed, mark `[verify]`.

## What Not to Do

- Don't produce a spec before brand-brain returns the brand's conversion event map.
- Don't assume events exist — confirm or flag `[verify]`.
- Don't skip the exclusion layer to keep the spec short.
- Don't conflate GA4 Audiences (for Ads activation) with Exploration segments (analysis
  only) — they are different objects built in different UI surfaces.
- Don't build predictive audiences without noting eligibility thresholds.
- Don't ignore the 200-audience limit or the 24–48 hr population lag for Ads.
- Don't re-derive RFM logic — call `segmentation-rfm-strategy-builder` if that's the goal.
- Don't run the data-quality check yourself in detail when `data-qa-measurement-gotcha-checker`
  is available — compose it.

## Quality Checklist

- `brand-brain` called; conversion event names and funnel milestones confirmed from brand.md?
- Audience type correctly classified (Standard / Sequence / Predictive / Exploration)?
- Every condition has an explicit scope (user / session / event)?
- Membership duration set and justified?
- Exclusion condition present on every audience spec?
- Predictive audiences: eligibility thresholds stated; unconfirmed marked `[verify]`?
- Data-quality gate applied (via `data-qa-measurement-gotcha-checker` or inline checklist)?
- GA4 UI step-by-step path provided?
- Naming convention applied?
- Sequence windows set per-step (not globally)?
