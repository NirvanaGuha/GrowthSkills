---
name: sql-query-generator-for-ga4-bigquery
description: >
  Plain-English analytics question + event schema → ready-to-run BigQuery SQL for funnels,
  reverse-ETL syncs, and A/B significance on the GA4 export. Translates what a growth marketer
  wants to know into correct, copy-paste BigQuery SQL targeting the GA4 `events_*` flat schema —
  without the user needing to know unnested arrays, event-param extraction, or BigQuery date-shard
  partitioning. Handles three primary query classes: funnel analysis, A/B test significance
  (frequentist), and reverse-ETL audience or cohort syncs. Also writes diagnostic queries for
  data-quality checks (missing events, duplicate users, broken session stitching). Flags GA4
  measurement gotchas before they corrupt a query — sampled data warnings, `(not set)` traps,
  cross-device identity caveats, and the difference between `user_pseudo_id` and `user_id`.
  Integrates with `data-qa-measurement-gotcha-checker` for pre-query data validation.
  Use when the user says "write me a BigQuery query," "GA4 BigQuery SQL," "query my GA4 export,"
  "funnel in BigQuery," "A/B significance in BigQuery," "build an audience from BigQuery,"
  "reverse-ETL from GA4," "how do I query event params," or hands over a plain-English analytics
  question and wants runnable SQL back.
---

# SQL Query Generator for GA4 / BigQuery

Plain-English analytics question in. Ready-to-run BigQuery SQL out.

The GA4 → BigQuery export is the most powerful raw source available to a growth team — and also the most unforgiving. The flat `events_*` schema, nested `event_params` arrays, shard-partitioned tables, and identity fragmentation trip up experienced analysts. This skill bridges the gap: you describe what you want to know, it writes correct SQL, flags the measurement traps relevant to your question, and explains how to validate the output before you act on it.

Three query classes covered: **funnel analysis**, **A/B experiment significance**, and **reverse-ETL audience / cohort syncs**. Also handles **data-quality diagnostics** (missing events, user-ID gaps, session stitching failures).

---

## Skills this calls

- **`brand-brain`** (always first) — loads the active brand's GA4 property ID, BigQuery project/dataset, known event taxonomy, and any brand-specific measurement gotchas stored in `brand.md`.
- **`data-qa-measurement-gotcha-checker`** — called when the user's question shows signs of known pitfalls (cross-device, paid attribution, session vs. user granularity conflicts, or they share a GA4 report that may be sampled). Pre-validates the data before queries run against it.
- **`experiment-results-analyzer`** — called if the user wants an interpretation narrative on top of the significance SQL output (p-value + lift storytelling for stakeholders).
- **`ltv-cac-payback-calculator`** — called when reverse-ETL cohort output feeds a CAC or payback calculation.
- **`a-b-multivariate-test-designer`** — called if the user needs to design the test before querying results (test brief → SQL is the right order).

---

## How a run works

```
Step 0  Load brand context    ──► brand-brain (GA4 property ID, BQ dataset, event taxonomy)
Step 1  Classify the request  ──► Funnel | A/B Significance | Reverse-ETL | Diagnostic
Step 2  Confirm schema inputs ──► event names + params, date range, identity field
Step 3  Check for gotchas     ──► flag before writing SQL, not after
Step 4  Write the SQL         ──► parameterized, with inline comments
Step 5  Output + save offer   ──► query + explanation + validation query + gotcha notes
```

### Step 0 — Load brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Retrieve the active brand's BigQuery project ID, dataset name (e.g. `analytics_313318114`), known event taxonomy (custom events defined by this brand), GA4 property ID, and any measurement quirks stored in `brand.md`. If the brand has no GA4/BQ config yet, ask for: BigQuery project ID, dataset name, and the key events relevant to the question. Do not write SQL until these are resolved.

---

## Query Class 1 — Funnel Analysis

**Framework: Sequential Event + Session Attribution**

GA4 funnels in the raw export require ordered-event logic within a session or user scope. Do not use `COUNTIF` across rows — use window functions (`ROW_NUMBER`, `LAG`, or existence flags via `MAX(IF(...))`) to enforce step ordering within a `user_pseudo_id + ga_session_id` window.

### Inputs required
| Input | Example |
|---|---|
| Funnel steps (event names in order) | `page_view → add_to_cart → begin_checkout → purchase` |
| Scope | Session (default) or User (cross-session) |
| Date range | Last 30 days / specific range |
| Segmentation dimension | `traffic_source.source`, `device.category`, experiment arm |

### Gotchas to flag automatically
- **Unnested params**: `event_params` is a repeated STRUCT; always `UNNEST` or use `(SELECT value.string_value FROM UNNEST(event_params) WHERE key = 'X')`.
- **Shard partitioning**: query `events_*` with `_TABLE_SUFFIX BETWEEN '20250101' AND '20251231'` — never `WHERE event_date`; it is a string field and won't prune partitions.
- **Session stitching**: GA4 `ga_session_id` is not globally unique — scope it as `CONCAT(user_pseudo_id, ga_session_id)`.
- **Duplicate events**: GA4 fires duplicates on SPA re-renders; deduplicate `page_view` by `(user_pseudo_id, event_timestamp, event_name)` before counting funnel entries.
- **`(not set)` sessions**: filter or bucket explicitly; leaving them in silently dilutes conversion rates.

### Output shape
```sql
-- GA4 Funnel: [Step 1] → [Step N]  |  Scope: Session  |  Date: YYYYMMDD–YYYYMMDD
-- Brand: [slug] · BQ: [project.dataset]
WITH
  raw_events AS (
    SELECT
      user_pseudo_id,
      CONCAT(user_pseudo_id,
             CAST((SELECT value.int_value FROM UNNEST(event_params)
                   WHERE key = 'ga_session_id') AS STRING)) AS session_key,
      event_name,
      event_timestamp
    FROM `[project.dataset].events_*`
    WHERE _TABLE_SUFFIX BETWEEN '[start]' AND '[end]'
      AND event_name IN ([step_list])
  ),
  session_flags AS (
    SELECT
      session_key,
      MAX(IF(event_name = '[step1]', 1, 0)) AS s1,
      MAX(IF(event_name = '[step2]', 1, 0)) AS s2,
      -- ... repeat per step
    FROM raw_events
    GROUP BY 1
  )
SELECT
  COUNT(*)                               AS entered,
  COUNTIF(s2 = 1)                        AS reached_step2,
  -- ...
  SAFE_DIVIDE(COUNTIF(s_final = 1),
              COUNT(*))                  AS overall_cvr
FROM session_flags
WHERE s1 = 1;
```

---

## Query Class 2 — A/B Experiment Significance

**Framework: Frequentist Chi-Square / Z-test in SQL (Welch's t-test for revenue)**

For binary outcomes (converted / did not convert), compute a two-tailed z-test for proportions directly in SQL. For revenue (continuous), use Welch's t-test via aggregated mean + variance. Surface p-value interpretation rules; do not declare significance for the user — return the p-value and let `experiment-results-analyzer` narrate.

### Inputs required
| Input | Example |
|---|---|
| Experiment arm event/param | `event_name = 'experiment_impression'`, param `experiment_id` = `'test_42'` |
| Conversion event | `purchase` |
| Metric type | Conversion rate (binary) or Revenue per user (continuous) |
| Date range | Experiment start to end |

### Gotchas to flag automatically
- **Sample-ratio mismatch (SRM)**: always include an SRM check query. If the actual arm split deviates >5% from the intended split, the result is invalid regardless of p-value.
- **Novelty effect**: flag if the experiment ran fewer than one full week (day-of-week confounding).
- **User-level, not session-level**: significance must be computed at `user_pseudo_id` level; session-level inflates N and produces false positives.
- **Multiple comparisons**: if querying multiple metrics, Bonferroni correction applies; note this.
- **Cross-device leakage**: if `user_id` is absent for >20% of experiment rows, arm assignment may be polluted.

### Output shape (binary CVR)
```sql
-- A/B Significance: [Experiment ID]  |  Metric: Conversion Rate
-- Brand: [slug] · BQ: [project.dataset]
WITH
  assignments AS (
    SELECT
      user_pseudo_id,
      (SELECT value.string_value FROM UNNEST(event_params)
       WHERE key = 'variant')                          AS arm,
      MIN(event_timestamp)                             AS first_exposure
    FROM `[project.dataset].events_*`
    WHERE _TABLE_SUFFIX BETWEEN '[start]' AND '[end]'
      AND event_name = 'experiment_impression'
      AND (SELECT value.string_value FROM UNNEST(event_params)
           WHERE key = 'experiment_id') = '[exp_id]'
    GROUP BY 1, 2
  ),
  conversions AS (
    SELECT DISTINCT user_pseudo_id
    FROM `[project.dataset].events_*`
    WHERE _TABLE_SUFFIX BETWEEN '[start]' AND '[end]'
      AND event_name = '[conversion_event]'
  ),
  arm_stats AS (
    SELECT
      arm,
      COUNT(DISTINCT a.user_pseudo_id)               AS users,
      COUNT(DISTINCT c.user_pseudo_id)               AS converters,
      SAFE_DIVIDE(COUNT(DISTINCT c.user_pseudo_id),
                  COUNT(DISTINCT a.user_pseudo_id))  AS cvr
    FROM assignments a
    LEFT JOIN conversions c USING (user_pseudo_id)
    GROUP BY 1
  ),
  -- Z-test for two proportions
  z_calc AS (
    SELECT
      MAX(IF(arm = 'control',  cvr, NULL))            AS p_ctrl,
      MAX(IF(arm = 'control',  users, NULL))          AS n_ctrl,
      MAX(IF(arm = 'variant',  cvr, NULL))            AS p_var,
      MAX(IF(arm = 'variant',  users, NULL))          AS n_var
    FROM arm_stats
  )
SELECT
  *,
  SAFE_DIVIDE(p_var - p_ctrl,
    SQRT(((p_ctrl * (1 - p_ctrl)) / n_ctrl)
       + ((p_var  * (1 - p_var))  / n_var))) AS z_score
  -- p-value lookup: |z| > 1.96 → p < 0.05 (two-tailed)
FROM z_calc;

-- ── SRM Check ───────────────────────────────────────────────────────────────
-- Expected 50/50 split; χ² > 3.84 → SRM detected → result unreliable
SELECT arm, COUNT(*) AS n,
  SAFE_DIVIDE(COUNT(*), SUM(COUNT(*)) OVER ()) AS actual_share
FROM assignments GROUP BY 1;
```

---

## Query Class 3 — Reverse-ETL Audience / Cohort Sync

**Framework: User-Level Cohort Export**

Produces a flat table of `user_pseudo_id` (and `user_id` when present) with behavioral attributes — ready to sync to a CRM, email ESP, or ad platform via reverse-ETL (Hightouch, Census, dbt, or direct BigQuery scheduled query).

### Inputs required
| Input | Example |
|---|---|
| Cohort definition | "Users who viewed pricing in last 7 days and did not purchase in last 30" |
| Output columns needed | `user_id`, `email`, `last_seen`, `product_viewed`, `session_count` |
| Identity preference | `user_id` (logged-in) / `user_pseudo_id` (anonymous) |
| Sync destination | CRM / ESP / Ads |

### Gotchas to flag automatically
- **`user_id` vs `user_pseudo_id`**: `user_id` is only populated when explicitly set via `setUserId()`; if the brand doesn't call this, `user_id` will be NULL for most rows. Confirm with `brand-brain` whether it is set.
- **Cross-device identity gap**: one `user_pseudo_id` per device; if the user logs in on a second device, there are two records. Only `user_id` bridges this — and only when populated.
- **PII in event params**: never `SELECT *` on `user_properties` for a sync; pull only the fields explicitly needed and confirm they don't contain raw PII that violates the destination platform's terms.
- **Freshness**: GA4 BigQuery export typically lags 24–48 hours; note this for time-sensitive audience syncs.

---

## Query Class 4 — Data-Quality Diagnostics

Before any business query runs, these diagnostic templates validate that the underlying data is trustworthy. Offer to run them first when the brand is new to BigQuery queries.

| Diagnostic | What it catches |
|---|---|
| Missing events | Events present in GA4 UI but absent in BQ (usually firing before BQ link was created) |
| Duplicate rows | SPA re-render duplication; `event_count` should match GA4 console ±2% |
| Session stitching gaps | % of sessions with `ga_session_id = NULL` |
| `user_id` population rate | % of events with a non-null `user_id` (signals identity coverage) |
| Event-param completeness | % of a key event that has the expected param populated (e.g. `purchase` without `value`) |

---

## Output format (all query classes)

Every response delivers:
1. **Gotcha brief** — 3–5 bullets on the traps specific to this question; shown before the SQL.
2. **Ready-to-run SQL** — parameterized (`[project.dataset]`, `[start]`, `[end]`), with inline comments on each non-obvious step.
3. **Validation query** — a second, simpler query to sanity-check the result (e.g., compare total conversions to GA4 UI for the same period; if they differ >5%, investigate before acting).
4. **Interpretation guidance** — one paragraph on what the numbers mean and what would make you distrust them.

Offer to save the query set to `./queries/[slug]-[question-slug].sql` and a companion `./queries/[slug]-[question-slug]-notes.md`.

---

## Principles

- **Brand-brain first.** No SQL before the BQ project + dataset + event taxonomy are confirmed. A query against the wrong dataset wastes compute and produces wrong answers with no error message.
- **Gotchas before SQL.** Always flag the measurement traps relevant to this question first. A technically correct query on corrupted data is worse than no query.
- **Scope explicitly.** Every query states whether it operates at user, session, or event granularity. Mixing silently inflates or deflates numbers.
- **Partition-prune by default.** Always use `_TABLE_SUFFIX BETWEEN` on `events_*` — never rely on the `event_date` string field for pruning. Unpartitioned scans can cost 10–100x more.
- **UNNEST correctly.** Never use `event_params.value.string_value` directly — always `(SELECT value.X FROM UNNEST(event_params) WHERE key = 'Y')`. The direct path returns NULL silently.
- **Validation is mandatory.** Every query output includes a validation step against GA4 UI totals. If they diverge >5%, raise a flag before the user acts on the data.
- **Truth only.** If the brand's event taxonomy is unknown, ask — do not guess event names. A query on a nonexistent event returns an empty table with no error.

## What Not to Do

- Don't write SQL before brand-brain returns the BQ project + dataset.
- Don't use `WHERE event_date = '2025-01-01'` for partition pruning — use `_TABLE_SUFFIX`.
- Don't reference `event_params.key` or `event_params.value` directly without UNNEST.
- Don't compute significance at session level — always user level.
- Don't declare statistical significance yourself — return the z-score / p-value and let the analyst or `experiment-results-analyzer` interpret it.
- Don't omit the SRM check on any A/B query.
- Don't SELECT unnecessary PII columns in reverse-ETL audience queries.
- Don't claim a result is accurate without including a validation query.

## Quality Checklist

- `brand-brain` called; BQ project, dataset, and relevant event names confirmed?
- Query class identified: Funnel / A/B Significance / Reverse-ETL / Diagnostic?
- Partition pruning via `_TABLE_SUFFIX BETWEEN`; no `WHERE event_date` for pruning?
- All `event_params` access via `UNNEST`; no direct field path?
- Session key scoped as `CONCAT(user_pseudo_id, ga_session_id)`, not bare `ga_session_id`?
- A/B query: user-level aggregation, SRM check included, no significance declaration?
- Gotcha brief present before SQL; validation query provided after?
- Save-to-file offer made for `./queries/`?
