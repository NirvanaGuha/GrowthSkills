---
name: looker-studio-live-dashboard-builder
description: >
  KPIs + data sources → a buildable Looker Studio spec with calculated fields,
  data-source blends, connector configuration, layout/widget map, and sharing
  settings — or a shareable auto-refreshing dashboard asset ready to hand to a
  stakeholder. Two modes: Spec Mode produces a complete implementation blueprint
  (connector config, blend keys, calculated fields with formulas, report-page
  layout, share/refresh settings) a marketer or analyst can execute verbatim;
  Asset Mode generates a self-contained HTML summary dashboard (inline charts,
  styled tables, key metric tiles) when Looker Studio API access isn't available.
  Composes brand-brain for brand context and data-qa-measurement-gotcha-checker
  for data-quality gates before any metric hits a chart. Analytics measurement
  skills — ga4-custom-report-exploration-builder, tracking-plan-taxonomy-builder-auditor,
  kpi-tree-builder — are upstream inputs; analytics-report-reviewer reviews the
  finished output. Use whenever the user says "build a Looker Studio dashboard,"
  "set up a live dashboard," "connect GA4 to Looker Studio," "data blend,"
  "calculated field," "auto-refresh report," "share a live dashboard," or hands
  over a list of KPIs and asks for a reporting view.
---

# Looker Studio & Live Dashboard Builder

Give it KPIs and data sources, get a dashboard that actually ships. Spec Mode writes the full implementation blueprint — every connector setting, blend join key, calculated-field formula, and widget placement — so you can build it without guessing. Asset Mode hands you a styled HTML summary dashboard when you need something shareable right now.

Neither mode produces vague advice. Both produce executable specs or actual assets. Every metric is gated by a data-quality check before it hits a chart.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — resolves the active brand's slug, color palette, typography fallbacks, and banned terminology so the dashboard header and chart colors match the brand, not Looker Studio defaults. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, primary/secondary hex colors, and any terminology to avoid before proceeding.
- **`data-qa-measurement-gotcha-checker`** (Step 1, gate) — validates each proposed data source and metric against the canonical measurement gotchas (attribution windows, GA4 sampling thresholds, (not set) / (other) inflation, self-referral, session vs. event scope mismatches, SRM in experiment reports). Dashboard build does not proceed past the gate if blockers are found.
- **`kpi-tree-builder`** (optional upstream) — if the user hasn't defined KPIs yet, call this to derive a north-star → leading-indicator tree first; pass the output as the KPI list into Step 2.
- **`ga4-custom-report-exploration-builder`** (optional upstream) — when the primary source is GA4, use this to confirm the correct dimensions/metrics and exploration surface before writing the connector spec.
- **`tracking-plan-taxonomy-builder-auditor`** (optional upstream) — audit that the events driving calculated fields actually exist and are correctly scoped in the live data.
- **`analytics-report-reviewer`** (optional downstream) — after the spec or asset is produced, pass it here for a structured review of unsupported claims, missing context, and visualization issues.
- **`channel-roi-scorecard`** (optional) — when blending paid + organic channel data, compose this for shared channel ROI math rather than re-deriving CPL/ROAS formulas inline.
- **`ltv-cac-payback-calculator`** (optional) — when the dashboard includes LTV, CAC, or payback-period tiles, call this for the canonical formulas; reference its output in the calculated-field spec.

---

## How a run works

```
Step 0  Brand context     ──► brand-brain (palette, terminology)
Step 1  Data-quality gate ──► data-qa-measurement-gotcha-checker (block on hard gotchas)
Step 2  KPI inventory     ──► confirm metrics, scopes, and owner per KPI
Step 3  Source mapping    ──► map each KPI to connector + blend strategy
Step 4  Build the spec    ──► Spec Mode OR Asset Mode
Step 5  Self-review       ──► quality checklist before presenting
```

---

## Step 0 — Brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Capture: active slug, primary hex, secondary hex, typography fallback stack, banned words. Apply to the dashboard header, metric-tile backgrounds, chart color sequences, and any copy labels. If the brand uses a specific name for a metric (e.g., "Subscriber Value" not "LTV"), honor it throughout.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, primary/secondary hex colors, and any terminology to avoid before proceeding.

---

## Step 1 — Data-quality gate

Before writing any spec, invoke `data-qa-measurement-gotcha-checker` against every proposed source/metric pair. Treat its output as a gate:

- **Blocker** (e.g., event not firing, scope mismatch session vs. event, known broken metric): surface it explicitly, pause the build, and offer a remediation path.
- **Warning** (e.g., sampling above 5% row threshold, (not set) > 10% on a key dimension, 30-day attribution vs. 7-day window mismatch): note inline in the spec so the dashboard consumer understands the limitation.
- **Clear**: proceed.

Do not bury gotcha warnings in a footnote. Put them where the builder will see them — in the connector config section, adjacent to the affected calculated field.

---

## Step 2 — KPI inventory

Confirm the KPI list before building. For each metric collect:

| Field | What to resolve |
|---|---|
| **KPI name** | Brand-correct label (from Step 0) |
| **Metric type** | GA4 event-scoped / session-scoped / user-scoped / external |
| **Owner dimension** | What can it be broken down by? (channel, landing page, segment, date) |
| **Target / threshold** | Green/amber/red threshold for scorecard tiles |
| **Data source slug** | Which connector feeds it |
| **Blend required?** | Does it need a cross-source join? |

If the user hasn't defined KPIs, call `kpi-tree-builder` first and pass its output here.

---

## Step 3 — Source mapping and blend strategy

Map every KPI to its connector. For each source produce:

**Connector config block:**
```
Source: [Google Analytics 4 | Google Search Console | Google Ads | BigQuery | Google Sheets | manual CSV | other]
Property / resource ID: [value or [verify]]
Date dimension: [Date | YearMonth | ISOWeek]
Refresh schedule: [15 min | 4 h | 12 h | manual]
Filters baked into connector: [hostname = brand domain; exclude internal IPs [verify range]]
```

**Blend config (when cross-source join is needed):**
```
Blend name: [descriptive slug]
Left source: [connector A]
Right source: [connector B]
Join key(s): [Date AND Landing Page | Date AND Campaign | Session ID — choose lowest-cardinality key that's present in both]
Join type: [Left outer — default; Inner when you need exact overlap]
⚠ Gotcha: date-grain mismatch — both sources must aggregate to the same granularity before joining. GA4 date vs. Ads date: use ISO date string, not timestamp.
⚠ Gotcha: (not set) propagation — a left outer join will pull (not set) rows from the right source into the blend; add a filter to exclude them from ratio metrics.
```

---

## Step 4A — Spec Mode (default)

Produce a complete, build-ready blueprint in this structure:

### Dashboard spec: [Brand] — [Dashboard name]

**Overview**
- Purpose: [one sentence]
- Audience: [who reads it and at what cadence]
- Refresh: [connector schedule → report cache → expected data lag]

**Report pages** (one section per page)

For each page:
```
Page: [name]
Layout: [Grid description — e.g., 2-column: scorecard strip (top), time-series chart (left 60%), bar chart (right 40%), table (bottom full-width)]
Widgets:
  1. Scorecard tile — [Metric], comparison: prior period, target line: [value or [verify]], color: [brand hex]
  2. Time-series chart — [Metric] by [Date dimension], breakdown by [dimension], style: line, color sequence: [brand palette]
  3. [etc.]
```

**Calculated fields** (complete, ready to paste into Looker Studio formula editor)

```
Name: Engaged Session Rate
Formula: SUM(Engaged Sessions) / SUM(Sessions)
Format: Percent, 1 decimal
Scope: Session
Gotcha note: Engaged Sessions only available for GA4 web+app streams; Universal Analytics properties return 0 — verify stream type before using.
```

```
Name: Revenue per Session
Formula: SUM(Purchase Revenue) / SUM(Sessions)
Format: Currency (USD), 2 decimals
Scope: Session
Gotcha note: Purchase Revenue is event-scoped in GA4; dividing by session-scoped Sessions is a cross-scope calculation — Looker Studio will aggregate correctly but sampling can skew it at high row counts; set date range to avoid sampling threshold (>500k rows in exploration).
```

*(Add one block per calculated field)*

**Sharing and access**
```
Share setting: [View only — link | Editor — named users | Embedded — iFrame URL]
Refresh token owner: [service account recommended for unattended refresh — [verify org policy]]
Row-level security: [none | filter by viewer email — requires user attribute dimension]
```

Save the spec to `./dashboards/[brand-slug]-[dashboard-name]-spec.md`.

---

## Step 4B — Asset Mode

Triggered when: the user says "give me something shareable now," "I don't have Looker Studio access," or asks for an HTML dashboard.

Produce a single self-contained HTML file (`./dashboards/[brand-slug]-[dashboard-name].html`) using:
- Inline CSS — brand hex colors, font-stack from brand-brain
- Metric tiles with large-number display, period-over-period delta (green/red arrow), and threshold badge
- Chart.js (CDN, no build step) for time-series and bar charts
- A data table section with sortable columns
- A data-freshness footer (timestamp, source list, known limitations from the DQ gate)

The file opens in any browser, embeds in a Notion page, and attaches to Slack without external dependencies.

---

## The five Looker Studio gotchas every build must address

These map to the canonical GA4/measurement gotchas the data-quality gate checks — but they surface here specifically as build decisions, not just warnings:

1. **Sampling above the row threshold.** GA4 explorations sample above ~500k rows per date range. In Looker Studio, sampling kicks in when the connector uses the sampling API. Mitigation: use the GA4 Data API connector with `samplingLevel = UNSAMPLED` (available for GA4 properties on all tiers [verify current limits]); or aggregate in BigQuery first and connect via BigQuery connector.

2. **(not set) dimension pollution.** (not set) in Session default channel group or Landing page inflates totals and breaks blended-source joins. Mitigation: add a connector-level filter `Session default channel group != (not set)` and annotate any scorecard that excludes it.

3. **Cross-source date-grain mismatch in blends.** GA4 Date = `YYYYMMDD`; Google Ads Date = `MM/DD/YYYY` in some export formats. Always cast to ISO date string before joining; test with a 7-day date range and spot-check row counts.

4. **Session vs. event scope in calculated fields.** Mixing scopes (e.g., event count / sessions) is legal in Looker Studio but silently aggregates at different granularities. Name the scope explicitly in every calculated-field comment.

5. **Attribution window mismatch across sources.** GA4 default attribution: data-driven, 30-day lookback. Google Ads default: last-click, 30-day. A blended revenue chart will double-count or under-count unless you document which model each source uses. Add an attribution note to the dashboard header.

---

## Framework: the SPEC → BUILD → VERIFY loop

This skill follows the **Measurement Spec → Implementation → Verification** discipline from the Google Analytics 4 measurement methodology [verify current Google documentation]:

1. **Spec first.** Never start in the Looker Studio UI without a written spec. The spec is the source of truth; the UI is the execution surface.
2. **Build to the spec verbatim.** Every calculated field name in the UI must match the spec; every blend join key must match. Divergence creates maintenance debt.
3. **Verify against the source.** After build, spot-check three metric tiles against GA4 Explore or BigQuery query results for the same date range. Document the delta (should be < 1% for un-blended metrics; up to 5% for blended due to join cardinality [verify]).

---

## Principles

- **Gate on data quality first.** A chart built on a broken event is worse than no chart — it creates false confidence. The DQ gate is non-negotiable.
- **Calculated fields are the audit trail.** Every derived metric gets a name, formula, scope annotation, and gotcha note. No implicit math.
- **Blends need join-key discipline.** The join key must exist in both sources, at the same grain, with no null leakage. Always call this out.
- **Sampling is a design decision, not a footnote.** Choose unsampled connectors or pre-aggregated sources deliberately; don't discover it after the dashboard is live.
- **The spec outlives the dashboard.** Save it to `./dashboards/` so any analyst can rebuild from scratch without reverse-engineering the UI.
- **Brand palette on every dashboard.** Looker Studio's default colors are not brand colors. Apply palette from brand-brain; every chart's first series uses the primary brand hex.

## What Not to Do

- Don't start building before the DQ gate clears on all blocking issues.
- Don't blend more than two sources in a single blend without documenting the join chain — three-way blends in Looker Studio cascade and are fragile [verify current Looker Studio blend limits].
- Don't use session-scoped and event-scoped metrics in the same ratio without annotating the scope mismatch.
- Don't hardcode property IDs or account IDs in documentation — use placeholders and mark as `[verify]`; they change at property migrations.
- Don't share dashboards with edit access to stakeholders unless they're the designated dashboard owners — viewer links only for consumers.
- Don't call a dashboard "live" if the connector refresh is manual or > 24h stale.

## Quality Checklist (self-review before presenting)

- `brand-brain` called; brand palette and terminology applied to header and chart colors?
- `data-qa-measurement-gotcha-checker` called; all blockers surfaced and build gated until resolved; all warnings annotated in the spec?
- Each KPI has: brand-correct label, data source, scope annotation, blend strategy (if cross-source), target/threshold?
- All five Looker Studio gotchas addressed explicitly (sampling, (not set), date-grain, scope, attribution window)?
- Every calculated field has: name, formula, format, scope, gotcha note?
- Spec saved to `./dashboards/[brand-slug]-[dashboard-name]-spec.md`?
- Asset Mode (if used): HTML is self-contained, opens in browser without dependencies, includes data-freshness footer?
- Attribution note in dashboard header?
- Sharing settings documented; no edit access handed to non-owners?
