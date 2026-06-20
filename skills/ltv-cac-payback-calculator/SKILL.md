---
name: ltv-cac-payback-calculator
description: >
  Revenue by cohort + CAC inputs → LTV, blended CAC, payback period, MRR cohort view, and a
  unit-economics narrative. Turns raw subscription or transactional data into the four unit-economics
  numbers every board, investor, and channel decision-maker needs: LTV, blended CAC, LTV:CAC ratio,
  and payback period — plus a plain-English narrative that explains what the numbers mean and what
  to act on. Handles SaaS (MRR/ARR, monthly cohorts), eCommerce (AOV + repeat-purchase), and
  hybrid models. Flags the classic attribution and data-quality traps before they corrupt the math.
  Use when the user says "calculate LTV," "what's my payback period," "LTV:CAC ratio," "unit
  economics," "cohort analysis," "blended CAC," "is my CAC sustainable," "how long to recoup
  acquisition spend," "MRR cohort view," or pastes revenue/cost data and asks what it means.
---

# LTV / CAC / Payback Calculator

Unit economics are the health panel of a growth business. Get them wrong and every channel decision, budget approval, and fundraise narrative is built on sand. This skill turns your cohort revenue data and acquisition cost inputs into the four numbers that matter — LTV, blended CAC, LTV:CAC, payback period — and then tells you what to do with them. It does not guess at your numbers; it asks for what it needs, flags every assumption, and marks unconfirmed figures `[verify]`.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's ICP, offer/pricing tiers, and positioning so the narrative and benchmarks are grounded in the real business, not a generic SaaS template.
- **`channel-roi-scorecard`** — call when the user wants CAC broken out by channel alongside unit economics; it owns channel-level spend → CPA → ROAS math.
- **`experiment-results-analyzer`** — call when a cohort split looks like an A/B test result and significance needs verifying before drawing LTV conclusions.
- **`growth-diagnostic-deep-dive`** — call when the unit-economics output surfaces a "why is LTV declining?" or "why is CAC rising?" question that needs deeper GA4/GSC investigation.
- **`funnel-drop-off-analyzer`** — call when payback analysis reveals a conversion-rate problem earlier in the funnel that is inflating effective CAC.
- **`offer-pricing-brain`** — call when the LTV:CAC ratio flags a pricing or packaging lever worth modeling.

---

## How a run works

```
Step 0  Load brand context  ──► call brand-brain
Step 1  Collect inputs       ──► cohort data, CAC inputs, model type (SaaS / eComm / hybrid)
Step 2  Validate data        ──► flag quality issues before computing
Step 3  Compute              ──► LTV → CAC → payback → LTV:CAC → MRR cohort table
Step 4  Narrative            ──► plain-English verdict + lever list
Step 5  Offer to save        ──► ./reports/unit-economics-[brand]-[date].md
```

### Step 0 — Load brand context (always first)

Invoke the `brand-brain` skill to get the active brand's ICP (segment, plan mix, billing cadence), offer mechanics (tiers, pricing, trial terms), and any known proof numbers. Use these to anchor benchmarks and calibrate the narrative. Do not produce a single output number before `brand-brain` returns.

### Step 1 — Collect inputs

Ask for only what is missing. The minimum viable input set:

**SaaS model**
| Input | Notes |
|---|---|
| Monthly cohort revenue table (cohort × month) | At least 6 months of retention per cohort; 12+ preferred |
| MRR or ARR per plan tier | Needed for weighted-average ARPU |
| Churn rate (logo or net-revenue) | Monthly; logo and NRR separately if available |
| Total S&M spend by month | Gross spend; exclude CS/onboarding unless fully loaded CAC |
| New customers acquired per month | Gross new logos only, not expansions |
| Gross margin % | Applied to LTV; defaults to [verify] if unknown |

**eCommerce / transactional model**
| Input | Notes |
|---|---|
| AOV (average order value) | Per transaction, not per customer |
| Purchase frequency or repeat-purchase data by cohort month | 12+ months ideal |
| Refund/return rate | Applied before gross margin |
| Gross margin % | Product margin, not contribution margin |
| Total S&M spend + new customers acquired (same as SaaS) | |

If the user can only provide summary numbers (single ARPU, single churn rate), proceed with those and flag assumptions explicitly.

### Step 2 — Validate the data

Before computing, run the **Data Quality Gate** (see below). Surface issues; do not silently absorb bad inputs.

### Step 3 — Compute

Run the **Weighted Cohort LTV Model** (framework below). Present results in the standard output table.

### Step 4 — Narrative

Write a 150–250 word plain-English verdict: what the numbers say about business health, which lever moves the ratio fastest, and what the payback period means for financing/channel budget decisions. Anchor to brand context from Step 0.

---

## The Weighted Cohort LTV Model (framework)

### Core formulas

```
ARPU (monthly)    = Total MRR ÷ Total active customers
                    (weight by plan mix when tiers differ materially)

Gross Margin LTV  = (ARPU × Gross Margin %) ÷ Monthly Churn Rate
                    — the standard SaaS LTV; use for payback and ratio

Cohort LTV (12m)  = Σ (cohort_month_n revenue × retention_rate_n)
                    for n = 1..12 — empirical; more accurate than formula LTV
                    when retention curves are non-linear

Blended CAC       = Total S&M Spend (period) ÷ New Customers Acquired (period)
                    — use same period; exclude trial-to-paid conversions already
                    counted in prior period's new customers

Payback Period    = Blended CAC ÷ (ARPU × Gross Margin %)
                    in months; the break-even point per acquired customer

LTV : CAC Ratio   = Gross Margin LTV ÷ Blended CAC
```

### Standard output table

| Metric | Value | Notes / Assumptions |
|---|---|---|
| ARPU (monthly) | | Weighted by plan mix? Y/N |
| Monthly churn rate | | Logo or NRR? |
| Gross margin % | | [verify] if assumed |
| **Gross Margin LTV** | | Formula-based |
| **Cohort LTV (12-month empirical)** | | If cohort data available |
| **Blended CAC** | | Period used: [date range] |
| **Payback period** | | months |
| **LTV : CAC ratio** | | |
| CAC Payback (by channel, if available) | | Call channel-roi-scorecard |

### Benchmarks (SaaS, B2B mid-market; mark `[verify]` for other segments)

| Metric | Healthy | Warning | Critical |
|---|---|---|---|
| LTV : CAC | ≥ 3× | 1.5–3× | < 1.5× |
| Payback period | < 12 months | 12–18 months | > 24 months |
| Monthly logo churn | < 2% | 2–5% | > 5% |
| NRR (net revenue retention) | > 110% | 90–110% | < 90% |

Explicitly label the segment and source when citing benchmarks. Benchmarks for eCommerce, PLG, or SMB-facing products differ materially — adjust or flag accordingly.

### MRR cohort view

When monthly cohort data is available, produce a retention heatmap table:

```
Cohort     M0    M1    M2    M3    M6    M12
Jan-25    100%   84%   78%   73%   61%   49%
Feb-25    100%   86%   80%   74%   …     …
…
```

Read across rows for per-cohort retention; read down columns for vintage comparison. Declining M1 values across newer cohorts signal acquisition quality degradation. Improving M6+ values signal product/onboarding improvements. Name both patterns if visible.

---

## Data Quality Gate (run before computing)

These are the gotchas that silently corrupt unit-economics math. Flag each before proceeding.

| Gotcha | Check | Remedy |
|---|---|---|
| Trial conversions in "new customers" | Are converted trials included in new-logo count for the same period as trial spend? | Remove trials from new-customer count in the trial-spend period, or use trial-to-paid conversion as a separate funnel metric |
| Expansion MRR inflating ARPU | Is ARPU computed on total MRR (including upsells) ÷ new customers? | Use new-customer-only ARR for CAC calc; keep expansion in NRR |
| S&M spend timing lag | CAC computed on same-month spend + same-month new customers ignores 4–8 week sales cycles | Shift spend 1 quarter forward for enterprise; use trailing 3-month average for self-serve |
| Logo vs. NRR churn confusion | Logo churn and revenue churn are different; using logo churn with negative churn products understates LTV | Confirm which churn rate is provided; prefer NRR-based LTV when expansion is significant |
| Blended CAC masking channel mix shifts | A rising blended CAC may be an organic/paid mix shift, not real deterioration | Ask for channel-split CAC; call channel-roi-scorecard if available |
| Short cohort windows | LTV calculated from 3-month cohorts systematically underestimates for products with 18-month typical retention | Note the cohort window; extrapolate with decay model only if window is too short, and flag as modeled |
| Gross margin not applied | Reporting LTV on revenue (not GM-adjusted) inflates the ratio | Always apply gross margin; mark [verify] if user hasn't confirmed it |

---

## Principles (Non-Negotiable)

- **Brand-brain first.** Load the active brand before computing anything — benchmarks and narrative must fit the actual business model.
- **Formula LTV is a floor, not the answer.** When cohort data exists, the empirical 12-month cohort LTV is more accurate; report both and note the gap.
- **Flag every assumption.** ARPU assumed? Churn assumed? Gross margin assumed? Every unconfirmed input gets `[verify]` and is listed in the Notes column.
- **Blended CAC is a summary — channel CAC is the lever.** Always offer to call `channel-roi-scorecard` when channel-split data exists.
- **Payback period drives budget, not just the ratio.** A 3× LTV:CAC ratio with a 30-month payback is a cash-flow problem, not a unit-economics win. Report both.
- **Benchmarks are segment-specific.** B2B SaaS, B2C SaaS, eCommerce, and PLG have meaningfully different healthy ranges. Label the benchmark source.

## What Not to Do

- Don't compute before brand-brain returns, and don't proceed if the Data Quality Gate has unfixed critical issues (expansion MRR in new customers, wrong churn type) — the output will mislead.
- Don't report LTV without applying gross margin; don't present formula LTV as definitive when cohort data exists.
- Don't use same-period spend and same-period new customers without checking for sales cycle lag.
- Don't cite benchmark ranges without labeling the segment they apply to.
- Don't produce a narrative that ignores a bad LTV:CAC ratio — name it and diagnose the lever.
- Don't save results to the skill folder or to brand.md; output to `./reports/` only.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and brand context loaded before computing?
- Data Quality Gate run; all critical gotchas surfaced (not silently absorbed)?
- Both formula LTV and cohort LTV computed (or one clearly labeled as unavailable and why)?
- Gross margin applied to LTV; all unconfirmed inputs marked `[verify]`?
- Payback period and LTV:CAC both reported; benchmarks labeled with segment?
- MRR cohort table included when cohort data was available?
- Narrative covers the verdict, the fastest-moving lever, and the payback implication for budgeting?
- Offered to save to `./reports/unit-economics-[brand]-[date].md`?
