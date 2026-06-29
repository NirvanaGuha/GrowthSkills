---
name: seo-cost-per-keyword-roi-calculator
description: >
  Converts raw Ahrefs or Semrush keyword/ranking data plus content production costs into a
  finance-grade SEO ROI model — cost-per-ranking-keyword, estimated traffic value, payback period,
  and a portfolio-level P&L view that a CFO can read without a glossary. Two modes: Quick (a single
  piece of content or a campaign URL with its ranking data) and Full Portfolio (a bulk export of
  ranking URLs with production costs). Outputs a ready-to-paste spreadsheet formula set AND a
  plain-English summary with the key signals. Calls brand-brain to anchor currency, pricing tier,
  and CAC/LTV context so the ROI math connects to real business numbers, not generic benchmarks.
  Composes data-qa-measurement-gotcha-checker before locking numbers, and can hand off to
  initiative-business-case-writer for CFO packaging. Use whenever the user says "how much does my
  SEO cost per keyword," "is our content spend worth it," "SEO ROI," "traffic value vs production
  cost," "cost per ranking," "justify the content budget," "SEO payback period," "content ROI
  model," or hands over an Ahrefs/Semrush export and asks for the finance view.
---

# SEO Cost-Per-Keyword ROI Calculator

Keyword rankings are assets. This skill treats them like one — assigning a unit cost (what you
spent to own each ranking), an estimated market value (what you would pay in CPC to buy the same
traffic), and a forward-looking payback period. The output is a defensible finance number, not an
SEO vanity metric.

The working model here — call it **Content Asset Economics** (our house framing, not an established framework): treat each ranking URL as a capitalized asset,
amortize its production cost over its useful life, and measure the traffic value it generates
against that cost. Borrowed from the same P&L logic finance uses for SaaS cohorts — except the
"cohort" is a content publish date and the "revenue" is avoided paid-search spend plus attributed
pipeline.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads active brand's currency, CAC/LTV benchmarks, CPC
  reference rates, and content production cost norms so the ROI model uses real business inputs,
  not generic assumptions. Fallback if brand-brain is absent or returns no brand: read
  `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user
  for their average content production cost per piece, organic traffic CPC reference rate, and
  monthly or annual content budget before proceeding.
- **`data-qa-measurement-gotcha-checker`** — runs a data-quality gate before the model locks;
  flags known Ahrefs/Semrush gotchas (position-zero inflation, branded keyword leakage, impression
  vs click-based volume, ranking fluctuation windows). Call before finalizing CPK or traffic-value
  numbers.
- **`initiative-business-case-writer`** (optional) — when the user wants CFO packaging, hand off
  the finished ROI model to this skill for a one-pager with problem, solution, and financials.
- **`budget-variance-p-l-reconciliation-analyst`** (optional) — call when the user wants to
  reconcile actuals-vs-forecast on a running SEO budget; this skill produces the ROI inputs and
  hands them over.

---

## How a run works

```
Step 0  Load brand context          ──► call brand-brain skill
Step 1  Ingest the data             ──► keyword export + cost inputs
Step 2  Data-quality gate           ──► call data-qa-measurement-gotcha-checker
Step 3  Run the Content Asset model ──► CPK, traffic value, payback
Step 4  Build the output artifact   ──► spreadsheet formulas + plain-English summary
Step 5  Handoffs                    ──► optional CFO pack via initiative-business-case-writer
```

---

## Step 0 — Load brand context (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Pull from the returned digest:
- **Currency and locale** (affects CPC norms and budget figures)
- **CAC and LTV** (to anchor an SEO-attributed conversion value where the brand has these)
- **Content production cost norms** (brand.md often has agency/freelancer rates or internal
  hourly burn; use them; else ask)
- **CPC reference rate** (if brand has run Google Ads or has a Semrush CPC column, use that;
  else default to `[verify]` placeholder)

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
brand's `brand.md` directly; if none exists, ask the user for their average content production
cost per piece, organic traffic CPC reference rate, and monthly or annual content budget before
proceeding.

Do not run the model until brand context (or the explicit user-supplied substitutes) is in hand.

---

## Step 1 — Ingest the data

### Accepted inputs

| Source | What to ask for |
|---|---|
| Ahrefs Site Explorer export | Top pages by organic traffic CSV, or Organic Keywords for a specific URL |
| Semrush Position Tracking / Organic Research | Export with columns: URL, keyword, position, search volume, CPC, traffic estimate |
| Manual paste | URL + keyword list + estimated monthly clicks + CPC values |
| Google Search Console | Queries export (position, clicks, impressions) — note: GSC gives clicks not rankings, flag the difference |

### Minimum viable inputs

For **Quick mode** (single URL): URL, primary keyword(s) and their positions, estimated monthly
organic clicks (from Ahrefs/Semrush traffic column), production cost for that piece.

For **Full Portfolio mode**: bulk export (CSV or paste) of ranking URLs with traffic estimates,
plus a cost lookup — either a uniform per-piece cost or a URL-keyed cost column.

### Clarifying questions (ask only what's missing after the brand-brain pull)

1. Production cost per piece (or cost schedule if it varies by format)
2. Content age / publish date (for amortization; default to 12-month useful life if unknown)
3. CPC reference — Semrush's CPC column, a known Google Ads average CPC, or the brand's CPC
   benchmark from brand.md
4. Conversion rate from organic landing to trial/lead/purchase (use brand's known rate or
   `[verify]` if not confirmed)
5. Whether to include attributed revenue or keep the model traffic-value-only (conservative)

---

## Step 2 — Data-quality gate

Call `data-qa-measurement-gotcha-checker` (Skill tool) and pass the raw data. Flag and correct
before proceeding:

- **Branded keyword leakage** — rankings for brand name inflate CPK artificially; strip or
  isolate them
- **Position-zero / featured snippet inflation** — position 1 in the export may be a snippet
  with lower CTR than a standard blue-link position 1; note the CTR difference
- **Volume freshness** — Ahrefs/Semrush volumes are 12-month rolling averages; seasonal spikes
  distort the annual traffic estimate
- **Ranking fluctuation window** — a point-in-time export captures a snapshot; flag if the data
  is older than 30 days
- **Impression vs. click-based volume** — Semrush traffic estimates and GSC clicks measure
  different things; never blend without noting the difference
- **Duplicate URL attribution** — the same content may rank for hundreds of keywords; the model
  should attribute cost once, not per keyword

Mark each identified gotcha in the output. The model does not proceed with unchecked branded
leakage or stale data older than 60 days without an explicit user override.

---

## Step 3 — The Content Asset Economics model

### Core metrics (define before computing)

**Cost-Per-Ranking-Keyword (CPK)**
```
CPK = Total production cost for the URL ÷ Number of ranking keywords in positions 1–20
      (or 1–10 for a tighter definition; state which cut you used)
```

**Estimated Traffic Value (ETV)** — the avoided-spend metric
```
ETV (monthly) = Sum over all ranking keywords of: (estimated monthly clicks × CPC)
ETV (annual)  = ETV monthly × 12
```
Where CPC comes from the Semrush column, brand's Google Ads reference, or the brand.md benchmark.
Mark as `[verify]` if CPC is a generic category average rather than a brand-specific number.

**Traffic-Value Payback Period**
```
Payback (months) = Production cost ÷ ETV monthly
```
A 3-month payback at median SEO traffic levels is strong. 6–12 months is typical for competitive
SaaS. Flag anything beyond 24 months as a rethink candidate (content refresh or sunset).

**SEO Content ROI (if conversion data is available)**
```
ROI = (Organic conversions × average deal value) ÷ Production cost  — 1
```
Only compute this when the brand has confirmed conversion rate and deal value from brand.md or
explicit user input. Never fill with industry-average conversion rates without marking `[verify]`.

**Portfolio CPK**
```
Portfolio CPK = Total content spend (all URLs in scope) ÷ Total ranking keywords across those URLs
```
Useful for budget defense: "We rank for X keywords; we spent $Y; cost per ranking is $Z."

### Segmentation cuts (run in Full Portfolio mode)

Slice the model by:
- **Age cohort** (0–6 months, 7–12 months, 12–24 months, 24+ months) — reveals compound returns
  from aging content
- **Format** (how-to, comparison, listicle, landing page) — surfaces best-performing content
  types for future budget allocation
- **Position band** (1–3, 4–10, 11–20) — P1–P3 keywords command higher CPC and CTR; weight them
  separately

---

## Step 4 — Output artifact

### Spreadsheet formula block

Produce a copy-ready formula set the user can drop into Google Sheets or Excel.

```
=== SEO Cost-Per-Keyword ROI Calculator ===
Column map: A=URL, B=Production_Cost, C=Publish_Date, D=Ranking_Keywords_P1_20,
            E=Monthly_Traffic_Est, F=Avg_CPC, G=Monthly_Organic_Conversions (optional)

CPK (per URL):           =B2/D2
ETV Monthly (per URL):   =E2*F2
ETV Annual (per URL):    =E2*F2*12
Payback (months):        =B2/(E2*F2)
ROI (if G available):    =(G2*[avg_deal_value])/B2-1

Portfolio CPK:           =SUM(B:B)/SUM(D:D)
Portfolio ETV (annual):  =SUMPRODUCT(E:E,F:F)*12
Portfolio payback (mo):  =SUM(B:B)/SUMPRODUCT(E:E,F:F)
```

Adapt column letters to match the user's actual sheet layout. Note any `[verify]` substitutions.

### Plain-English summary (always included)

```
## SEO Content Asset Report — [brand slug] — [date]
Data source: [Ahrefs/Semrush/GSC + export date]
URLs analyzed: [n] | Keywords in scope: [n] (P1–20)
Total content spend: [currency + amount]

Cost-Per-Ranking-Keyword:  $X (portfolio) | Best: $X for [URL] | Worst: $X for [URL]
Estimated Traffic Value:   $X/mo ($X/yr) — equivalent avoided paid-search spend
Payback period (median):   X months
[ROI: X% if conversion data available | [verify] if not]

Top 3 performers by ETV/cost ratio: [URL, ratio, insight]
Bottom 3 by payback period: [URL, months, flag]
Data quality notes: [gotchas flagged by data-qa-measurement-gotcha-checker]
[verify] items: [list any unconfirmed inputs]
```

Save the full artifact to `./seo-roi/[brand-slug]-seo-cpk-report-[YYYY-MM-DD].md` (or the user's
preferred path). Quick-mode output is inline only.

---

## Step 5 — Optional handoffs

- If the user wants a CFO-ready one-pager, invoke `initiative-business-case-writer` and pass the
  portfolio summary as the "expected outcome" input.
- If the user wants to track budget variance over time, surface the portfolio ETV and CPK figures
  as inputs to `budget-variance-p-l-reconciliation-analyst`.

---

## Principles (Non-Negotiable)

- **Brand context first.** No model runs before brand-brain returns (or the user supplies the
  explicit substitutes). The model is only as good as its cost and CPC inputs.
- **Data gate before lock.** data-qa-measurement-gotcha-checker runs before any CPK or ETV
  number is presented as final. Unreviewed branded leakage invalidates the model.
- **Mark uncertainty.** Unconfirmed CPCs, unverified conversion rates, and generic benchmarks are
  always marked `[verify]`. Never present a guess as a measured number.
- **Amortize, don't expense.** Content is a durable asset; the model reflects its useful-life
  cost, not a one-time spend hit. State the amortization window you used.
- **Actionable triage.** The output always concludes with a ranked list of rethink/refresh/invest
  signals — not just a model dump.

---

## What Not to Do

- Don't run the model without knowing the production cost — "average industry cost" is a
  placeholder, not an input; ask.
- Don't blend Semrush traffic estimates with GSC clicks without flagging the source difference.
- Don't compute ROI if conversion rate and deal value are not confirmed; surface the traffic-value
  metric instead (it's defensible on its own).
- Don't let branded keywords inflate the CPK; always strip or isolate them before presenting
  portfolio numbers to a non-SEO audience.
- Don't call this an "SEO ROI" if the model is traffic-value only — label it correctly
  ("avoided paid-search spend") and note what would be needed to compute true revenue ROI.
- Don't produce a report without `[verify]` tags on every unconfirmed input.

---

## Quality Checklist (self-review before presenting)

- [ ] brand-brain called; brand context (currency, CPC reference, content cost norms) loaded or
      user-supplied substitutes confirmed?
- [ ] data-qa-measurement-gotcha-checker run; branded leakage stripped; data freshness noted?
- [ ] CPK, ETV, and payback computed with stated inputs (not implied)?
- [ ] All unconfirmed CPCs, conversion rates, and benchmarks marked `[verify]`?
- [ ] Amortization window stated explicitly (default 12 months or brand-specific)?
- [ ] Spreadsheet formulas match the column layout described; no hardcoded values that should be
      references?
- [ ] Plain-English summary includes top performers, bottom performers, and data-quality notes?
- [ ] Artifact saved to project-relative path (not skill folder)?
- [ ] Handoff to initiative-business-case-writer offered if the user needs CFO packaging?
