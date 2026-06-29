---
name: ga4-anomaly-detector
description: >
  Takes a GA4 property, one or more metrics, and a lookback window and produces a plain-English
  alert report: which metrics deviated from expected, by how much, in which direction, and what
  the most likely causes are — with a prioritized action list. Flags data quality issues first
  (tracking breakage, sampling, (not set) spikes, attribution shifts, bot/spam) before leaping
  to marketing conclusions. Two modes: Scan (run across all core metrics to surface anything
  anomalous) and Focused (investigate a specific metric the user already suspects). Composes
  `data-qa-measurement-gotcha-checker` as a mandatory data-quality gate so instrumentation
  failures are never misread as real performance changes. Composes `analytics-report-reviewer`
  for a final narrative QA pass. Outputs a structured anomaly brief ready to paste into a Slack
  ping, stakeholder update, or board pack. Use when the user says "something looks off in GA4,"
  "sessions dropped," "conversions spiked," "traffic anomaly," "explain this metric change,"
  "run a GA4 anomaly check," "what happened to my [metric]," or pastes a screenshot or data
  export with an unusual number.
---

# GA4 Anomaly Detector

Something moved. This skill tells you whether it's real, why it happened, and what to do — in that order.

Most "traffic dropped" panics are measurement problems, not marketing ones. This skill runs a data-quality gate before reasoning about causes, so you don't rewrite your homepage because a tag broke. It applies an independent statistical anomaly-detection method (Z-score / IQR over a rolling baseline, adjusted for weekly seasonality) and encodes the classic GA4 measurement gotchas as first-pass checks — attribution windows, (not set) swells, sampling thresholds, self-referral, SRM in active experiments, and hostname pollution.

It does not query GA4 directly; it works from data you provide or pull via `~/ga4-query/` scripts. It reasons like a senior analyst, not a dashboard.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's context: GA4 property ID, known measurement quirks, ICP, and any brand-specific anomaly thresholds. Does not produce output before brand-brain returns.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for the GA4 property ID, the metric(s) in scope, the comparison period, and any known instrumentation issues before proceeding.

- **`data-qa-measurement-gotcha-checker`** (required, Step 2) — mandatory data-quality gate. Always invoked before interpreting metric changes as real performance signals. Gates on: tracking code status, sampling flags, (not set) dimension share, self-referral contamination, hostname spam, attribution model and lookback window changes, active experiment SRM.

- **`analytics-report-reviewer`** (Step 4, final QA) — reviews the draft anomaly brief for unsupported causal claims, missing context, and weak assumptions before presenting.

- **`weekly-wins-anomalies-slack-ping`** *(optional, if installed)* — formats the alert as a Slack message when the user needs it channel-ready.

- **`kpi-tree-builder`** *(optional, if installed)* — used to trace a top-line anomaly downstream to its component metrics.

- **`ga4-custom-report-exploration-builder`** *(optional, if installed)* — builds the follow-on Exploration spec the user needs to dig deeper.

---

## How a run works

```
Step 0  Load brand context   ──► brand-brain (GA4 property ID, known quirks, thresholds)
Step 1  Ingest & scope       ──► metric(s), date range, comparison baseline
Step 2  Data-quality gate    ──► data-qa-measurement-gotcha-checker (blocking)
Step 3  Anomaly scoring      ──► baseline → deviation → Z-score / IQR → severity
Step 4  Causal triage        ──► instrumentation → external → internal → channel-level
Step 5  Draft alert brief    ──► structured output: what / how much / why / next action
Step 6  Narrative QA         ──► analytics-report-reviewer (removes unsupported claims)
Step 7  Present              ──► alert brief + optional Slack format
```

### Step 0 — Load brand context (always first)

Invoke the `brand-brain` skill. It returns the active brand digest including GA4 property ID, any documented measurement quirks (e.g., spam hostnames to filter, known (not set) issues, attribution model in use), and brand-specific alert thresholds if set.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for the GA4 property ID, the metric(s) in scope, the comparison period, and any known instrumentation issues before proceeding.

### Step 1 — Ingest and scope

Accept any of:
- Raw metric values pasted inline (e.g., "sessions: 4,200 this week vs 6,800 last week")
- A CSV or table export from GA4 / Looker Studio
- A screenshot (extract visible numbers; note what's not visible)
- A description ("conversions are down ~40% this week")

Confirm: metric name, date range of the anomaly period, comparison baseline (same period prior week, prior 4-week average, YoY), and any filters applied (hostname, segment, channel). If the comparison baseline is unclear, default to a 4-week rolling average of the same weekday pattern — weekend vs weekday seasonality is a top source of false alarms.

**Scope the mode:**
- **Scan mode** — no specific metric named → check all core metrics (sessions, users, engaged sessions, engagement rate, key events/conversions, revenue if eCommerce) and surface anything beyond the anomaly threshold.
- **Focused mode** — specific metric named → go deep on that metric and its component dimensions.

### Step 2 — Data-quality gate (blocking)

**Always invoke `data-qa-measurement-gotcha-checker` before reasoning about causes.** Pass it the metric, the data, and any context from brand-brain. It checks:

1. **Tracking integrity** — GA4 tag firing rate, page_view event completeness, missing data-stream days
2. **Sampling** — GA4 standard properties sample at 10M sessions/day; Analytics 360 raises this. Note if data is sampled.
3. **(not set) share** — sudden spike in `(not set)` for channel, source/medium, or landing page = broken referral or UTM stripping
4. **Self-referral** — payment gateway, subdomain, or OAuth redirect not in referral exclusion list → inflates sessions, deflates conversion rate
5. **Hostname pollution** — GA4 property receiving hits from dev/staging/test hostnames, scrapers, or phantom referrers
6. **Attribution model / lookback window change** — recent change in GA4 attribution settings shifts credit between channels
7. **Active experiment SRM** — if an A/B test is running, check for Sample Ratio Mismatch before attributing changes to it
8. **UTM drift** — check if UTM coverage changed (campaigns going untagged = session fragmentation into direct/organic)

If any gate fails: **surface it as the primary finding** and label the metric reading `[UNRELIABLE — instrumentation]` before proceeding. Do not speculate about marketing causes for an instrumentation failure.

### Step 3 — Anomaly scoring

For each metric that clears the data-quality gate:

**Baseline:** compute expected value from the comparison window. Default: 4-week rolling average, same weekday pattern. If the user supplies a prior-year comparison, use it (seasonality-adjusted baseline is stronger than WoW for metrics with weekly cycles).

**Deviation:** `(actual − expected) / expected × 100` → percentage delta.

**Severity tiers:**

| Tier | Threshold | Label |
|---|---|---|
| Watch | 10–20% delta | Low |
| Alert | 21–40% delta | Medium |
| Critical | >40% delta or >2 SD from rolling baseline | High |

Z-score method: `Z = (actual − μ) / σ` where μ and σ are the rolling baseline mean and standard deviation. Z > 2 = High anomaly. (Mark `[verify σ]` if you're estimating variance from fewer than 4 data points.)

### Step 4 — Causal triage (structured, not a guess)

Work through causes in this order. Stop at the first confirmed cause; don't stack speculative ones.

**Tier A — Instrumentation (most common, check first)**
- Tag firing failure (site deploy, CMP blocking, cookie consent drop)
- New redirect or subdomain not in referral exclusion list
- GA4 property misconfiguration (data filter change, event modification)

**Tier B — External / market**
- Algorithm update (check GSC for impression/click delta on same dates)
- Paid budget paused, reduced, or reallocated
- Seasonal / calendar shift (holiday, end-of-month, day-of-week mismatch in comparison)
- Competitor action, PR event, or news coverage

**Tier C — Internal / brand**
- Landing page or funnel change (new design, copy test, form change)
- Email or push send (spike) or suppression (drop)
- Pricing or offer change
- Product outage or degraded performance (Core Web Vitals, server errors)

**Tier D — Channel-specific patterns**
- Organic: GSC impression/click check (algorithm, index, manual action)
- Paid: budget, bid, audience, creative rotation
- Email/push: send volume, deliverability, bounce rate, BIMI/authentication
- Direct: branded search volume shift (use GSC "site:" queries as a proxy)

For each probable cause: state the evidence available, the evidence missing, and one diagnostic action that would confirm or rule it out.

### Step 5 — Draft anomaly brief

```
## GA4 Anomaly Brief — [Brand] — [Date]
**Property:** [ID]
**Period:** [anomaly period] vs [baseline]

### Alert Summary
[One sentence: metric, direction, magnitude, primary cause or "cause unclear — see diagnostics"]

### Metrics
| Metric | Expected | Actual | Delta | Severity | DQ Status |
|---|---|---|---|---|---|

### Data-Quality Gate
[Pass / Fail per check — if any fail, lead with this section and mark affected metrics UNRELIABLE]

### Probable Causes (ranked by evidence weight)
1. [Most likely] — [evidence] — [confirm via: ...]
2. [Next] — ...

### Diagnostics Needed
- [ ] [Specific action] → [what it rules in/out]

### Recommended Next Action
[One concrete action the operator should take in the next 24h]
```

Save to `./analytics/anomaly-[brand]-[YYYY-MM-DD].md` if the user asks for a file; otherwise present inline.

---

## The anomaly-detection framework

This skill applies a **rolling-baseline Z-score model with weekday-seasonality correction**. This is an independent statistical method, not a reproduction of GA4's own Intelligence/anomaly detection — Google's built-in detection uses a Bayesian state-space (probabilistic) model with a training period and forecast confidence band [verify], and does not expose anomaly scoring at the metric+dimension level. The Z-score approach here is a transparent, hand-checkable substitute.

**Why not just WoW delta?** Week-over-week is the most common mistake. A Tuesday-vs-Tuesday comparison already has lower variance than a Tuesday-vs-Monday one, but it still misses: multi-week trend drift, holidays that shift day-of-week patterns, and slowly decaying metrics that never show a large single-week jump.

**The four-week same-weekday baseline** (mean and SD computed across the prior 4 occurrences of the same weekday) is the minimum robust baseline for most marketing metrics. For low-volume events (<200/period), use a 6–8-week window; for high-volume daily metrics, 4 weeks is sufficient.

**Seasonality flag:** if the anomaly period overlaps a known high-variance week (BFCM, New Year, tax season for finance verticals), widen the threshold by one tier before escalating. Note the adjustment in the brief.

---

## Principles

- **Data quality before causation.** A metric reading is only interpretable after the instrumentation is confirmed sound. Every run passes through the data-quality gate. No exceptions.
- **Rank causes by evidence, not by narrative.** Tag breakage is more common than algorithm updates; algorithm updates are more common than competitor sabotage. Follow the prior.
- **Name the missing evidence.** "Conversions dropped 35% — probably the algorithm" is an opinion. "Conversions dropped 35%; GSC impressions are unchanged (rules out organic), last deploy was Thursday at 14:00 — check if the thank-you page GA4 tag is firing" is analysis.
- **No invented proof.** Every number is from data the user supplied or noted `[verify]`. Never fabricate a baseline.
- **Severity calibration matters.** A 15% drop in a noisy metric is a Watch. A 15% drop in a stable high-value conversion metric is an Alert. Context drives threshold, not just percentage.
- **One action next.** Every brief ends with one concrete diagnostic or fix — not a menu of five things to "keep an eye on."

## What Not to Do

- Don't skip the data-quality gate even when the user is confident it's a real change.
- Don't stack multiple speculative causes — rank and confirm one before adding another.
- Don't treat WoW delta as the sole signal; always correct for weekday seasonality.
- Don't call a 10% fluctuation in a noisy metric a "significant drop."
- Don't invent a baseline from memory or training data; if data wasn't provided, ask.
- Don't present a causal narrative without naming the evidence that would falsify it.
- Don't conflate a GA4 data-freshness lag (standard reports are 24–48h behind) with a real drop.

## Quality Checklist (self-review before presenting)

- `brand-brain` called first; GA4 property ID and known quirks loaded?
- `data-qa-measurement-gotcha-checker` invoked and all gate results recorded in the brief?
- Any instrumentation failure surfaced as the primary finding, with affected metrics marked `[UNRELIABLE]`?
- Baseline uses weekday-corrected rolling window (not raw WoW); seasonality noted if applicable?
- Z-score / severity tier computed and labeled for each anomalous metric?
- Causes ranked by evidence weight, not by narrative appeal; missing evidence named explicitly?
- Brief ends with one concrete next action (not a list of "monitor these")?
- `analytics-report-reviewer` has reviewed the draft and no unsupported causal claims remain?
