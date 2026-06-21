---
name: segmented-conversion-rate-breakdown
description: >
  Takes raw conversion data — GA4 exports, ESP reports, CRM exports, or pasted tables — with at least
  one segment dimension (traffic source/medium, device category, geography, new vs returning, campaign,
  landing page, browser, screen size, day-of-week) and returns a pivot-style breakdown ranking each
  segment by conversion rate, absolute volume, and weighted contribution to total conversions. Surfaces
  your highest-leverage segments for CRO investment and your worst-performing for either fix-or-suppress
  decisions. Applies the classic GA4/analytics validity gates (SRM, sampling, attribution window gaps,
  (not set) carve-out, self-referral pollution) before drawing any conclusion — so you never optimize on
  noise. Composes `data-qa-measurement-gotcha-checker` for the data-quality pass, `validity-threat-checker`
  for experiment-design threats, and `analytics-report-reviewer` to review the finished breakdown before
  delivery. Calls `a-b-multivariate-test-designer` when a test hypothesis is identified.
  Use when the user says "break down conversion rates by segment," "which segments convert best/worst,"
  "why is my overall CVR so low," "segment my funnel," "show me CVR by source / device / geo / new vs
  returning," "pivot my conversion data," "where should I focus my CRO effort," or pastes a data table
  and asks "what should I fix first."
---

# Segmented Conversion Rate Breakdown

Your overall conversion rate is an average — and averages hide everything. A 2.4% blended CVR might mean mobile traffic converting at 0.8% while desktop converts at 5.1%, or that one source drives 60% of sessions but 10% of revenue. This skill slices the data, ranks the segments, and tells you exactly where to win and what to suppress.

Framework: **Segment Contribution Analysis** — rank every segment by CVR, index against the site average, weight by session share, and flag high-volume/low-CVR segments as the primary optimization lever. Every conclusion passes a data-quality gate first; no action recommendation ships on corrupted or sparse data.

---

## Skills this calls

- **`brand-brain`** (Step 0, required) — loads the active brand's ICP, known conversion goals, key URLs, and offer mechanics so segment analysis is anchored to the right funnel and goal. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their primary conversion goal, goal URL or event name in GA4, and the date range of the data before proceeding.
- **`data-qa-measurement-gotcha-checker`** (Step 1, required) — runs the data-quality gate before any math. Do not skip; bad segments corrupt the ranking.
- **`validity-threat-checker`** (Step 3, conditional) — when the breakdown reveals a clear test hypothesis (fix mobile checkout, suppress a low-intent source), invoke to pressure-test the experiment design for SRM, novelty effect, and seasonality.
- **`analytics-report-reviewer`** (Step 5, required) — reviews the finished breakdown for unsupported claims and weak assumptions before delivery.
- **`a-b-multivariate-test-designer`** (Step 3, on request or when a strong hypothesis emerges) — converts a high-confidence segment finding into a structured test design.

---

## How a run works

```
Step 0  Load brand context         ──► brand-brain (funnel, goals, ICP)
Step 1  Data-quality gate          ──► data-qa-measurement-gotcha-checker
Step 2  Build the pivot            ──► segment × CVR × volume × contribution
Step 3  Rank & diagnose            ──► best/worst segments + hypothesis
Step 4  Compose test design (opt)  ──► a-b-multivariate-test-designer / validity-threat-checker
Step 5  Reviewer pass              ──► analytics-report-reviewer
Step 6  Deliver the breakdown      ──► ranked table + action brief
```

---

## Step 0 — Load brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Capture: primary conversion goal and event name or URL, ICP traffic profile (device/channel mix expectations), any known funnel stages (trial signup, checkout, demo request), and offer mechanics that influence segment behavior (free vs paid, self-serve vs assisted).

**Fallback if brand-brain is absent or returns no brand:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for (1) the primary conversion goal/event, (2) the GA4 property or export source, and (3) the reporting date range before proceeding.

---

## Step 1 — Data-quality gate

Invoke `data-qa-measurement-gotcha-checker` on the raw data. Specifically enforce:

**GA4-specific gotchas (always check):**
- **Sampling** — GA4 free tier samples at 10M+ event explorations; flag if the export looks aggregated.
- **(not set) carve-out** — `(not set)` in source/medium or device is not a segment; exclude it from CVR ranking; flag its session share separately.
- **Self-referral pollution** — known payment-gateway or subdomain referrals inflate a "referral" segment; flag and advise the exclusion fix.
- **Attribution window mismatch** — if the date range is shorter than the typical conversion window (e.g., 30-day email nurture evaluated over 7 days), results are structurally incomplete; flag and suggest expanding the window.
- **Goal configuration** — if the conversion event has a 0% CVR across all segments equally, the event is likely misconfigured, not a segment problem; surface as a blocking issue.
- **Cross-device fragmentation** — new-vs-returning splits in GA4 are device-scoped, not user-scoped; note this when interpreting that dimension.
- **SRM (Sample Ratio Mismatch)** — when comparing segments that map to an active experiment, check that traffic splits match the intended allocation ratio.

If data quality is insufficient to draw conclusions, stop and return a findings-only report with remediation steps. Do not produce a ranked table on corrupt data.

---

## Step 2 — Build the pivot

For each segment dimension provided, compute:

| Column | Definition |
|---|---|
| Segment | Dimension value (e.g., "organic / google", "mobile", "US") |
| Sessions | Raw session (or user) count |
| Conversions | Goal completions for the defined conversion event |
| CVR % | Conversions / Sessions × 100 |
| CVR Index | Segment CVR / Site-average CVR × 100 (100 = average) |
| Session Share % | Segment sessions / Total sessions × 100 |
| Conversion Share % | Segment conversions / Total conversions × 100 |
| CVR × Volume Flag | Tag: HIGH-LEVERAGE (high sessions + low CVR), STAR (high sessions + high CVR), NICHE (low sessions + high CVR), DRAG (low sessions + low CVR) |

Sort by Conversion Share descending for the primary view; offer a CVR-sorted view on request.

When multiple segment dimensions are provided, run the pivot for each independently. Do not cross-segment (e.g., mobile × organic) unless the user requests it and the data volume supports it (minimum 200 sessions per cell before drawing CVR conclusions).

---

## Step 3 — Rank and diagnose

**Best segments (STAR + NICHE):** confirm they align with the brand's ICP. If a high-CVR segment is small, diagnose whether it's small because of audience size or because it's under-invested in (media budget, SEO, messaging).

**Worst segments with HIGH-LEVERAGE flag:** these are the primary CRO targets — high traffic, low conversion. For each:
1. Name the most plausible hypothesis (landing page mismatch, device friction, intent mismatch, offer misalignment, attribution gap).
2. Rate hypothesis confidence: HIGH (data pattern is unambiguous) / MEDIUM (plausible but needs investigation) / LOW (insufficient data, needs instrumentation first).
3. Propose the next action: run a test, instrument better, suppress traffic, or fix a known technical issue.

**Segment minimum threshold:** call out any segment with fewer than 100 sessions as directional only — not actionable without more data.

---

## Step 4 — Test design (conditional)

When a HIGH-confidence hypothesis emerges (HIGH-LEVERAGE segment with a clear fix), offer to invoke `a-b-multivariate-test-designer` to produce a structured test design. Invoke `validity-threat-checker` on any proposed test design to check for SRM risk, novelty effects, or seasonal confounds before committing to a test plan.

If the user declines, provide a one-paragraph test brief they can action manually.

---

## Step 5 — Reviewer pass

Invoke `analytics-report-reviewer` on the completed breakdown and action brief. Pass: the pivot table, the ranked diagnoses, any test hypotheses, and the data-quality findings from Step 1. Revise based on its output before delivery.

---

## Step 6 — Deliver the breakdown

Present in this structure:

```
## Segmented Conversion Rate Breakdown — [brand / goal / date range]

### Data quality summary
[Pass/flag items from data-qa-measurement-gotcha-checker; any blocking issues]

### Pivot: CVR by [dimension]
[Ranked table per Step 2]

### Top findings
1. [STAR segment] — [CVR], [conversion share], [implication]
2. [HIGH-LEVERAGE segment] — [sessions], [CVR index], [hypothesis, confidence]
3. [additional segments as relevant]

### Action brief
| Priority | Segment | Finding | Recommended action | Confidence |
|---|---|---|---|---|

### Test hypothesis (if applicable)
[One-paragraph brief or link to a-b-multivariate-test-designer output]
```

Save to `./cro/[brand-slug]-cvr-breakdown-[YYYY-MM-DD].md` when the user asks for a saved artifact or when the output is more than one segment dimension.

---

## Principles

- **Data quality gates before conclusions.** A corrupted segment ranking wastes optimization budget. The gotcha-checker pass is mandatory, not optional.
- **CVR alone doesn't rank segments.** A 20% CVR on 50 sessions is not your priority; a 0.8% CVR on 40,000 sessions is. Always weight by volume.
- **Attribution windows determine what you're measuring.** Name them explicitly; never compare segments across mismatched windows.
- **(not set) is not a segment.** Exclude it from rankings; treat it as a data health issue to fix upstream.
- **Hypotheses must be falsifiable.** "Mobile converts worse" is an observation. "Mobile checkout drop-off at the address field is causing 60% abandonment" is a testable hypothesis.
- **Minimum 100 sessions per cell.** Below that, report directionally only; do not rank or prioritize.
- **One dimension at a time by default.** Cross-segmentation multiplies sparsity; only do it when volume supports and the user requests it.

---

## What not to do

- Don't skip the data-quality gate and build a pivot on (not set)-heavy or sampled data.
- Don't rank a segment with fewer than 100 sessions alongside high-volume segments as though they're comparable.
- Don't treat CVR index alone as the optimization priority — weight always beats rate when session share is large.
- Don't propose a test before running `validity-threat-checker` — SRM and novelty effects regularly invalidate mobile/device-split experiments.
- Don't reimplement brand resolution — call `brand-brain`; don't ask the user to re-explain their funnel if the brand already has an active brain.
- Don't conflate GA4's device-scoped new/returning with true user-level new/returning behavior; note the distinction when that dimension is included.
- Don't cross-segment (e.g., source × device) unless asked and cells are sufficient — sparse intersections produce misleading CVRs.

---

## Quality checklist (self-review before delivery)

- `brand-brain` called; conversion goal, funnel stage, and ICP anchored before building the pivot?
- `data-qa-measurement-gotcha-checker` run; (not set) excluded; self-referral, sampling, and attribution window issues flagged?
- Each segment has: CVR, CVR index, session share, conversion share, and a HIGH-LEVERAGE / STAR / NICHE / DRAG tag?
- Segments with fewer than 100 sessions marked directional only?
- At least one HIGH-LEVERAGE segment diagnosed with a hypothesis and confidence rating?
- `validity-threat-checker` invoked on any proposed test design?
- `analytics-report-reviewer` pass complete; unsupported claims removed?
- Saved artifact written to `./cro/` when output spans multiple dimensions or user requested it?
