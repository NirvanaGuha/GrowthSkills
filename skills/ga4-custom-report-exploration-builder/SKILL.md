---
name: ga4-custom-report-exploration-builder
description: >
  Business question + GA4 dimensions/metrics → custom report config AND Exploration surface
  spec (free-form, path, funnel), plus a standup-ready summary. Takes a concrete analytical
  question ("which landing pages convert new users from paid search?", "where does the
  checkout funnel drop?", "what is the path after a feature event?") and produces three
  deliverables: (1) a GA4 Reports Builder configuration — metric/dimension set, filter
  conditions, comparison groups; (2) an Explorations spec — surface type, segments, steps,
  variables panel, and tab settings ready to replicate in the UI; (3) a four-bullet
  standup-ready narrative with key numbers, anomalies, and the one recommended action.
  Encodes the seven classic GA4 measurement gotchas so every spec ships with the right
  caveats baked in. Composes data-qa-measurement-gotcha-checker for a data-quality gate,
  analytics-report-reviewer for a final narrative review, and tracking-plan-taxonomy-builder-auditor
  to catch event gaps before they become blind spots. Use when the user asks to "build a GA4
  report," "set up an exploration," "analyze [metric] in GA4," "custom report for [question],"
  "GA4 funnel," "path analysis," "free-form exploration," "segment [users] in GA4," or hands
  over a business question and asks what they should be looking at.
---

# GA4 Custom Report & Exploration Builder

Turn a business question into a production-ready GA4 spec — not a vague "here's what to look at" memo. Every run produces a report or exploration config you can replicate in the GA4 UI in under ten minutes, plus a standup-ready narrative framing what the numbers actually mean.

This skill specs and narrates. It does not read your GA4 live (use `ga4-weekly-traffic-digest` or `pushengage-analytics` for live pulls). If your event taxonomy is the real problem, it says so and hands off to `tracking-plan-taxonomy-builder-auditor` instead of papering over gaps.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's analytics context: property IDs, hostname filters, known data-quality caveats, conversion events, and any brand-specific banned interpretations.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for the GA4 property ID, primary hostname filter, and the top 2–3 conversion events before proceeding.
- **`data-qa-measurement-gotcha-checker`** (required, Step 1 gate) — runs the seven classic GA4 gotchas against the proposed spec before any report config is written; blocks specs that would produce misleading output.
- **`tracking-plan-taxonomy-builder-auditor`** (conditional) — invoked when Step 1 finds event gaps (missing events, broken key-event marking, no conversion funnel events); produces remediation spec before exploration config.
- **`analytics-report-reviewer`** (Step 4) — reviews the standup narrative for unsupported claims, missing context, and interpretation errors before it's handed to a stakeholder.
- **`looker-studio-live-dashboard-builder`** — call when the user wants the exploration turned into a persistent dashboard rather than a one-off Exploration.
- **`kpi-tree-builder`** — call when the business question reveals an undefined north-star metric; don't spec a report against a metric that hasn't been defined.
- **`attribution-model-configurator`** — call when the question involves comparing channels, session sources, or cross-channel revenue attribution; don't derive attribution logic inline.

---

## How a run works

```
Step 0  Load brand context   ──► brand-brain (property IDs, hostname, conversion events, caveats)
Step 1  Data-quality gate    ──► data-qa-measurement-gotcha-checker (7 GA4 gotchas)
            └── if event gaps found → tracking-plan-taxonomy-builder-auditor (remediation)
Step 2  Pick the surface     ──► Reports Builder | Free-Form | Path | Funnel | Segment Overlap
Step 3  Build the spec       ──► config block (dimensions/metrics/filters/segments/steps)
Step 4  Write the narrative  ──► standup summary → analytics-report-reviewer pass
Step 5  Save artifacts       ──► ./analytics/[slug]-ga4-spec.md
```

---

## Step 0 — Load brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Extract from the returned digest:
- **Property ID** — required for every spec block
- **Hostname filter** — exclude internal traffic, dev subdomains, and known spam hostnames; record the filter condition explicitly
- **Key conversion events** — the events already marked as key events in GA4; never spec a funnel around an event that isn't one
- **Known caveats** — any data-quality notes already documented for this brand (broken `page_view`, OAuth-redirect attribution distortion, SDK↔direct flip-flop, powered-by carve-out, etc.)

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for the GA4 property ID, primary hostname filter, and the top 2–3 conversion events before proceeding.

---

## Step 1 — Data-quality gate (the seven GA4 gotchas)

Call `data-qa-measurement-gotcha-checker` before writing any spec. It evaluates the proposed question + event set against all seven classic GA4 measurement failure modes:

| # | Gotcha | Block condition |
|---|--------|----------------|
| 1 | **Sampling** | Exploration date range > ~90 days on a high-volume property; recommend shorter windows or BigQuery export |
| 2 | **`(not set)` dimension values** | Landing page, session source, or campaign dimension missing for >15% of sessions; flag before filtering |
| 3 | **Self-referral / hostname pollution** | Cross-domain or subdomain traffic inflating sessions; hostname filter required |
| 4 | **Attribution window mismatch** | Comparing paid conversions using GA4's default 30-day click window against a 7-day ad-platform window; note the gap |
| 5 | **SRM (Sample Ratio Mismatch)** | Exploration segment sizes diverging >10% from expected split; note before drawing conclusions |
| 6 | **Key-event marking gaps** | Funnel or conversion spec references events not marked as key events in GA4 Admin |
| 7 | **Session vs. user scope confusion** | Mixing session-scoped and user-scoped dimensions in the same report (e.g., `session_source` with `user_engagement_time`); will produce misleading joins |

If any gotcha fires a **block**, halt the spec and surface the remediation first. If it fires a **warning**, annotate the spec with the caveat inline — don't silently omit it.

---

## Step 2 — Pick the right GA4 surface

| Business question shape | Right surface |
|------------------------|---------------|
| "Which [dimension] drives [metric]?" | **Reports Builder** — standard report with secondary dimension + comparison |
| "How do users move through [steps]?" | **Funnel Exploration** — open or closed funnel, with breakdown dimension |
| "What do users do after [event]?" | **Path Exploration** — forward path from event node |
| "What path leads to [conversion]?" | **Path Exploration** — backward path to event node |
| "Which segments overlap (e.g., email + paid)?" | **Segment Overlap** exploration |
| "Arbitrary cross-tab of events + user properties?" | **Free-Form Exploration** |
| "Cohort retention after first [event]?" | **Cohort Exploration** |

When the question is ambiguous, name the ambiguity and offer two surface options with a clear recommendation.

---

## Step 3 — Build the spec

Every spec block is a structured, UI-replicable config. Use this template:

### Reports Builder config

```
## GA4 Report Config — [Question slug]
Property: [ID]
Report type: Detail | Overview | Realtime
Date range: [range] | Comparison: [period or segment]
Hostname filter: hostname exactly matches [value] (applied as report filter)
Dimensions: primary=[dim], secondary=[dim]
Metrics: [list in order]
Filters: [dimension] [operator] [value]
Comparison groups: (none | Segment A vs. Segment B)
Sort: [metric] [DESC/ASC]
Gotcha annotations: [inline caveats from Step 1]
```

### Exploration spec

```
## GA4 Exploration Spec — [Question slug]
Surface: [Free-Form | Funnel | Path | Segment Overlap | Cohort]
Exploration name: [human-readable]
Date range: [range]
Segments (Variables panel):
  - Segment 1: [name] — condition(s)
  - Segment 2: [name] — condition(s)   (if applicable)
Dimensions (Variables panel): [list]
Metrics (Variables panel): [list]

Tab settings:
  Technique: [surface]
  [Funnel] Steps:
    Step 1: [event name] — optional condition
    Step N: ...
    Type: Open / Closed
    Breakdown: [dimension]
    Elapsed time: on/off
  [Path] Starting/ending point: [event or page]
  [Free-Form] Rows: [dim], Columns: [dim or none], Values: [metrics], Filter: [condition]

Gotcha annotations: [inline]
```

Dimension and metric names use GA4's exact API names (e.g., `sessionDefaultChannelGroup`, `screenPageViews`, `keyEvents`) so they copy-paste directly into the Variables panel without lookup.

---

## Step 4 — Standup-ready narrative

Four bullets, tight. No weasel-words.

```
## Standup Summary — [Question], [Date range]
Brand: [slug] | Property: [ID]

- WHAT: [one sentence on what the data shows — specific numbers, not "significant increase"]
- ANOMALY: [one sentence on anything unexpected, or "No anomalies in this window"]
- CAVEAT: [most important gotcha annotation — sampling, (not set) %, attribution window]
- ACTION: [one recommended next step — test, investigate, fix, or double down]
```

Invoke `analytics-report-reviewer` on this block before presenting. If reviewer flags an unsupported claim or missing context, revise before handing to stakeholder.

---

## Step 5 — Persist artifacts

Save the full spec + narrative to `./analytics/[brand-slug]-[question-slug]-ga4-spec.md`. Do not overwrite if the file exists — append a datestamped version block. Never write to the brand-brain data root or skill folder.

---

## The GA4 Exploration framework (opinionated baseline)

GA4 Explorations uses a **Variables / Tab Settings** split: you drag dimensions, metrics, and segments from the Variables panel into the Tab Settings canvas. Specs must mirror this split — dimensions and metrics listed in the Variables section, then explicitly assigned in Tab Settings — or they're useless in the UI.

**Segment scoping rule.** User segments persist across sessions (right for LTV/cohort questions). Session segments isolate one visit (right for channel-level funnel questions). Event segments apply per-event (right for interaction-level comparisons). Mixing scopes without intent is Gotcha 7. Every segment spec states its scope explicitly.

**Funnel discipline.** Closed funnels require users to complete steps in order; open funnels count re-entries. For checkout abandonment: closed + strict order. For feature adoption: open. State the choice and the reason.

**Path analysis limit.** GA4 Path Exploration surfaces up to 10 nodes by default; the full path requires BigQuery. If the question needs >10 steps, note this and suggest `sql-query-generator-for-ga4-bigquery` for the BigQuery equivalent.

---

## Principles

- **Spec to replicate, not to advise.** Every output is a step-by-step UI config, not a list of things to consider.
- **Gotchas are non-negotiable.** If `data-qa-measurement-gotcha-checker` blocks a spec, the spec doesn't ship until the underlying issue is addressed.
- **Exact API names only.** GA4 UI labels drift from API names; always use the API name so the spec copy-pastes cleanly.
- **Scope every segment.** User, session, or event scope is stated in every segment definition — never left implicit.
- **Brand-brain first.** Property ID, hostname filter, and known caveats come from the brand; never hardcode or assume them.
- **Truth discipline.** Numbers in the narrative are from the data the user provides; use `[verify]` for anything you're inferring.

## What Not to Do

- Don't write a spec before `data-qa-measurement-gotcha-checker` runs — a spec built on broken data is worse than no spec.
- Don't use GA4 UI labels as dimension/metric names in the spec (e.g., "Sessions" not `sessions` — use the API name).
- Don't spec a funnel around events that aren't marked as key events; fix the key-event marking first.
- Don't mix session-scoped and user-scoped dimensions in the same exploration without an explicit scope note.
- Don't produce a standup narrative for a date range that triggers the sampling warning without noting it.
- Don't rebuild attribution logic — call `attribution-model-configurator` when channel comparison is the question.
- Don't build a dashboard spec — hand off to `looker-studio-live-dashboard-builder` for anything persistent.

## Quality Checklist

- `brand-brain` called; property ID, hostname filter, and known caveats loaded?
- `data-qa-measurement-gotcha-checker` run; all block-level gotchas resolved; warning-level gotchas annotated inline?
- Surface choice stated with rationale; dimensions/metrics listed using exact GA4 API names?
- Exploration spec mirrors the Variables / Tab Settings split; segment scope (user/session/event) explicit?
- Funnel type (open/closed), path direction (forward/backward), and breakdown dimension all stated?
- Standup narrative: four bullets, specific numbers, one concrete action, `analytics-report-reviewer` pass done?
- Artifact saved to `./analytics/[brand-slug]-[question-slug]-ga4-spec.md` without overwriting prior versions?
