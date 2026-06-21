---
name: segmentation-rfm-strategy-builder
description: >
  Takes an ESP schema (field list, available custom properties) or a raw orders/events export
  and produces two tightly coupled artifacts: (1) a segmentation logic spec — the named segments,
  their AND/OR filter conditions mapped to the ESP's actual field names, and a priority/override
  order for overlapping contacts; and (2) an RFM score table — Recency, Frequency, and Monetary
  scored 1–5 each, combined into named behavioral segments (Champions, Loyal, At-Risk, Hibernating,
  Lost, Promising, Need-Attention, Can't-Lose) with cut-point thresholds, a scoring formula, and a
  per-segment action playbook. The output is implementation-ready: SQL or ESP filter logic, score
  definitions, and a recommended first campaign per segment. Calls `brand-brain` to anchor segments
  to the brand's real ICP, offer mechanics, and lifecycle stage. Composes `lead-scoring-routing-model-designer`
  for PQL/sales handoff boundaries, `lifecycle-email-push-copy-reviewer` to sanity-check the
  per-segment messaging angles, and `win-back-re-engagement-campaign-builder` for the Hibernating
  and Lost tiers.
  Use when the user says "build my RFM model," "segment my list," "define my customer segments,"
  "score my contacts," "who are my champions/at-risk users," "create behavioral segments," "segment
  by recency frequency monetary," "identify churned customers," "set up lifecycle segments," or
  hands over an ESP schema or orders CSV asking what to do with it.
---

# Segmentation & RFM Strategy Builder

Give it a schema or data export, get a segmentation logic spec and a scored RFM table — both
mapped to your ESP's actual field names and anchored to your brand's real ICP. Output goes
straight into a filter condition, a SQL WHERE clause, or a Klaviyo/ActiveCampaign/Braze segment
builder — no re-translation needed.

The framework is the classic **RFM model** (Hughes, 1994; widely validated across eCommerce,
SaaS, and publisher lists) extended with a named-tier action playbook. The skill scores, names,
and prescribes — it does not write the campaign copy itself (it calls siblings for that).

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's ICP, offer mechanics, lifecycle
  stage, and voice. Segmentation labels and action angles use the brand's real data, not generic
  archetypes. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active`
  + that brand's `brand.md` directly; if none exists, ask the user for their ICP definition,
  primary offer/product, average order value, and typical repurchase cadence before proceeding.
- **`lead-scoring-routing-model-designer`** (compose, optional) — if the brand has a sales team
  or PQL threshold, call this to define the Champions→sales-handoff boundary and MQL/PQL gating
  logic that sits alongside the RFM tiers.
- **`lifecycle-journey-mapper`** (compose, optional) — for mapping RFM tiers onto the full
  lifecycle arc (Awareness → Advocacy), so segment definitions don't conflict with lifecycle
  stage gates already in use.
- **`win-back-re-engagement-campaign-builder`** (compose, downstream) — hand off the Hibernating
  and Lost tiers directly to this skill for the churn-save sequence; don't write it here.
- **`lifecycle-email-push-copy-reviewer`** (compose, downstream) — route drafted per-segment
  message angles through this reviewer before the user copies them into their ESP.

---

## How a run works

```
Step 0  Load brand context      ──► brand-brain (always first)
Step 1  Ingest + audit input    ──► schema walk / data audit; surface gaps
Step 2  Build the RFM model     ──► score definitions, cut-points, named tiers
Step 3  Build the segment spec  ──► filter conditions, priority order, ESP field map
Step 4  Write the action playbook ──► per-tier first campaign + metric to watch
Step 5  Compose downstream      ──► hand Hibernating/Lost to win-back builder if requested
Step 6  Save artifacts          ──► ./segments/[slug]-rfm.md + ./segments/[slug]-segments.md
```

---

## Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's
ICP, offer mechanics, real proof, positioning, and voice. Do not define a single segment label or
cut-point before it returns.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
brand's `brand.md` directly; if none exists, ask the user for their ICP definition, primary
offer/product, average order value, and typical repurchase cadence before proceeding.

Use the brand digest to:
- Anchor the Monetary score thresholds to the brand's real AOV range (not a generic $50/$200 split).
- Anchor the Recency cut-points to the brand's actual repurchase cycle (a SaaS monthly-subscription
  brand has a different "at-risk" window than a one-time-purchase B2B brand).
- Use the brand's ICP persona names in segment labels when they fit (e.g. "Power Users" instead of
  "Champions" if that's the brand's language).

---

## Step 1 — Ingest & audit the input

Accept any of:
- **ESP field export** — a list of available contact properties and their data types (e.g. Klaviyo
  profile properties, ActiveCampaign custom fields, Braze user attributes).
- **Orders/events CSV** — headers + a few sample rows; the skill infers field types.
- **Plain English description** — "I have order date, order count, LTV, last open date."
- **Nothing yet** — ask the 4-question minimum (see *Data gaps* below).

**Data audit — flag before scoring:**

| Signal to check | Why it matters |
|---|---|
| Recency field: last purchase date vs. last email open | These measure different things; using open date for a purchase-recency model inflates Recency scores for window-shoppers |
| Frequency: order count vs. session count vs. login count | Use the metric that maps to revenue intent; session/login frequency alone overfits to power users who never buy |
| Monetary: LTV vs. AOV vs. single-order total | LTV is the right M metric for repurchase businesses; AOV for single-purchase; clarify which and why |
| Null/zero records | Contacts with zero orders must be filtered out of the RFM table or assigned a fixed floor score (score = 1,1,1) — mixing them silently corrupts every percentile |
| Recency vs. brand repurchase cycle | A 30-day Recency window is too tight for a quarterly-repurchase B2B product; calibrate to the brand's P50 inter-purchase interval |

Surface every flag before building the model. Do not paper over bad inputs with a clean-looking table.

---

## Step 2 — Build the RFM model

### The framework: RFM scoring (Hughes 1994, updated for digital)

Score each contact 1–5 on three dimensions. 5 = best. Assign via **quintile ranking** on the
active base (exclude never-purchased contacts from the ranking, assign them floor scores).

| Dimension | Definition | Calibration note |
|---|---|---|
| **R — Recency** | Days since last purchase (lower = better → higher score) | Anchor the R3/R4 boundary to the brand's P50 inter-purchase interval (from brand-brain or the data) |
| **F — Frequency** | Number of orders (or active subscription months) in the lookback window | Lookback window: use 12 months for subscription/SaaS, 24 months for low-frequency retail, TTL for brand new lists |
| **M — Monetary** | Cumulative LTV (preferred) or total spend in the lookback window | Use brand's real AOV tiers; mark thresholds `[verify]` if not confirmed from data |

**Quintile cut-points (example template — replace with brand-specific actuals):**

```
R score  5: last purchase ≤ [brand P20 recency days]
         4: ≤ [P40] days
         3: ≤ [P60] days  ← P50 inter-purchase cycle sits near here
         2: ≤ [P80] days
         1: > [P80] days

F score  5: ≥ [P80 order count] orders
         4: ≥ [P60]
         3: ≥ [P40]
         2: ≥ [P20]
         1: < [P20]

M score  5: LTV ≥ [P80 LTV] (≥ [brand AOV × F-score-5 freq])
         4: ≥ [P60 LTV]
         3: ≥ [P40 LTV]
         2: ≥ [P20 LTV]
         1: < [P20 LTV]
```

Mark every bracket that relies on user-provided data as confirmed; mark inferred brackets `[verify]`.

### Named tiers (RFM composite → segment name)

| Segment | RFM profile | Definition rule |
|---|---|---|
| **Champions** | R5, F4–5, M4–5 | Bought recently, buy often, spend the most |
| **Loyal** | R3–5, F3–5, M3–5 | Regular buyers, solid LTV, not necessarily the biggest spenders |
| **Potential Loyalists** | R4–5, F1–3, M1–3 | Recent first or second-time buyers with strong recency |
| **Promising** | R5, F1, M1–2 | Brand new, one purchase, high recency — nurture now |
| **Need Attention** | R3–4, F2–3, M2–3 | Above-average but recency and frequency fading |
| **At-Risk** | R2–3, F3–5, M3–5 | Previously high-value, haven't bought recently |
| **Can't Lose** | R1–2, F4–5, M4–5 | Were Champions; now gone silent — highest LTV at stake |
| **Hibernating** | R2–3, F1–2, M1–2 | Low frequency, fading recency, low value |
| **Lost** | R1, F1–2, M1–2 | Long since purchased, low frequency, low value — sunset candidates |

**Priority / overlap rule:** a contact matches the first tier whose conditions are satisfied top-to-bottom in this table. Document the tie-break in the segment spec.

---

## Step 3 — Build the segment spec

For every tier, produce an ESP-ready filter block. Use the field names from the user's actual
schema (Step 1). Show the logical structure in pseudocode that maps directly to the ESP's filter UI
or to a SQL WHERE clause.

**Template per segment:**

```
## [Segment Name]
RFM profile: R[range] F[range] M[range]
Size estimate: [% of active base or row count if data provided]

### ESP filter conditions (AND/OR block)
  [recency_field]  is on or after  [cutoff date derived from R-score bracket]
  AND [frequency_field]  is between  [F-lower]  and  [F-upper]
  AND [monetary_field]  is between  [M-lower]  and  [M-upper]
  AND [email_unsubscribed]  is false

### SQL equivalent (if orders table available)
  WHERE days_since_last_order BETWEEN [x] AND [y]
    AND order_count BETWEEN [a] AND [b]
    AND ltv BETWEEN [p] AND [q]
    AND opted_in = true

### Exclusions
  Exclude: [any overlap-resolution rule — e.g. "exclude if already in Champions"]
```

**Segment spec summary table:**

| Segment | Size est. | R | F | M | Priority |
|---|---|---|---|---|---|
| Champions | — | 5 | 4–5 | 4–5 | 1 |
| Loyal | — | 3–5 | 3–5 | 3–5 | 2 |
| ... | | | | | |

---

## Step 4 — Per-segment action playbook

For each tier, prescribe the **first campaign** (what, why, metric to watch). Keep it directive.

| Segment | Goal | First campaign | Key metric | Downstream skill |
|---|---|---|---|---|
| **Champions** | Deepen loyalty + referral | VIP-exclusive preview or referral program invite | Referral conversion rate, repeat purchase rate | `win-back-re-engagement-campaign-builder` (not needed), `upsell-cross-sell-campaign-builder` |
| **Loyal** | Increase basket size | Cross-sell the second product tier; surface upgrade trigger | AOV delta, upgrade rate | `upsell-cross-sell-campaign-builder` |
| **Potential Loyalists** | Lock in second purchase | Post-first-purchase sequence with social proof + low-commitment next step | Day-30 repeat purchase rate | `welcome-onboarding-email-sequence-builder` |
| **Promising** | Activate before recency decays | Welcome + education sequence within 7 days | Day-7 engagement, day-30 purchase | `welcome-onboarding-email-sequence-builder` |
| **Need Attention** | Re-engage before drop-off | Personalized "we noticed" check-in + value reminder | Re-open rate, 30-day purchase | `lead-nurture-drip-builder` |
| **At-Risk** | Win back before they lapse | Urgency-framed offer with social proof; no spray discounts | Reactivation rate, unsubscribe rate | `win-back-re-engagement-campaign-builder` |
| **Can't Lose** | Emergency save | High-personalization outreach: acknowledge the gap, offer real value (not a coupon blast) | Reactivation rate vs. sunset cost | `win-back-re-engagement-campaign-builder` |
| **Hibernating** | Low-cost re-engagement or sunset | Single re-permission or value email; branch on response | Click-to-reactivate rate, list hygiene | `win-back-re-engagement-campaign-builder` |
| **Lost** | Sunset cleanly | Final sunset email (re-opt-in or unsubscribe) → remove non-responders from active sends | Deliverability improvement, list size | `win-back-re-engagement-campaign-builder` |

Adapt tier names and angles to the brand voice from brand-brain. Mark any offer detail
(discount amount, product name) `[verify]` if not confirmed from the brand digest.

---

## Step 5 — Compose downstream skills

After the model and spec are complete:
- If the user wants to action Hibernating or Lost immediately: invoke `win-back-re-engagement-campaign-builder`
  (Skill tool), passing the segment definition and the brand's churn reasons if known.
- If the brand has a sales team or PQL boundary: note the Champions/Can't-Lose handoff point and
  offer to invoke `lead-scoring-routing-model-designer`.
- Route any drafted per-segment message angles through `lifecycle-email-push-copy-reviewer` before
  the user puts them into an ESP sequence.

---

## Step 6 — Save artifacts

Save two files into the project-relative path `./segments/` (the user's CWD, never the skill
folder):

- `./segments/[brand-slug]-rfm.md` — the RFM score table, named tiers, cut-points, and
  scoring formula.
- `./segments/[brand-slug]-segments.md` — the full segment spec with per-tier ESP filter
  blocks, SQL equivalents, size estimates, and the action playbook.

Confirm in one line after saving. If saving is not possible, output inline and note the path
to use.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No cut-point, no tier label, no action angle before brand-brain returns.
  Its ICP, AOV, and repurchase cycle override any generic benchmark.
- **Quintiles over fixed thresholds.** Fixed-dollar or fixed-day thresholds break when a new brand
  runs the model. Always derive from the actual data distribution; mark inferred values `[verify]`.
- **Separate the RFM table from the segment spec.** The RFM score is a fact about the contact; the
  segment assignment is a business decision. Keep them as distinct artifacts so one can be updated
  independently.
- **Null safety.** Never silently include zero-order contacts in the RFM ranking. Floor-score them
  or exclude them — flag the choice.
- **Overlap resolution is explicit.** Every contact must be in exactly one active segment. The
  priority order is documented, not assumed.
- **Compose, don't copy.** The copy for each tier's campaign lives in sibling skills; this skill
  writes the strategic brief and hands off.
- **Real data or `[verify]`.** No invented AOV benchmarks, no made-up list sizes, no assumed
  repurchase cycles. Source from the brand digest or the user's data; mark unknowns.

---

## What Not to Do

- Don't write campaign copy here — call `win-back-re-engagement-campaign-builder`,
  `welcome-onboarding-email-sequence-builder`, or `upsell-cross-sell-campaign-builder`.
- Don't use fixed-dollar M thresholds (e.g. "$100 = M3") without anchoring to the brand's actual
  AOV — they will be wrong for most brands.
- Don't include unsubscribed or hard-bounced contacts in any active segment — always add the
  opt-in/subscription-status exclusion to every filter block.
- Don't produce an RFM table without surfacing the data audit flags from Step 1.
- Don't conflate open/click recency with purchase recency — they answer different questions.
- Don't invent segment names that don't map to a describable RFM range — every named tier must have
  an explicit R/F/M profile.
- Don't skip the overlap-resolution rule — the first engineer to build the query will ask it.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or fallback path completed)?
- Cut-points anchored to brand's real AOV and P50 inter-purchase interval (not generic benchmarks)?
- Data audit flags surfaced before the model was built?
- Null/zero-order contacts handled (excluded or floor-scored, choice documented)?
- All 9 named tiers present with explicit R/F/M profiles and priority order?
- ESP filter blocks use the user's actual field names, not generic placeholders?
- SQL equivalent provided where orders table was supplied?
- Per-segment action playbook complete: goal, first campaign, key metric, downstream skill?
- Downstream skill hand-offs called or offered for Hibernating, Lost, and any PQL boundary?
- Artifacts saved to `./segments/[brand-slug]-rfm.md` + `./segments/[brand-slug]-segments.md`?
- Every inferred threshold marked `[verify]`?
