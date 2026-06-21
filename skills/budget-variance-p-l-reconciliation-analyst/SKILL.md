---
name: budget-variance-p-l-reconciliation-analyst
description: >
  Takes actuals, a forecast or budget, and supporting invoices/POs and produces three things:
  (1) a variance narrative — plain-English explanation of every material over/under-spend with
  percent deltas, causal labeling (timing, scope-change, pricing, volume, model error), and
  a corrected forward forecast; (2) a reconciled tracker — a structured table or CSV-ready
  sheet with line-item matching, mismatch flags, unmatched invoices, and a net cash-position
  summary; (3) a quality-gate review of the budget assumptions — checking for outdated unit
  economics, missing contingency, double-counted line items, and currency/period misalignment
  before they compound into a larger discrepancy. The skill follows the Flexible Budget
  Variance framework (separating price variance from efficiency/volume variance) so stakeholders
  can tell whether a miss was a rate problem, a volume problem, or a bad plan. On-brand voice
  courtesy of brand-brain. Use whenever the user says "explain our budget variance," "reconcile
  actuals vs plan," "why are we over budget," "P&L walkthrough," "reforecast this," "where did
  the money go," "reconcile the invoices," or pastes a spend sheet asking for analysis.
---

# Budget Variance & P&L Reconciliation Analyst

Give it numbers, get a reconciled P&L story that holds up in a CFO review. This skill does not just highlight variances — it explains *why* each one happened (rate, volume, timing, or model error), flags every unmatched invoice, pressure-tests the assumptions that drove the original plan, and produces a corrected forward view you can act on.

It will not paper over a bad plan with hopeful math. If the variance reveals a structural problem — a unit-economics assumption that was wrong at inception — it says so.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's context, voice, and financial conventions (currency, fiscal calendar, naming norms). Variance narratives go to finance, ops, and exec; they must be written in the brand's exact register, not generic finance-speak.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, currency, fiscal year convention, and preferred financial voice (terse/exec vs. annotated/analyst) before proceeding.

- **`data-qa-measurement-gotcha-checker`** — gates the input data before any analysis. Catches period misalignment (actuals in calendar months vs. a fiscal-quarter budget), currency mix, duplicate rows, and attribution breaks that would make the variance arithmetic meaningless. Call before building the tracker.

- **`analytics-report-reviewer`** — reviews the finished variance narrative for unsupported claims, missing context, weak assumptions, and visualization issues before the analyst presents it upstream.

- **`initiative-business-case-writer`** *(optional, on request)* — if the variance analysis surfaces a need to re-request budget (e.g., a scope-change overage that requires incremental funding), hand off to this skill for a CFO-defensible one-pager.

- **`cfo-ready-budget-summary-slide-builder`** *(optional, on request)* — converts the reconciled tracker and variance narrative into a board-ready slide outline.

---

## How a run works

```
Step 0  Load brand context ──► brand-brain (voice + fiscal conventions)
Step 1  Validate inputs    ──► data-qa-measurement-gotcha-checker
Step 2  Reconcile          ──► line-item match: actuals vs. budget/forecast
Step 3  Decompose variances──► Flexible Budget Variance framework
Step 4  Assumption audit   ──► quality-gate review of budget inputs
Step 5  Draft narrative    ──► plain-English walkthrough + corrected reforecast
Step 6  Review             ──► analytics-report-reviewer pass
Step 7  Deliver artifacts  ──► narrative + reconciled tracker + assumption flags
```

---

## Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns the active brand's voice adjectives, banned words, and any financial conventions noted in `brand.md` (currency, fiscal year, cost-center naming). Use these throughout the narrative and tracker headers.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, currency, fiscal year convention, and preferred financial voice before proceeding.

---

## Step 1 — Validate inputs (data gate)

Before touching the numbers, call **`data-qa-measurement-gotcha-checker`**. Surface and resolve:

- **Period alignment** — actuals and budget must share the same time grain (week / month / quarter). Mismatched grains produce phantom variances; flag and normalize before proceeding.
- **Currency consistency** — multi-currency line items need a stated exchange rate and conversion date. Mark `[verify]` on any converted amount the user hasn't confirmed.
- **Duplicate rows** — same invoice or PO appearing in both an actuals export and a manual entry; flag before summing.
- **Attribution completeness** — costs split across departments or channels need a clear allocation basis; absence of one is an assumption the analyst must flag.

If critical inputs are missing (no actuals, no budget baseline), pause and ask before continuing.

---

## Step 2 — Reconcile: line-item matching

Match each actual line item to a corresponding budget line using the Flexible Budget Variance framework:

**Three-way split per line:**

| Column | Definition |
|---|---|
| Budget (Static) | Original plan at budgeted volume |
| Flexible Budget | What the budget *should* have been at *actual* volume |
| Actuals | What was actually spent/earned |

This split isolates two separate problems:
- **Volume/efficiency variance** = Flexible Budget − Static Budget (the plan assumed wrong volume)
- **Price/rate variance** = Actuals − Flexible Budget (the plan assumed wrong unit price or rate)

Build the reconciled tracker as a table with these columns:

```
| Line Item | Cost Center | Budget ($) | Flexible Budget ($) | Actuals ($) | Vol Variance ($) | Vol Var (%) | Price Variance ($) | Price Var (%) | Total Variance ($) | Total Var (%) | Flag | Invoice/PO Ref |
```

**Flags:**
- `UNMATCHED` — actual spend with no corresponding budget line (rogue PO, unplanned vendor)
- `TIMING` — spend is correct but falls in a different period than budgeted
- `SCOPE` — project scope changed after budget was set; not a model error
- `MODEL_ERROR` — original assumption was wrong (wrong CPM, wrong headcount rate, wrong conversion rate)
- `PENDING` — invoice received but not yet confirmed/paid; exclude from closed-period totals

Save the reconciled tracker to `./finance/[brand-slug]-budget-reconciliation-[period].md` (or `.csv` if the user requests export format).

---

## Step 3 — Variance narrative

Write the plain-English walkthrough. Structure:

### Summary (3–5 sentences)
Net position (over/under, dollar and percent), the 2–3 biggest drivers, and whether the overall variance is material (use >5% of total budget as the default materiality threshold unless the user specifies otherwise).

### Line-by-line explanation of material variances
For each variance above the materiality threshold:
- **Amount and direction** ($X over / $X under; Y%)
- **Root cause label** from the flag taxonomy (TIMING / SCOPE / MODEL_ERROR / UNMATCHED)
- **Plain-English cause** — one sentence; never jargon
- **Corrective action or revised assumption** — what changes in the reforecast

### Corrected reforecast
Rebuild remaining-period spend using corrected assumptions. Show:
- Revised full-year (or full-quarter) total
- Change from original budget
- Confidence level (HIGH / MEDIUM / LOW) with one-line rationale per LOW item

---

## Step 4 — Assumption quality-gate

Audit the original budget's assumptions for structural weaknesses. Check each against actuals:

| Assumption type | Check |
|---|---|
| Unit economics (CPL, CAC, CPM, ARPU, LTV) | Did they hold? If not, what was the actual? Mark `[verify]` if unconfirmable. |
| Headcount / contractor rates | Match to signed contracts or rate cards |
| Volume assumptions (impressions, clicks, seats, orders) | Actual vs. modeled; flag if delta >20% |
| Contingency % | Was one included? Was it adequate? Standard: 5–10% for discretionary; 10–15% for new channels |
| Double-counting | Same cost in two line items (e.g., agency fee + platform fee both billing for creative) |
| Currency / period scope | All line items in same currency and same accrual period? |

Flag each weakness with severity: **CRITICAL** (materially distorts the P&L), **MODERATE** (affects reforecast reliability), **LOW** (cosmetic / hygiene).

---

## Step 5 — Present and deliver

Deliver in this order:
1. **Summary narrative** (inline — exec-readable, brand-voice)
2. **Reconciled tracker table** (inline + saved to `./finance/`)
3. **Assumption quality-gate report** (inline — flags only, no padding)
4. **Corrected reforecast** (inline)

Offer to call **`analytics-report-reviewer`** for a peer-review pass before the user sends upstream.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No narrative before `brand-brain` returns. Voice and fiscal conventions override defaults.
- **Flexible Budget Variance, always.** Never report a single total-variance number without decomposing it into price and volume components. Blended variances hide the real problem.
- **Flags, not patches.** An unmatched invoice gets flagged UNMATCHED — not silently absorbed into a catch-all line. A scope change gets labeled SCOPE — not buried in MODEL_ERROR to spare someone embarrassment.
- **`[verify]` over invention.** Any converted currency, estimated accrual, or assumed rate the user hasn't confirmed gets `[verify]`. Never synthesize a "close enough" number.
- **Materiality discipline.** Don't narrate every rounding difference. Default materiality threshold is 5% of the relevant line's budget; state the threshold used.
- **Reforecast is mandatory.** A variance analysis without a corrected forward view is archaeology. Always close with what the revised full-period number looks like.
- **Data quality gates before math.** Calling `data-qa-measurement-gotcha-checker` is non-optional; bad input produces bad variance narratives that get worse the further upstream they travel.

---

## What Not to Do

- Don't present a net variance without the price/volume decomposition — it hides causation.
- Don't round unmatched invoices into miscellaneous; flag them individually.
- Don't absorb scope-change overages into MODEL_ERROR — that misattributes the problem and corrupts future planning.
- Don't invent exchange rates, accrual amounts, or unit costs; mark `[verify]` and proceed.
- Don't skip the assumption audit because "the numbers close" — a balanced P&L can still rest on broken assumptions.
- Don't reimplement brand resolution; call `brand-brain`.
- Don't deliver the narrative without first running the data-quality gate.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded; voice + fiscal conventions applied?
- `data-qa-measurement-gotcha-checker` called; period alignment, currency, and duplicates cleared?
- Three-way Flexible Budget split present (static budget / flexible budget / actuals) for every material line?
- Every variance above the materiality threshold has a root-cause label (TIMING / SCOPE / MODEL_ERROR / UNMATCHED)?
- Unmatched invoices individually flagged, not absorbed?
- Assumption quality-gate completed with CRITICAL / MODERATE / LOW severity ratings?
- Corrected reforecast included with confidence levels?
- `[verify]` on every unconfirmed rate, conversion, or accrual estimate?
- Reconciled tracker saved to `./finance/[brand-slug]-budget-reconciliation-[period].md`?
- `analytics-report-reviewer` offered (or called) for upstream submissions?
