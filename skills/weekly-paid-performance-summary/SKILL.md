---
name: weekly-paid-performance-summary
description: >
  Takes raw stats from Google Ads, Meta Ads, and/or LinkedIn Campaign Manager —
  pasted tables, CSV exports, or a mix of screenshots and numbers — and produces a
  plain-English weekly paid performance summary covering total spend, blended ROAS,
  CPA, and per-channel commentary with clear week-over-week and vs-goal deltas. The
  report is diagnostic, not just descriptive: it surfaces the one or two levers most
  worth pulling next week. Integrates with brand-brain for voice and ICP context,
  and calls channel-roi-scorecard for normalized cross-channel math rather than
  reimplementing it. Saves a timestamped markdown report to ./reports/ on request.
  Use when the user says "paid weekly summary," "recap our ad spend," "write the
  paid performance report," "summarize this week's Google/Meta/LinkedIn stats,"
  "what happened in paid this week," or pastes a block of campaign numbers and asks
  for a summary or analysis.
---

# Weekly Paid Performance Summary

Raw numbers in, sharp narrative out. This skill turns a messy paste of Google Ads, Meta, and LinkedIn stats into a one-page weekly report your CMO or board can actually read — with spend, ROAS, CPA, WoW deltas, goal attainment, and clear "here's what to do next week" commentary.

The diagnosis matters more than the description. A real summary names the one campaign that dragged the blended CPA up, not just the average. It flags when a ROAS improvement is a creative win versus a seasonality tailwind. It holds the numbers to the measurement gotchas that corrupt paid reporting — last-click inflation, cross-channel attribution overlap, impression-share blind spots.

---

## Skills this calls

- **`brand-brain`** (required) — loads active brand voice, ICP, offer, and any known paid benchmarks or targets stored in `brand.md`. The summary tone and emphasis align to the brand's positioning.
- **`channel-roi-scorecard`** (required if installed) — handles the normalized cross-channel math: CPA, ROAS, marketing P&L contribution, ranked channel table. Call it rather than recomputing. Synthesize inline only if absent.
- **`data-qa-measurement-gotcha-checker`** — call when the user's data looks suspect or contains raw GA4 exports; it surfaces model-blending, self-referral, and view-through inflation issues before they corrupt the narrative.
- **`bid-budget-pacing-checker`** — call when the user wants pacing status or asks "are we on track with budget?" alongside the weekly summary.
- **`ltv-cac-payback-calculator`** — call when the summary needs to comment on payback or unit economics, not just CPA.
- **`ad-copy-variant-generator`** — call when the summary surfaces a creative fatigue flag and the user wants replacement copy in the same session.

---

## How a run works

```
Step 0  Load the brand       ──► brand-brain (voice, ICP, known targets/benchmarks)
Step 1  Accept + validate    ──► parse the raw data; flag gaps before computing
Step 2  Normalize & compute  ──► channel-roi-scorecard (or inline if absent)
Step 3  Apply the gotcha filter ──► measurement red flags before writing a word
Step 4  Write the narrative  ──► the 3-section report (see framework below)
Step 5  Offer to save        ──► ./reports/paid-weekly-[YYYY-MM-DD].md
```

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Retrieve: voice adjectives, banned words, ICP, and any paid benchmarks (target CPA, target ROAS, channel-level spend allocations) stored in `brand.md`. If the user tracks targets in a separate doc, ask for them once for this run; to persist them, hand them to `brand-brain` (it owns the `brand.md` write — e.g. `brand-brain refresh`). Never write `brand.md` yourself.

Fallback: read `~/.brandbrain/brands/.active` → `brand.md` directly; if absent, ask the user to install `brand-brain` or provide targets + voice inline.

### Step 1 — Accept and validate inputs

Accept data in any form: pasted table, CSV, screenshot description, or a numbered list of stats. Before computing anything, confirm you have — or flag the absence of:

| Field | Required | Notes |
|---|---|---|
| Spend per channel | Yes | Must be same currency |
| Impressions or clicks | Yes | One or both |
| Conversions (type named) | Yes | "Purchase," "trial," "lead" — name it explicitly |
| Revenue or conversion value | For ROAS | Ask if not provided |
| Prior week comparables | Strongly preferred | Without WoW delta the report is weaker; flag it |
| Channel-specific goals/targets | Preferred | Pull from brand.md if present |

If a field is missing and consequential, ask one consolidated question — not five separate ones.

### Step 2 — Normalize and compute

Call `channel-roi-scorecard` if installed; pass it the cleaned per-channel data. If absent, compute inline:

- **CPA** = spend ÷ conversions (per channel + blended)
- **ROAS** = conversion value ÷ spend (per channel + blended; mark as `[no revenue data — omit ROAS]` if value absent)
- **CPM / CPC** = spend ÷ (impressions/1000) or clicks
- **CTR** = clicks ÷ impressions
- **WoW delta** = (this week − last week) ÷ last week × 100; flag direction + magnitude
- **vs-goal** = (actual − target) ÷ target × 100; green/amber/red status

Produce a normalized channel comparison table (see format below) before writing prose.

### Step 3 — Apply the gotcha filter (before writing)

Run the **Paid Reporting Gotcha Checklist** mentally before committing to any claim in the narrative:

| Gotcha | Check |
|---|---|
| **Platform-reported vs. GA4 conversions** | Platform usually over-counts (view-through, click + view duplication). Note the gap if both data sets present; never blend without flagging |
| **Attribution window mismatch** | Google default is 30-day click / 1-day view; Meta can be 7-day click / 1-day view; LinkedIn 30-day. Cross-channel blended CPA is only comparable when windows are matched |
| **Cross-channel overlap** | A buyer who clicked a Google ad and a Meta ad will appear in both platforms' conversion counts. Blended ROAS will be inflated vs. true incrementality |
| **Conversion type drift** | If the conversion event changed mid-week (e.g., from "Add to Cart" to "Purchase"), WoW delta is meaningless — flag it |
| **Budget pacing distortion** | A campaign that exhausted budget mid-week looks artificially efficient (high ROAS on fewer impressions). Check impression share / budget-limited status |
| **Brand vs. non-brand split** | Brand search campaigns almost always have better ROAS and CPA. Blended numbers that mix brand and non-brand mislead on true acquisition efficiency — break them out if data allows |
| **Seasonality / external event** | Holidays, product launches, competitor activity can explain deltas that look like optimizations. Note if known |

Flag any live gotcha explicitly in the report. Do not suppress a real measurement concern to make the summary look cleaner.

---

## The Report Format (3 sections)

### Section 1 — Scorecard table

```
## Week of [Mon DD] – [Sun DD, YYYY]

| Channel     | Spend   | Conversions | CPA    | ROAS  | WoW Δ CPA | vs Goal |
|-------------|---------|-------------|--------|-------|-----------|---------|
| Google Ads  | $X,XXX  | XXX         | $XX.XX | X.Xx  | −12%      | ✓ +8%   |
| Meta        | $X,XXX  | XXX         | $XX.XX | X.Xx  | +4%       | ✗ −11%  |
| LinkedIn    | $X,XXX  | XXX         | $XX.XX | —     | —         | —       |
| **Blended** | **$X,XXX** | **XXX**  | **$XX.XX** | **X.Xx** | **−5%** | **✓ +3%** |
```

Omit ROAS column if revenue data is unavailable; note why.

### Section 2 — Channel-level commentary (3–5 sentences per active channel)

Per channel: what the number says, what likely drove it (creative, bid change, audience, seasonality), any measurement flag, and one concrete action for next week. Do not just describe the table — diagnose it.

Structure each channel block as:
- **Headline verdict** (one sentence, e.g., "Google non-brand efficiency improved but impression share dropped — gains are fragile.")
- **What drove it** (specific, not generic — name the campaign or creative if the data allows)
- **Measurement note** (flag any gotcha from Step 3 that applies to this channel's data)
- **Next-week action** (one specific thing to test, pause, scale, or investigate)

### Section 3 — Top takeaway + single highest-leverage action

One paragraph (4–6 sentences) that answers: *if you could only do one thing differently next week, what is it and why?* Ground it in the numbers. Name the campaign, channel, or creative. Do not list five things — pick one.

If the data is too thin to name a confident recommendation, say so rather than inventing one.

---

## Measurement principles (non-negotiable)

- **Platform-reported numbers are claimed, not verified.** Always note this and, where possible, cite the GA4 or third-party-attributed number alongside.
- **Blended ROAS and blended CPA are directional.** Cross-channel overlap and mismatched attribution windows mean they are not additive. Mark them as blended and note the overlap caveat.
- **WoW deltas need a stable denominator.** If spend changed materially WoW, comment on it — efficiency metrics will move even if nothing else changed.
- **No numbers invented or extrapolated.** Missing fields get flagged, not estimated. Guessed proof is `[verify]`.
- **Don't suppress measurement concerns.** A report that hides a conversion-type drift or a budget-pacing distortion is worse than a report with a caveat.

---

## Principles

- **Diagnose, don't just describe.** The table describes. The commentary diagnoses. The takeaway prescribes. All three layers are required.
- **Brand-brain first.** No summary until voice and known targets are loaded. The report tone and framing match the brand's communication style.
- **One lever per takeaway.** A "top 5 recommendations" list is a non-answer. Identify and defend one.
- **Name things.** "One campaign" is vague; "Google Ads — Non-Brand Prospecting" is actionable.
- **Gotcha-first.** Run the measurement checklist before drafting prose. Clean data or flagged data — never silent bad data.

## What Not to Do

- Don't blend platform-reported and GA4 conversions without an explicit note that you're doing it.
- Don't produce a blended ROAS number without flagging cross-channel overlap.
- Don't write "performance improved" without naming what metric, by how much, and on which channel.
- Don't invent a recommendation when the data is too thin — flag the data gap instead.
- Don't reimplement the CPA/ROAS math if `channel-roi-scorecard` is installed — call it.
- Don't produce the report before `brand-brain` returns the active brand context.
- Don't equate brand-search efficiency with true acquisition efficiency — break them out.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and brand digest + known targets loaded?
- All input channels present? Missing fields flagged before computing?
- `channel-roi-scorecard` called (or inline math confirmed correct)?
- Gotcha checklist run; any live flags documented in the report?
- Scorecard table complete with WoW Δ and vs-goal columns (or absence explained)?
- Each channel block has: headline verdict + driver + measurement note + next-week action?
- Top takeaway names one lever, grounded in the numbers, not a generic list?
- No invented numbers; unconfirmed proof marked `[verify]`?
- Offer to save to `./reports/paid-weekly-[YYYY-MM-DD].md` included at the end?
