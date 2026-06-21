---
name: growth-reporting-auto-compiler-weekly-monthly
description: >
  GA4 + GSC + paid + CRM data on a recurring cadence → ONE board-ready weekly or monthly growth
  report: traffic, channel ROI, paid performance, a diagnostic of what moved and why, anomaly
  callouts, and a ranked recommendation list — exec-narrative on top, evidence underneath. This is
  a FLAGSHIP ORCHESTRATOR: it does not re-implement analytics: it CHAINS the library's reporting
  skills (`ga4-weekly-traffic-digest`, `channel-roi-scorecard`, `weekly-paid-performance-summary`,
  `growth-diagnostic-deep-dive`), runs every number through `data-qa-measurement-gotcha-checker`
  before it trusts it, and hands the synthesized story to `stakeholder-update-status-writer` for the
  final voice. It calls `brand-brain` first for context. Use whenever the user says "compile my
  weekly/monthly growth report," "growth report," "exec report," "board report," "marketing report,"
  "month in review," "WoW/MoM report," "put together the numbers for the leadership update," "run the
  weekly report," or hands over GA4/GSC/ads/CRM exports and asks for one polished report. It compiles
  and narrates a report from existing skills — it is not a dashboard, not a single-channel digest, and
  it does not invent numbers.
---

# Growth Reporting Auto-Compiler (Weekly / Monthly)

Point it at your GA4, Search Console, paid, and CRM data once a week or once a month and get back the report a growth lead would otherwise spend a day assembling: traffic, channel economics, paid spend, a diagnosis of *why the numbers moved*, the anomalies worth flagging, and a short ranked list of what to do next — all in one document, exec narrative on top, evidence and tables underneath, written in the brand's voice.

This is a **growth team in a box**: it does not query GA4 itself, score channels itself, or analyze paid itself. It **orchestrates** the specialist skills that already do those jobs, QA's their output before trusting a single figure, then synthesizes one coherent story. It replaces a junior marketer's reporting week — without inventing a number, ever.

It compiles and narrates. It is not a live dashboard, not a single-channel digest, and it will not paper over a missing data source with a guess — a gap is reported as a gap.

---

## Skills this calls

In pipeline order (every one already exists — invoke via the **Skill tool**; never re-implement a stage):

- **`brand-brain`** (required, first) — loads the active brand's voice, ICP, positioning, offer, and proof so the narrative and recommendations are on-voice and on-strategy. This orchestrator never writes `brand.md` itself.
- **`ga4-weekly-traffic-digest`** — sessions, users, channels, top pages, WoW/MoM delta. The traffic spine.
- **`channel-roi-scorecard`** — spend + revenue/conversions per channel → CPA, ROAS, ranked channel table, marketing-P&L contribution.
- **`weekly-paid-performance-summary`** — Google/Meta/LinkedIn raw stats → plain-English paid commentary (spend, ROAS, CPA per channel).
- **`growth-diagnostic-deep-dive`** — the *why* layer: turns the assembled metrics into a diagnosis of what's driving and dragging growth plus a prioritized lever list.
- **`data-qa-measurement-gotcha-checker`** — the gate. Run each upstream output through it to catch self-referral, spam hostnames, broken events, model blending, attribution flips, and dedupe overlapping anomaly callouts **before** they reach the report.
- **`stakeholder-update-status-writer`** (final stage) — turns the QA'd, synthesized findings into the board-ready narrative (context · progress · risks · next steps) in the brand's voice and the right register for the audience.

*Optional, when relevant and installed:* `ltv-cac-payback-calculator` (add unit-economics when CRM revenue/cohort data is present), `funnel-drop-off-analyzer` (when a conversion-rate move needs funnel-level cause), `sql-query-generator-for-ga4-bigquery` (when the user has BigQuery export instead of standard GA4). Skip silently if absent.

---

## How a run works

The pipeline. Each stage's output is the next stage's input — that handoff is the whole point.

```
Stage 0  Brand + scope   ─► brand-brain → cadence, audience, period, data sources confirmed
Stage 1  Traffic         ─► ga4-weekly-traffic-digest        → traffic block + deltas
Stage 2  Channel ROI     ─► channel-roi-scorecard            → ranked channel economics
Stage 3  Paid            ─► weekly-paid-performance-summary   → paid commentary
            └── GATE ──►  data-qa-measurement-gotcha-checker (runs across Stages 1–3 + dedupe)
Stage 4  Diagnose        ─► growth-diagnostic-deep-dive       → why-it-moved + ranked levers
Stage 5  Synthesize      ─► merge into one narrative + recommendation list
Stage 6  Narrate         ─► stakeholder-update-status-writer  → board-ready report
Stage 7  Human approval  ─► save bundle to ./growth-reports/...
```

### Stage 0 — Load the brand and frame the run (always first)
**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the request and any named brand. Use the returned digest — voice, banned words, ICP, positioning, North-Star/growth priorities — to set the lens for *what counts as good or bad* and the voice for the final write-up. Do not produce a report before it returns.
**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user to install `brand-brain` (preferred) or answer a 3-question mini-setup (what it is · who the report is for · top 2 growth priorities), then proceed.

Then confirm the **run frame** in one batch: **cadence** (weekly vs monthly), **period + comparison window** (this week vs last; this month vs last + YoY if available), **audience** (team Slack vs exec/board — drives depth and tone), and **which data sources are available** (GA4 property, GSC site, paid accounts, CRM/revenue export). Missing sources are noted now and reported later as gaps — never filled in by guessing.

### Stage 1 — Traffic (`ga4-weekly-traffic-digest`)
Invoke it for the property + period + comparison window. **Handoff out:** the traffic block — sessions, users, channel mix, top pages, WoW/MoM deltas, and any GSC movement — passed forward as the demand-side picture.

### Stage 2 — Channel ROI (`channel-roi-scorecard`)
Pass the spend-by-channel + revenue/conversion data. **Handoff out:** the ranked channel table — CPA, ROAS, contribution — so the report can say which channels are earning their budget, not just which got traffic.

### Stage 3 — Paid (`weekly-paid-performance-summary`)
Pass the raw paid stats (Google/Meta/LinkedIn). **Handoff out:** plain-English paid commentary that explains the spend and efficiency moves inside the channel table from Stage 2 (they must reconcile — see the gate).

### Gate — Measurement QA (`data-qa-measurement-gotcha-checker`)
Before any of Stages 1–3 are trusted, run their outputs through the gotcha checker. It flags the classics (self-referral, spam hostnames, broken/duplicated events, model blending, attribution flips, double-counted conversions) **and** dedupes anomaly callouts that three stages surfaced about the same underlying event. **Branch:** if it flags a figure as untrustworthy, mark it `[verify]` in the report (or drop it) and note the caveat — never launder a suspect number into the narrative. If two stages disagree on the same metric, reconcile here or flag the discrepancy explicitly.

### Stage 4 — Diagnose (`growth-diagnostic-deep-dive`)
Feed it the **QA'd** traffic + channel + paid picture (plus CRM revenue if present). **Handoff out:** the causal layer — what's driving and dragging growth — and a *prioritized lever list*. This is what turns a stack of metrics into a report a leader can act on.

### Stage 5 — Synthesize
Merge the four stages into one structure (template below). De-duplicate findings, reconcile any remaining cross-stage conflicts, and rank the diagnostic's levers against the brand's stated priorities from Stage 0. Decide the 3–5 headline takeaways and the recommendation shortlist. Mark anything still unverified `[verify]`.

### Stage 6 — Narrate (`stakeholder-update-status-writer`)
Hand the synthesized, QA'd findings to the status writer with the audience + voice from Stage 0. **Handoff out:** the exec narrative — TL;DR, what changed, why, risks, next steps — in the brand's voice and the audience's register. Tables and evidence stay attached beneath it.

### Stage 7 — Human approval + save
Present the assembled report. On approval, save the bundle (below). For a recurring cadence, offer to register it (e.g. a scheduled task) so next period runs the same pipeline against the new window — but the human approves the *first* run of any new cadence.

---

## Report structure (the compiled deliverable)

```
# [Brand] Growth Report — [Weekly | Monthly] · [period] vs [comparison]
Prepared for: [audience] · Brand: [slug, via brand-brain] · Data sources: [GA4 / GSC / paid / CRM]
[⚠ data-quality caveats from the QA gate, if any]

## TL;DR            3–5 headline takeaways + the single most important number
## What changed     Traffic · Channel ROI · Paid — deltas with WoW/MoM (+ YoY if monthly)
## Why it moved     Diagnostic narrative — drivers and drags, not just "down 8%"
## Anomalies & risks  Flagged callouts (QA'd, deduped) + what's at risk if ignored
## Recommendations  Ranked levers tied to brand priorities — each with owner-ready next step
## Appendix         The full tables from each stage + [verify] items + source/method notes
```

**Bundle on save:** to `./growth-reports/[brand]-[cadence]-[YYYY-MM-DD]/` (leading `./` = the user's project/CWD, never the skill folder):
`report.md` (the compiled narrative) · `traffic.md` · `channel-roi.md` · `paid.md` · `diagnostic.md` · `qa-notes.md`. Offer a single-file `report.md` for users who just want the one doc.

---

## Orchestration logic (gates, branches, loops)

- **Brand gate.** No stage runs before `brand-brain` returns.
- **Source gate.** Run only the stages whose data exists. Missing GA4 → no traffic block (say so). No paid spend → skip Stages 2–3's paid math and note it. A partial report that's honest beats a complete one that's invented.
- **QA gate (hard).** A figure the gotcha checker flags cannot enter the narrative clean — it's `[verify]`, caveated, or dropped. Cross-stage conflicts are reconciled or flagged, never silently averaged.
- **Diagnostic-weak branch.** If `growth-diagnostic-deep-dive` returns low-confidence or "insufficient data," do not manufacture causes — report the *correlation* and explicitly call the *cause* unknown, and list what data would resolve it.
- **Narrative loop.** If the Stage-6 narrative buries the lead, mixes audiences, or drifts off-voice, send it back to `stakeholder-update-status-writer` with a one-line correction. Authoring and review are separate passes — don't self-approve voice in the same breath you wrote it.
- **Human approval.** The human signs off before the report ships and before any new recurring cadence is registered. Subsequent same-cadence runs can auto-compile, but anomalies above a set threshold should still ping a human.

---

## Principles

- **Orchestrate, don't reimplement.** Every number comes from a specialist skill. This skill's job is sequencing, QA, synthesis, and narrative — not querying.
- **Brand-brain first, voice throughout.** The report reads like the brand wrote it and judges metrics against the brand's actual priorities.
- **QA before narrative.** No figure reaches the exec summary until the gotcha checker has cleared it. Trust is earned per number.
- **Diagnosis over description.** "Organic down 8% because the top-3 commercial pages slipped to page 2" beats "organic down 8%." The *why* is the deliverable.
- **Truth discipline.** Real numbers or `[verify]`. Never invent a metric, a cause, a benchmark, or a quote. A gap is a gap.
- **Audience-shaped.** Weekly-team and monthly-board are different documents; depth, tone, and length follow the Stage-0 audience.
- **Recurring by design.** Built to run the same pipeline every period against a fresh window — same structure, comparable deltas.

## What not to do

- Don't produce any report before `brand-brain` returns the active brand.
- Don't re-implement GA4 querying, channel scoring, paid analysis, diagnosis, or status-writing — call the sibling skills.
- Don't let an un-QA'd or QA-flagged number into the narrative as if it were clean.
- Don't invent numbers, causes, competitor benchmarks, or quotes to fill a gap — flag the gap.
- Don't silently average two stages that disagree — reconcile or flag.
- Don't ship a diagnostic conclusion the data doesn't support; separate correlation from cause.
- Don't self-approve the final narrative's voice — run the review pass separately.
- Don't save the bundle into the skill folder; use the project-relative `./growth-reports/` path.

## Quality checklist (self-review before presenting)

- `brand-brain` called first; cadence, period+comparison, audience, and available sources confirmed?
- Every available stage run in order, each handed the prior stage's output (not raw data re-fetched)?
- Skipped stages were skipped for *missing data*, and that gap is stated in the report?
- `data-qa-measurement-gotcha-checker` run across Stages 1–3; flagged figures `[verify]`/caveated/dropped; cross-stage conflicts reconciled; anomalies deduped?
- Diagnostic levers ranked against the brand's Stage-0 priorities; correlation vs cause kept honest?
- Final narrative came from `stakeholder-update-status-writer`, on-voice, audience-correct, lead not buried (looped if weak)?
- TL;DR, what-changed, why, anomalies/risks, ranked recommendations, and appendix all present; every number traceable to a stage or marked `[verify]`?
- Bundle saved to `./growth-reports/...`; recurring cadence offered but first run human-approved?
