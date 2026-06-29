---
name: cro-reporting-audit-packager
description: >
  Takes the raw outputs of a CRO program — heuristic audit notes, experiment results, running test
  inventory, and any analytics-backed friction data — and packages them into two reusable artifacts:
  (1) a formatted CRO scorecard that scores each site section by opportunity tier and test status,
  and (2) a weekly standup digest that gives stakeholders a plain-English read on what's running,
  what shipped, what moved the needle, and what's next. Uses an ICE-backed scoring model
  (Impact, Confidence, Ease), with opportunities sourced from ResearchXL-style conversion
  research, then cross-checked with the PIE framework's Potential/Importance/Ease — plus a
  validity-threat gate to prevent shipping learnings from compromised tests.
  Calls brand-brain for voice/brand context so the digest reads in-brand, not like a generic
  analytics dump. Composes data-qa-measurement-gotcha-checker to gate data quality before any
  conversion rate is treated as ground truth. Calls analytics-report-reviewer to QA the packaged
  scorecard before it goes to stakeholders. Use whenever the user says "package my CRO audit,"
  "CRO scorecard," "CRO standup report," "weekly CRO digest," "format my experiment results,"
  "CRO status for stakeholders," or "what's running / what won this week."
---

# CRO Reporting & Audit Packager

Raw audit notes and a spreadsheet of tests are not stakeholder communication. This skill takes whatever CRO material you have — heuristic findings, experiment results, a running test log — and turns it into two clean artifacts: a scored **CRO Scorecard** stakeholders can act on, and a **Weekly Standup Digest** they'll actually read. The scoring framework is ICE, validity-gated to keep bad-data "wins" out of the record.

This skill **packages and formats**. It does not run new heuristic audits (use `landing-page-heuristic-live-cro-auditor` for that), design new experiments (use `a-b-multivariate-test-designer`), or calculate sample sizes (use `sample-size-calculator`). It composes those siblings' outputs into reporting.

---

## Skills this calls

- **`brand-brain`** (required) — loads active brand voice, ICP, and banned words so the digest stays on-brand.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, voice tone (2–3 adjectives), and any banned words before proceeding.
- **`data-qa-measurement-gotcha-checker`** — gates every conversion rate in the input before it enters the scorecard; flags GA4 sampling, attribution window mismatches, (not set) inflation, and SRM.
- **`analytics-report-reviewer`** — QA pass on the packaged scorecard before delivery; catches unsupported claims and missing context.
- *(optional, when installed)* `validity-threat-checker` for deeper experiment-design review; `experiment-results-analyzer` if raw test data needs statistical interpretation first; `post-test-learning-logger` to archive winning learning cards after packaging.

---

## How a run works

```
Step 0  Load brand context    ──► brand-brain
Step 1  Ingest + triage       ──► accept audit notes, test log, results data
Step 2  Data-quality gate     ──► data-qa-measurement-gotcha-checker on any CVR / metric
Step 3  Score opportunities   ──► ICE score, PIE cross-check
Step 4  Build the Scorecard   ──► tiered opportunity table + test-status grid
Step 5  Build the Digest      ──► plain-English standup narrative
Step 6  QA pass               ──► analytics-report-reviewer
Step 7  Save + deliver        ──► artifacts to ./cro/
```

### Step 0 — Load brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Use the returned digest for voice, banned words, and ICP so both artifacts read like the brand's own language, not boilerplate.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, voice tone (2–3 adjectives), and any banned words before proceeding.

### Step 1 — Ingest

The opportunities you score should come from real conversion research, not gut feel. If the input was gathered ResearchXL-style (CXL / Conversion's six-area conversion-research method — heuristic analysis, technical analysis, web analytics, mouse-tracking/heatmaps, qualitative/user testing, and survey/voice-of-customer), tag each finding with the research area it came from. That tag feeds the Confidence axis later: a finding triangulated across multiple areas scores higher-C than one resting on a single heuristic pass.

Accept any combination of:
- Heuristic audit notes or screenshots (structured or freeform) — *heuristic analysis*
- Experiment results (win/loss/inconclusive + sample sizes, CVRs, confidence levels) — *web analytics*
- Running test inventory (name, hypothesis, variant count, start date, current status)
- Analytics friction data (funnel drop-off %, scroll depth, rage-click coordinates) — *web analytics + mouse-tracking*
- Session replays, heatmaps, on-site survey responses, or VoC quotes — *qualitative / survey*

ResearchXL is the *research* method that surfaces these opportunities; it is not a scoring formula. Scoring happens in Step 3 (ICE) with a PIE cross-check.

Ask for whatever is missing only if the ask is genuinely blocking — most inputs are partial; fill gaps with `[verify]` rather than asking for a perfect input set.

### Step 2 — Data-quality gate

Before any conversion rate or metric enters the scorecard, invoke `data-qa-measurement-gotcha-checker`. Flag and quarantine:
- CVRs from under-sampled segments (GA4 threshold sampling)
- Attribution window mismatches (e.g., 30-day last-click on a 7-day test window)
- SRM (sample ratio mismatch) — run a chi-square goodness-of-fit on the observed variant/control counts against the *intended* split; flag the result INVALID at p < 0.01. Do not use a flat percentage cutoff: at high traffic a sub-1% deviation is already significant SRM, while at low traffic a 5% swing can be ordinary noise. The deviation size is a symptom, not the test. `[verify thresholds against this test's traffic volume]`
- `(not set)` inflation in the relevant dimension — treat any meaningful share as a context-dependent flag, not a fixed cutoff; quarantine the metric and `[verify]` the share against the segment size
- Self-referral loops distorting source/medium

Any metric flagged as unreliable gets a `⚠ data quality` tag in the scorecard and is excluded from the Digest's "what moved the needle" section.

---

## The Scoring Framework (ICE × PIE)

Opportunities are *surfaced* by ResearchXL-style research (Step 1) and *scored* here. The scoring spine is two real prioritization frameworks: **ICE** (Sean Ellis / GrowthHackers) as the primary rank, **PIE** (Chris Goward / WiderFunnel) as the cross-check. ResearchXL is not part of the scoring math.

### ICE — the primary rank

Score every identified opportunity on three axes, 1–10 each:

| Axis | What it measures | Pitfalls to avoid |
|---|---|---|
| **Impact** (I) | Potential CVR uplift × traffic volume on that surface | Don't conflate low-traffic pages with high opportunity |
| **Confidence** (C) | Strength of evidence (quantitative > qualitative > heuristic) | A heuristic alone is low-C; a finding triangulated across 2+ ResearchXL areas is high-C |
| **Ease** (E) | Dev effort inverse (low effort = high score) | Never score "easy" on a test that requires backend changes |

**ICE Score = (I + C + E) / 3** — a starting rank, not the final word.

### Confidence anchors (so C isn't a vibe)

| Evidence behind the finding | Confidence (C) |
|---|---|
| One heuristic observation, no data | 1–3 |
| Single quantitative source (analytics OR heatmap OR survey) | 4–6 |
| Two ResearchXL areas agree (e.g. funnel drop-off + session replay) | 7–8 |
| Three+ areas triangulate, or a prior validated test on the same surface | 9–10 |
| Any source that *failed* the Step 2 data-quality gate | capped at ≤3 |

### PIE — the cross-check

After ICE, recompute the rank with PIE (**Potential, Importance, Ease**, 1–10 each, averaged). PIE's *Importance* axis weights the surface's strategic value and traffic in a way ICE's single Impact axis can blur. Use the divergence as a signal:

| ICE vs PIE rank gap | Read | Action |
|---|---|---|
| ≤1 rank | Frameworks agree | Trust the tier |
| 2 ranks | Mild tension | Note it; tier stands |
| ≥3 ranks | Real disagreement | Flag for stakeholder review — usually high-traffic/low-evidence (high Importance, low Confidence) or the reverse |

A ≥3-rank gap is the single most useful signal this skill produces: it is where a formula would ship the wrong test and a human shouldn't.

### Validity gate

Any experiment result used to calibrate Confidence must pass the Step 2 data-quality check first. An SRM-flagged or sampling-compromised result caps that finding's Confidence at ≤3 and tags it `Invalid` in the scorecard — it cannot promote an opportunity into P1.

---

## Artifact 1 — CRO Scorecard

```
## CRO Scorecard — [Brand] — [Date]

### Data quality flags
[List any ⚠ data quality items here before the scores]

### Opportunity tiers

| # | Page / Surface | Finding source (ResearchXL area) | ICE | PIE Δ | Tier | Test status | Notes |
|---|---|---|---|---|---|---|---|
| 1 | [page] | funnel drop-off + session replay (analytics + mouse-tracking) | 8.2 | ≤1 | P1 | Running (wk 3) | SRM chi-square p=0.42, clear |
| 2 | ...

Tier key: P1 = ship a test this sprint · P2 = next sprint · P3 = backlog · Invalid = data quality fail
PIE Δ = ICE-vs-PIE rank gap; flag any ≥3 for stakeholder review.

### Running tests
| Test name | Hypothesis | Variants | Start | Confidence | Status |
| ...

### Shipped (last 30 days)
| Test name | Result | CVR delta | Action taken |
| ...

### Graveyard (inconclusive / stopped)
| Test name | Why stopped | Learning |
| ...
```

Save to `./cro/cro-scorecard-[brand]-[YYYY-MM-DD].md`.

---

## Artifact 2 — Weekly Standup Digest

Plain English. Senior operators skim this in 90 seconds. No jargon, no raw numbers without a "so what." Structure:

```
## CRO Standup — [Brand] — Week of [date]

**What's running** (n tests)
[1–2 sentences per active test: name, what it's testing, when it started, any early signal]

**What shipped**
[Any tests concluded or changes rolled out this week + result in plain terms]

**What moved the needle**
[Only results that cleared the data-quality gate; CVR delta + estimated revenue/conversion impact if calculable; mark others ⚠]

**Blockers / decisions needed**
[SRM flags, dev backlog, stakeholder approvals outstanding]

**What's next**
[Next 1–2 tests queued, each with one-line hypothesis]
```

Tone: confident, factual, in-brand voice. No hedging on confirmed results; explicit `[verify]` on anything unchecked. No invented impact numbers — if revenue impact is unquantifiable from the data, say so.

Save to `./cro/cro-digest-[brand]-[YYYY-MM-DD].md`.

---

## Step 6 — QA pass

After drafting both artifacts, invoke `analytics-report-reviewer`. It will flag:
- Claims not supported by the input data
- Missing segment context (device split, new vs returning)
- Visualization or formatting issues
- Weak assumptions

Revise before delivering. If the reviewer flags a material claim, either support it or remove it — do not soften and keep.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No artifacts before the brand context loads. Voice and banned-words override everything here.
- **Data quality gates before scoring.** A CVR from a bad sample is not a real CVR. Mark it and exclude it from "what moved."
- **ICE ranks, PIE checks, neither decides.** A ≥3-rank ICE-vs-PIE gap goes to a human, not the backlog. Don't pretend a formula replaces judgment.
- **Validity before learning.** SRM (chi-square p < 0.01 against the intended split), sampling, or attribution issues make a test result inadmissible in the learning record — label it explicitly, cap its Confidence at ≤3.
- **Plain English for stakeholders.** The Digest's job is to inform a non-analyst in 90 seconds, not to showcase methodology.
- **No invented impact.** Revenue/conversion impact only from real data; otherwise omit or `[verify]`.
- **Separate packaging from analysis.** This skill formats and communicates; it calls siblings for statistical interpretation and experiment design.

## What Not to Do

- Don't run a new heuristic audit here — call `landing-page-heuristic-live-cro-auditor` and feed its output in.
- Don't calculate statistical significance from scratch — use `experiment-results-analyzer` or `sample-size-calculator`.
- Don't publish a "win" from a test that fails the SRM chi-square check or any other Step 2 gate — mark it `Invalid` in the scorecard.
- Don't treat ResearchXL as a scoring formula. It surfaces opportunities; ICE and PIE score them.
- Don't use a flat percentage cutoff for SRM. Run the chi-square goodness-of-fit and read the p-value against the test's traffic.
- Don't inflate ICE scores to make a backlog look healthy — low scores are honest signals.
- Don't skip the `analytics-report-reviewer` QA pass before delivering to stakeholders.
- Don't reimplement brand scanning — call `brand-brain`.

## Quality Checklist (self-review before delivering)

- `brand-brain` called; voice + banned-words honored throughout both artifacts?
- `data-qa-measurement-gotcha-checker` run on every CVR / metric in the input; unreliable metrics tagged `⚠ data quality`?
- ICE scores computed (Confidence set off the anchor table); PIE cross-check run; every ≥3-rank gap flagged for stakeholder review?
- SRM checked with a chi-square goodness-of-fit (not a flat % cutoff); failed or invalid tests capped at Confidence ≤3 and excluded from "what moved the needle" in the Digest?
- Scorecard has all four sections (Opportunity tiers, Running, Shipped, Graveyard)?
- Digest is 90-second readable; no raw numbers without a "so what"; no invented revenue impact?
- `analytics-report-reviewer` QA pass completed; findings resolved before delivery?
- Both artifacts saved to `./cro/` with date-stamped filenames?
