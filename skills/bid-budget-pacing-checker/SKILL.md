---
name: bid-budget-pacing-checker
description: >
  Spend data, monthly budget, days elapsed → pacing status per campaign with anomaly flags and
  cross-channel reallocation suggestions. Takes raw spend numbers (pasted, CSV, or typed) plus
  the monthly budget and calendar position, and returns a pacing scorecard for every campaign:
  on-pace / under-pacing / over-pacing, burn-rate delta, projected end-of-month spend, an anomaly
  flag for anything statistically unusual, and a ranked reallocation recommendation table so
  budget moves to the highest-ROAS channel before the month ends. Optionally accepts ROAS or
  CPA alongside spend so the reallocation column is efficiency-weighted, not just spend-weighted.
  Calls brand-brain to anchor ICP, offer, and channel priorities so reallocation suggestions
  respect strategic intent — not just raw numbers. Use when the user says "check my pacing,"
  "am I on track to hit budget," "where should I reallocate spend," "which campaigns are
  overspending," "pull my pacing report," "budget check," or pastes a spend snapshot and wants
  a verdict.
---

# Bid & Budget Pacing Checker

Pasting spend data into a spreadsheet and manually dividing by day count is how budget leaks happen. This skill automates the math, surfaces anomalies, and turns raw numbers into a reallocation decision — in one pass, anchored to the brand's channel strategy and ICP.

Give it spend + budget + days elapsed. Get a pacing scorecard, anomaly flags, and a ranked cross-channel reallocation table — with a clear recommendation on where to shift before month-end.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's ICP, offer, positioning, and channel priorities so reallocation suggestions honor strategic intent, not just efficiency math.
- *(optional, when installed)* `channel-roi-scorecard` for ROAS/CPA inputs if the user hasn't already supplied them; `weekly-paid-performance-summary` to pull formatted channel stats before running pacing.

---

## How a run works

```
Step 0  Load the brand  ──► call brand-brain (ICP, offer, channel priorities)
Step 1  Ingest inputs   ──► spend data, budget, days elapsed (+ optional ROAS/CPA)
Step 2  Calculate       ──► pacing status, burn rate, projected EOM spend per campaign
Step 3  Flag anomalies  ──► statistical and operational outliers
Step 4  Recommend       ──► ranked reallocation table + executive verdict
Step 5  Persist         ──► offer to save to ./reports/
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`) before producing any output. It returns the active brand's digest — ICP, offer mechanics, positioning, and any channel hierarchy the brand has documented. Use channel priorities as a tie-breaker in reallocation recommendations: when two channels have similar ROAS, favor the one aligned to the ICP's primary channel. Obey banned-words in any copy-adjacent output (recommendations, commentary).

Fallback if `brand-brain` is absent: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none, ask the user to install `brand-brain` or answer a 3-question mini-setup (primary ICP · primary conversion channel · any budget caps or channel hold-backs), then proceed.

### Step 1 — Ingest inputs

Required:
- **Spend data** — per-campaign or per-channel actual spend to date (pasted table, CSV, or typed)
- **Monthly budget** — total or per-campaign (total works if the user wants a cross-channel view)
- **Days elapsed** — calendar days into the month (or "today is the 14th")

Optional but materially improve the reallocation column:
- ROAS or CPA per campaign/channel
- Platform (Google Ads, Meta, LinkedIn, etc.) — unlocks platform-specific anomaly patterns
- Hold-backs or hard caps (e.g. "never exceed $X on LinkedIn")

If days elapsed is missing, ask. Everything else can be inferred or flagged.

---

## The Pacing Framework — Linear Burn Rate + Efficiency Weighting

The base model is **linear burn rate** (simplest defensible baseline), then adjusted for efficiency when ROAS/CPA is available.

### Core formulas

| Metric | Formula |
|---|---|
| Expected spend to date | `Monthly budget × (days elapsed ÷ days in month)` |
| Pacing ratio | `Actual spend ÷ Expected spend` |
| Projected EOM spend | `Actual spend ÷ days elapsed × days in month` |
| Remaining budget | `Monthly budget − actual spend` |
| Daily run rate needed | `Remaining budget ÷ days remaining` |

**Pacing status thresholds** (adjustable by platform — Google and Meta smooth delivery; LinkedIn and programmatic can spike):

| Pacing ratio | Status | Flag color |
|---|---|---|
| < 0.80 | Under-pacing | 🔴 |
| 0.80 – 0.90 | Slightly under | 🟡 |
| 0.90 – 1.10 | On pace | 🟢 |
| 1.10 – 1.25 | Slightly over | 🟡 |
| > 1.25 | Over-pacing / at risk | 🔴 |

### Efficiency weighting (when ROAS/CPA supplied)

Reallocation priority = `(efficiency score rank × 0.6) + (pacing headroom rank × 0.4)`

- Efficiency score: ROAS (higher = better) or CPA (lower = better), normalized across campaigns
- Pacing headroom: remaining budget as % of monthly, weighted toward campaigns running under-pace with room to scale

Hold-backs and caps from the brand context override any pure-math recommendation.

---

## Output format — the Pacing Scorecard

Present as a table, then the reallocation section, then the executive verdict.

```
## Pacing Scorecard — [Brand] · [Month / Period]
Inputs: [total budget · days elapsed · days remaining · data as of]

| Campaign / Channel | Budget | Spent | Expected | Pacing % | Status | Proj. EOM | ROAS/CPA |
| --- | --- | --- | --- | --- | --- | --- | --- |
...

## Anomaly Flags
[flag + one-line explanation per anomaly]

## Reallocation Recommendations
| Move budget from | Move budget to | Amount | Rationale |
...

## Executive Verdict
[2–3 sentences: overall health, biggest risk, one clear action]
```

If the user has fewer than 3 campaigns, collapse to a simpler inline block — the table overhead isn't worth it.

---

## Anomaly flags to check

Encode these regardless of whether the user asks:

- **Spend cliff** — any campaign that ran normally then dropped to near-zero mid-period (budget exhausted, paused, or disapproved ad)
- **Spend spike** — any campaign with a daily spend 2× the prior 7-day average (bid strategy change, auction volatility, overlap with competitor exit)
- **Under-pacing with high ROAS** — money being left on the table; the reallocation recommendation should flag this first
- **Over-pacing toward a hard cap** — project if it will hit the cap before month-end; recommend a daily budget reduction
- **Zero spend** — campaign exists in the data but has $0; likely paused or disapproved — flag, don't just report $0
- **Attribution window mismatch** — if the user mixes 1-day click ROAS with 7-day click ROAS across campaigns, flag it; the efficiency weighting will be apples-to-oranges
- **Platform data lag** — Google Ads has ~3-hour lag; Meta can lag 24–48 hours on view-through conversions; warn if the user is pulling "today" data

---

## Principles

- **Math before opinions.** Run the linear burn rate on every campaign before forming any view. Don't eyeball.
- **Anomalies are their own section.** Don't bury a spend cliff in the main table commentary.
- **Reallocation respects the brand.** Channel strategy from `brand-brain` is a constraint, not a suggestion.
- **Efficiency weighting requires verified inputs.** If ROAS/CPA numbers are user-supplied and unverified, note `[user-provided]` and don't treat them as authoritative — recommend the user cross-check against the platform UI.
- **Project EOM, not just current pacing.** The only number that matters is whether you end the month at budget. Always show projected EOM spend.
- **Data lag is a gotcha.** Always ask when the spend data was pulled; stale data turns a correct analysis into a wrong recommendation.

## What Not to Do

- Don't produce any output before `brand-brain` returns.
- Don't recommend reallocation based solely on spend volume — efficiency (ROAS/CPA) must be the primary weight when available.
- Don't infer a campaign is performing well just because it's on-pace — pacing and efficiency are separate axes.
- Don't mix attribution windows across campaigns in the efficiency column without flagging the mismatch.
- Don't recommend moving budget into a channel the brand has documented as a hold-back.
- Don't skip the anomaly section even when everything looks on-pace — a spend cliff could be hiding behind an average.
- Don't pad the output with platform setup instructions — this skill reads pacing data, it doesn't configure campaigns.

## Quality Checklist

- `brand-brain` called and active brand loaded before any calculations or recommendations?
- All pacing ratios and projected EOM spends calculated from the formulas — no eyeballing?
- All five common anomaly patterns checked (cliff, spike, under-pace + high ROAS, hard-cap risk, zero spend)?
- Attribution window mismatch flagged if ROAS/CPA inputs span different windows?
- Reallocation table efficiency-weighted when ROAS/CPA available; brand channel hold-backs respected?
- Executive verdict present: overall health + biggest risk + one clear action?
- Data-lag caveat noted if user pulled live or same-day data?
- Offer to save report to `./reports/pacing-[brand]-[YYYY-MM].md` if the output is more than 3 campaigns?
