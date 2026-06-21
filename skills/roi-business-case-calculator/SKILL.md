---
name: roi-business-case-calculator
description: >
  Turns a prospect's stated metrics and a solution's impact benchmarks into a
  quantified ROI model and a champion-ready executive note — in one pass. Two
  modes: Quick (single metric, fast time-to-value dollar figure) and Full Model
  (multi-lever spreadsheet-ready model with payback period, NPV, and a
  signed-off sensitivity table). Uses the McKinsey Value Framework to bucket
  every lever into Revenue Growth, Cost Reduction, or Capital Efficiency so the
  number lands credibly with a CFO, not just a champion. Every unconfirmed
  benchmark is marked [verify]; every confirmed number is cited to source. The
  model saves to a project-relative path so it travels with the deal. Calls
  brand-brain to load voice and proof before writing any customer-facing copy.
  Composes data-qa-measurement-gotcha-checker on any pasted data before
  arithmetic, and board-exec-summary-writer for the final one-pager. Use
  whenever the user says "build an ROI model," "make a business case," "how do
  I justify this to the CFO," "what's the payback period," "help my champion
  sell internally," "ROI calculator," "value case," or hands over prospect
  metrics and asks for a number.
---

# ROI & Business Case Calculator

Give it a prospect's metrics and a set of impact levers, get a defensible ROI model and a champion-ready executive note. The model is structured around McKinsey's three value buckets — Revenue Growth, Cost Reduction, Capital Efficiency — so the math survives a CFO's first question. Every input is labeled (confirmed vs. benchmarked vs. assumed), every benchmark is sourced or flagged `[verify]`, and the sensitivity table shows the champion exactly what to argue about.

This skill quantifies and structures. It does not invent proof, inflate benchmarks, or paper over a thin value proposition. If the inputs don't support a credible number, it says so and tells the user what data to collect.

---

## Skills this calls

- **`brand-brain`** (required first) — loads the active brand's voice, proof points, and real customer outcomes used in the model's evidence column. Do not write any customer-facing copy before brand-brain returns. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, 2–3 confirmed customer proof points with metrics, and voice adjectives before proceeding.
- **`data-qa-measurement-gotcha-checker`** (recommended) — run on any pasted CSV, spreadsheet export, or CRM pull before arithmetic. Catches unit mismatches, sampling gaps, and attribution errors that would corrupt the model.
- **`proof-vault`** (optional) — pulls confirmed customer proof points and case-study metrics for the evidence column. Synthesize from brand.md if absent.
- **`board-exec-summary-writer`** (optional) — converts the completed model into a board-ready one-pager. Call after the model is reviewed and approved; do not call in the same pass that builds the model.
- **`analytics-report-reviewer`** (optional) — reviews the finished model for unsupported claims, weak assumptions, and missing context before it goes external.

---

## How a run works

```
Step 0  Load the brand          ──► call brand-brain; get voice + confirmed proof
Step 1  Gate the inputs         ──► data-qa check on any pasted data; flag dirty inputs
Step 2  Scope the mode          ──► Quick (single lever, fast answer) | Full Model
Step 3  Build the model         ──► McKinsey three-bucket structure
Step 4  Sensitivity table       ──► conservative / base / optimistic scenarios
Step 5  Write the executive note ──► champion-ready, brand-voice, proof-cited
Step 6  Save artifacts          ──► ./roi/[slug]-roi-model.md (Full Model)
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's digest — voice adjectives, banned words, confirmed proof points with metrics, positioning, ICP. Use the returned proof in the model's evidence column; mark anything else `[verify]`. Obey voice and banned words in the executive note.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, 2–3 confirmed customer proof points with metrics, and voice adjectives before proceeding.

### Step 1 — Gate the inputs

Before any arithmetic:
1. Ask for the **prospect's confirmed baseline metrics** (see *Required inputs*). If the user pastes data, invoke `data-qa-measurement-gotcha-checker` on it first.
2. Identify what is **confirmed** (prospect-provided), **benchmarked** (industry study or your own customer data), or **assumed** (modeled). Label every number in the model.
3. If key baseline metrics are missing (e.g., current revenue, headcount cost, churn rate), ask — don't guess. Name exactly what's needed and why.

---

## Mode selection

- **Quick mode (default)** — one lever, one clear dollar figure. Champion-ready in under 5 minutes. Use when: single use case, early-stage deal, prospect needs a fast "is it worth exploring?"
- **Full Model** — multi-lever spreadsheet-ready model with payback period, 3-year NPV, and sensitivity table. Use when: late-stage deal, procurement or CFO review, multi-department impact, or the user explicitly asks.

If unclear, default to Quick and offer Full Model at the end.

---

## The McKinsey Value Framework (shared by both modes)

Every lever maps to exactly one bucket. This is not decoration — it determines which executive owns the number and how hard it is to challenge.

| Bucket | What it measures | Who cares most | Example levers |
|---|---|---|---|
| **Revenue Growth** | Net new revenue or protected revenue | CRO, CMO | Higher conversion rate, lower churn, faster activation, upsell lift |
| **Cost Reduction** | Headcount, tooling, process efficiency | CFO, COO | Manual hours automated, tool consolidation, support ticket deflection |
| **Capital Efficiency** | Faster payback, lower CAC, better working capital | CFO, CEO | Shorter sales cycle, lower CAC, improved LTV:CAC |

Map each lever the user provides to its bucket before calculating. If a lever spans buckets, split it — do not double-count.

---

## Quick mode

**Input:** one baseline metric + one impact lever + desired output type (annual savings / revenue uplift / time saved).

**Output:**
```
## ROI quick estimate — [what / prospect]
Lever: [bucket] — [description]
Baseline: [metric] (source: [confirmed/benchmarked/assumed])
Impact assumption: [%] (source: [proof point or [verify]])
Annual impact: $[X] or [X] hrs/yr
Confidence: [High/Medium/Low] — [one-line rationale]
⚠ Data quality: [pass | flag from data-qa]
Next: ask the champion for [the missing baseline that would make this High confidence]
```

Keep Quick-mode output to one screen. No table unless the user asks.

---

## Full Model

### 1. Input collection

Collect (or confirm already provided):

| Input | Type | Notes |
|---|---|---|
| Annual revenue (or GMV) | Confirmed / estimated | The denominator for revenue levers |
| Current metric being improved | Confirmed | e.g., churn rate %, conversion rate %, hours/week |
| Team size / affected headcount | Confirmed | For cost levers |
| Contract or license cost for solution | Confirmed | Year 1 and ongoing |
| Time-to-value estimate | Confirmed / assumed | When does the impact start accruing? |
| Discount rate | Assumed default 10% | Adjust for the CFO's preference |

### 2. Model structure

Build one table per McKinsey bucket containing levers that apply. Within each lever:

```
Lever name | Baseline | Impact % | Annual $ impact | Source / confidence | Bucket
```

- **Confirm, then calculate.** Never back-solve from a target ROI. If the prospect says "we need 3x ROI," that is a constraint to check, not an input to reverse-engineer.
- **Attribution discipline.** Only claim the incremental impact attributable to the solution. If churn drops 20% and the solution drives 5 pp of that, model 5 pp. Say so.
- **Conservative anchor.** Default to conservative assumptions; show upside in sensitivity. Champions can defend a conservative number; they can't defend one that got challenged at the table.

### 3. Summary roll-up

```
## ROI model summary — [prospect / brand / date]
Brand: [slug, via brand-brain]
─────────────────────────────────────────────────
Revenue Growth levers:   $[X] / yr
Cost Reduction levers:   $[X] / yr
Capital Efficiency levers: $[X] / yr (or qualitative if not quantifiable)
─────────────────────────────────────────────────
Total annual value:       $[X]
Solution cost (Yr 1):     $[X]
Net annual benefit:       $[X]
Payback period:           [X] months
3-yr NPV (@ [rate]%):     $[X]
ROI (Yr 1):               [X]x
─────────────────────────────────────────────────
Confidence: [High/Medium/Low]
Key assumption to validate: [the one number that moves the model most]
```

### 4. Sensitivity table

Three scenarios across the lever(s) with the highest variance. Conservative / Base / Optimistic. Label what changes in each. Show the ROI and payback period for each.

```
| Scenario      | Key assumption    | Annual value | ROI   | Payback |
|---------------|-------------------|--------------|-------|---------|
| Conservative  | [what's lower]    | $[X]         | [X]x  | [X] mo  |
| Base          | [central estimate]| $[X]         | [X]x  | [X] mo  |
| Optimistic    | [what's higher]   | $[X]         | [X]x  | [X] mo  |
```

Tell the champion: "The conservative scenario is your floor for the CFO conversation. If you can confirm [key input], you can defend the base."

### 5. Champion executive note

One page, brand-voice. Structure:

1. **The problem** (one sentence — the prospect's current cost/risk in dollar or time terms)
2. **The lever** (what changes and why, tied to a confirmed proof point from `brand-brain`/`proof-vault`)
3. **The number** (base-case ROI and payback period, labeled conservative)
4. **What we'd need to confirm** (the one input that would move the model to high confidence)
5. **Recommended next step** (a specific ask, not a generic "let's talk")

Voice and banned words from `brand-brain` apply. Every proof point must be in `proof-vault` or brand.md, or flagged `[verify]`. No invented social proof.

### 6. Save artifacts

Full Model output saves to `./roi/[brand-slug]-[prospect-slug]-roi-model.md`. Quick-mode output is inline only. Never overwrite an existing model without prompting.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No customer-facing copy before brand-brain returns. Its voice and proof override everything here.
- **Data gate before arithmetic.** Run data-qa on pasted inputs; dirty data produces a defensible-looking but wrong number.
- **Confirm, benchmark, or flag.** Every input is labeled. Never pass an assumed number as confirmed.
- **Conservative anchor.** Default to conservative; show upside in sensitivity. A challenged number kills the deal.
- **No double-counting.** If a lever touches two buckets, split it explicitly.
- **Attribution discipline.** Only claim impact attributable to the solution. Don't credit market tailwinds.
- **One model per prospect.** Save with both brand slug and prospect slug so nothing gets overwritten.

## What Not to Do

- Don't back-solve from a target ROI — model forward from confirmed inputs.
- Don't invent customer proof points — use only what brand-brain or proof-vault returns, or `[verify]`.
- Don't call board-exec-summary-writer in the same pass that builds the model — review first.
- Don't skip the sensitivity table to save space — it's the table that survives CFO scrutiny.
- Don't use growth-rate assumptions without labeling their source (analyst report, your own cohort data, assumed).
- Don't combine multiple prospects' metrics in one model — build one model per deal.

## Quality Checklist (self-review before presenting)

- [ ] brand-brain called and returned before any customer-facing copy was written?
- [ ] data-qa run on any pasted input data; flags addressed or disclosed?
- [ ] Every input labeled: confirmed / benchmarked / assumed?
- [ ] Every lever mapped to exactly one McKinsey bucket; no double-counting?
- [ ] Conservative anchor used as the base; optimistic reserved for sensitivity table?
- [ ] Payback period and 3-yr NPV calculated (Full Model)?
- [ ] Sensitivity table covers the highest-variance lever with three scenarios?
- [ ] Champion executive note names one specific next step?
- [ ] All proof points in brand.md / proof-vault, or `[verify]`?
- [ ] Full Model artifact saved to `./roi/[brand-slug]-[prospect-slug]-roi-model.md`?
