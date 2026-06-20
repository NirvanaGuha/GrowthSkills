---
name: channel-roi-scorecard
description: >
  Spend by channel + revenue/conversion data → cost-per-acquisition, ROAS, marketing P&L
  contribution, and ranked channel table. Takes raw spend and performance inputs (from
  spreadsheet paste, ad-platform exports, or typed numbers) and produces a standardized
  scorecard: CPA, ROAS, revenue contribution %, marginal efficiency, payback context, and
  a ranked performance table with a clear investment recommendation. Accounts for attribution
  windows, last-touch vs. modeled credit, and common data-quality traps before drawing
  conclusions. Calls brand-brain for ICP + offer context so cost benchmarks and verdict copy
  are grounded in the brand's actual economics, not generic industry averages. Use when the
  user says "which channels are working," "where should I cut or double down," "show me ROAS
  by channel," "channel performance report," "marketing P&L," "cost per acquisition by source,"
  "channel mix scorecard," or pastes a spend/revenue table and asks what it means.
---

# Channel ROI Scorecard

Spend data without a ranked verdict is just a spreadsheet. This skill takes your channel inputs and turns them into a decision-ready scorecard: who earned the next dollar, who should get cut, and what the gaps in the data mean before you act on any of it.

Built on the **ROAS → CPA → Marginal-Efficiency → Payback** stack. Every number flows through a data-quality gate first, because the most common channel-ROI mistake is optimizing off broken attribution.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's offer economics (price, margin, trial/subscription model, average order value), ICP, and positioning. CPA benchmarks and verdict copy are anchored to the brand's actual unit economics, not generic industry averages.
- **`ltv-cac-payback-calculator`** (call when LTV or payback period is needed) — if the brand has cohort data, delegate LTV/CAC/payback math rather than re-deriving it here.
- **`data-qa-measurement-gotcha-checker`** (call when the user pastes a GA4 report) — run data quality flags before interpreting any GA4-sourced figures.
- **`experiment-results-analyzer`** (call if the user wants to validate whether a channel's uplift is statistically significant, not just arithmetically higher).

---

## How a run works

```
Step 0  Load brand context     ──► call brand-brain (ICP, offer mechanics, margin/AOV)
Step 1  Data-quality gate       ──► flag attribution issues before computing anything
Step 2  Compute the scorecard   ──► CPA, ROAS, revenue %, marginal efficiency per channel
Step 3  Rank + verdict          ──► ranked table + investment recommendation
Step 4  Narrative summary       ──► plain-English P&L story for stakeholders
Step 5  Save (optional)         ──► offer to write to ./reports/channel-roi-[date].md
```

---

## Step 0 — Load brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`) before any calculation. The returned digest supplies:

- **AOV / ACV / MRR per plan** — the revenue denominator for ROAS
- **Gross margin %** — required for marketing P&L contribution; don't use revenue alone
- **Trial/freemium mechanics** — signals whether CPA should be measured at trial-start, paid-conversion, or a blended activation gate
- **ICP segment** — calibrates what a "good" CPA looks like against typical LTV for this customer

If `brand-brain` is absent, ask the user for: (1) average revenue per acquired customer, (2) gross margin %, (3) whether you're measuring at trial or paid conversion. Don't guess margin.

---

## Step 1 — Data-quality gate (run before computing)

Measurement errors compound in channel scorecards. Flag these before drawing conclusions:

| Gotcha | How to detect | How to handle |
|---|---|---|
| Attribution window mismatch | Paid channels default 7-day click / 1-day view; GA4 defaults last-click 30d | Flag if comparing Google Ads ROAS to GA4 revenue — windows differ. Ask for export-level data or reconcile to one window |
| Last-touch over-credits direct & branded search | Direct/organic capture intent built by other channels | Note it. Recommend view-through or data-driven attribution if available |
| Self-referral / spam hostnames (GA4) | Sessions from payment gateways, internal tools, or spam TLDs | Flag if present; revenue attributed to these is inflated |
| GA4 broken page_view or session inflation | Check if sessions >> users ratio is implausible (>5:1 in a session) | Flag; note which channels look inflated |
| Spend ≠ revenue period alignment | Spend pulls by billing date; conversions pull by event date | Ask the user to confirm periods match |
| Blended vs. incremental ROAS | Brand-search ROAS looks high because of organic intent steal | Flag brand-search as a separate line item if it's mixed in |
| Sample-ratio mismatch (if comparing test vs. control) | Variant received meaningfully different traffic share than planned | Flag; call `experiment-results-analyzer` for significance |

If any flag fires, state it before the scorecard table. Don't hide data issues in footnotes.

---

## Step 2 — Compute the scorecard

For each channel, calculate:

| Metric | Formula | Notes |
|---|---|---|
| **CPA** | Spend ÷ Conversions | Use the conversion event the brand cares about (trial, paid, MQL) |
| **ROAS** | Revenue ÷ Spend | Use gross revenue, note if margin-adjusted below |
| **Margin-adjusted ROAS (mROAS)** | (Revenue × Gross Margin %) ÷ Spend | This is the real profitability signal; pure ROAS flatters high-volume/low-margin channels |
| **Revenue contribution %** | Channel Revenue ÷ Total Attributed Revenue | Shows mix, not efficiency |
| **Marginal CPA / efficiency trend** | (CPA this period) ÷ (CPA prior period) | Is the channel getting more or less efficient as spend scales? Flat ROAS + rising CPA = scaling into diminishing returns |
| **Payback period (days)** | CPA ÷ (Monthly Revenue per Customer × Gross Margin %) | Context-only; delegate to `ltv-cac-payback-calculator` for full cohort view |

Mark any number `[verify]` if the user hasn't supplied the underlying data. Do not impute margin or AOV from industry benchmarks without flagging it.

---

## Step 3 — Ranked channel table + verdict

```
## Channel ROI Scorecard — [Brand] — [Period]
Attribution note: [window / model / flags from Step 1]

| Rank | Channel      | Spend | Revenue | ROAS | mROAS | CPA  | Rev % | Trend | Verdict  |
|------|--------------|-------|---------|------|-------|------|-------|-------|----------|
| 1    | Paid Search  | $X    | $Y      | X.Xx | X.Xx  | $Z   | XX%   | ↑ eff | Scale    |
| 2    | Email        | $X    | $Y      | X.Xx | X.Xx  | $Z   | XX%   | →     | Hold     |
| 3    | Paid Social  | $X    | $Y      | X.Xx | X.Xx  | $Z   | XX%   | ↓ eff | Cut/Test |
| —    | Organic SEO  | $0    | $Y      | ∞    | ∞     | $0   | XX%   | —     | Invest   |
| —    | Direct       | $0    | $Y      | ∞    | ∞     | $0   | XX%   | —     | [note last-touch bias] |
Total | All          | $X    | $Y      | X.Xx | X.Xx  | $Z   | 100%  |       |          |
```

**Verdict tags:** Scale (mROAS above brand's breakeven threshold, efficiency stable or improving) · Hold (at or near breakeven, efficiency flat) · Cut/Test (mROAS below breakeven or efficiency declining) · Invest (non-paid; attribute ROI via incrementality test if actionable) · Data gap (insufficient period, broken tracking, or missing data).

**Breakeven mROAS** = 1.0 (spend equals gross profit generated). Anything below 1.0 is contribution-negative.

---

## Step 4 — Narrative summary

Write 3–5 sentences in the brand's voice (per `brand-brain`):

1. **Lead with the headline verdict** — which channel is the engine, which is the drag.
2. **The one non-obvious finding** (e.g., email is under-resourced relative to its mROAS; social ROAS looks fine but efficiency is declining at current spend levels).
3. **The data-quality caveat** if Step 1 flagged anything — state what it means for how much to trust the conclusions.
4. **The recommended next action** — a concrete reallocation, a tracking fix to implement before next review, or a test to run.

Keep this to 5 sentences maximum. This section pastes directly into a stakeholder Slack or deck.

---

## Principles

- **mROAS over ROAS.** Gross margin is the business's real currency. A channel with 4x ROAS and 20% margin is contribution-negative at the same dollar as a channel with 2.5x ROAS and 60% margin.
- **Attribution is a hypothesis, not a fact.** State the model and window explicitly. Never present a CPA or ROAS without its attribution context.
- **Efficiency trend matters more than absolute rank.** A channel at 2x mROAS and deteriorating is riskier than one at 1.5x mROAS and improving.
- **Brand-brain first.** Without the brand's actual unit economics (AOV, margin, conversion gate), every benchmark is invented. Get them before you score anything.
- **Flag gaps, don't fill them.** Missing data → `[verify]` or "Data gap" in the verdict column. Don't extrapolate from industry averages without labeling it.
- **Payback context belongs here; LTV math belongs in `ltv-cac-payback-calculator`.** Don't re-derive cohort curves — call the skill.

---

## What not to do

- Don't rank channels on revenue contribution % alone — that's volume, not efficiency.
- Don't compare ROAS across channels if they use different attribution windows (Google Ads vs. GA4 default).
- Don't present organic/direct alongside paid on the same mROAS scale without noting last-touch attribution bias.
- Don't invent margin or AOV figures; ask the user or flag `[verify]`.
- Don't produce a verdict if Step 1 flags a broken-tracking issue severe enough to make the data unreliable — fix the measurement first.
- Don't use "blended ROAS" (all spend ÷ all revenue) as a channel-level signal. It's a portfolio average, not a decision tool.

---

## Quality checklist

- `brand-brain` called; AOV, gross margin %, and conversion gate confirmed (or `[verify]` flagged)?
- Attribution window/model stated for every channel row?
- Step 1 data-quality flags surfaced before the scorecard table?
- mROAS calculated (not just ROAS); breakeven threshold (1.0) noted?
- Efficiency trend (scaling behavior) assessed, not just current-period snapshot?
- Verdict column uses Scale / Hold / Cut-Test / Invest / Data-gap — no ambiguous language?
- Narrative summary is ≤5 sentences, leads with the headline verdict, states the data caveat?
- Save offer made; output path is `./reports/`, never the skill folder or brand.md?
