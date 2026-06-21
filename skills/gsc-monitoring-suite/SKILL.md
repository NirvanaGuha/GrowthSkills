---
name: gsc-monitoring-suite
description: >
  Turns a raw Google Search Console export — or a live GSC connection — into a structured
  weekly SEO operations digest. Surfaces the five signals that matter most: top-mover queries
  (rank rises and falls), index-coverage issues and their triage priority, manual action or
  security alerts, low-CTR opportunity queries where rank is solid but clicks aren't following,
  and URL Inspection anomalies that need human action. Built around the GSC Signal Stack
  framework: five distinct signal types, each with a defined threshold, a root-cause tree, and
  a concrete next action — so a junior analyst produces the same output an experienced SEO
  director would hand to their team on Monday morning. Brand-aware through brand-brain: voice
  and product context shape how findings are framed in stakeholder language. Saves a dated
  digest and an action-item CSV to ./gsc/ so weekly runs accumulate into a traceable history.
  Use when the user says "GSC digest," "rank movers," "index coverage issues," "manual actions,"
  "low CTR queries," "URL inspection," "weekly SEO report," "what's happening in Search Console,"
  "GSC monitoring," or hands over a GSC export and asks what to do with it.
---

# GSC Monitoring Suite

A weekly SEO operations digest built on the GSC Signal Stack framework. Five signal types, five
root-cause trees, five classes of concrete next action — so monitoring is systematic, not
scroll-and-hope.

This skill reads, triages, and acts on GSC data. It does not crawl pages, generate new content,
or run a full technical audit. Those jobs belong to `technical-seo-audit-fix-prioritizer` and
`on-page-seo-optimizer` respectively — called below when findings warrant it.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's context so all findings are framed
  in brand language and product positioning. Also provides the canonical domain to filter
  against when the export includes multiple properties. **Fallback if `brand-brain` is absent
  or returns no brand:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly;
  if none exists, ask the user for the canonical domain (and voice) before proceeding — the
  domain filter the Signal Stack depends on must never be left undefined.
- **`technical-seo-audit-fix-prioritizer`** (optional, on trigger) — escalated when coverage
  issues reveal crawl or redirect problems beyond a single URL fix.
- **`on-page-seo-optimizer`** (optional, on trigger) — called per-URL when a low-CTR query
  reveals a title/meta mismatch worth fixing in the same run.
- **`content-decay-refresh-sweep`** (optional, on trigger) — called when a rank-drop cluster
  signals content staleness rather than a technical cause.
- **`internal-linking-planner`** (optional, on trigger) — called when a newly-risen query
  lacks internal links from supporting pages.

---

## How a run works

```
Step 0  Load the brand          ──► call brand-brain (domain filter, brand language)
Step 1  Ingest data             ──► parse or request GSC export(s); confirm date range
Step 2  Run the Signal Stack    ──► five signal types in parallel; apply thresholds
Step 3  Triage & root-cause     ──► each flagged item gets a root-cause hypothesis + next action
Step 4  Compose the digest      ──► stakeholder-ready narrative + action-item table
Step 5  Save artifacts          ──► ./gsc/YYYY-WW-digest.md + ./gsc/YYYY-WW-actions.csv
Step 6  Cascade (conditional)   ──► escalate deep crawl/content/linking items to sibling skills
```

---

## The GSC Signal Stack — five signals, five root-cause trees

### Signal 1 — Rank Movers (Queries)

**What to export:** Performance report → Queries → last 7 days vs. prior 7 days, sorted by
position delta. Include: query, URL, clicks, impressions, CTR, position (both periods).

**Thresholds that trigger a flag:**
- Rise: average position improved ≥ 3 spots, ≥ 100 impressions/week
- Drop: average position degraded ≥ 3 spots, ≥ 100 impressions/week, or any top-20 query
  drops ≥ 5 spots

**Root-cause tree for drops:**

```
Drop confirmed?
├── URL still indexed? (Signal 5 — URL Inspection)
│   └── No → canonicalization / noindex / redirect error → fix first
├── Page changed in period? (CMS change log, Git diff, Wayback Machine)
│   ├── Yes → content/UX change caused drop → revert or optimize
│   └── No → competitor movement or algorithm update
├── Competitor jumped above? (manual SERP check)
│   └── Yes → content gap → call content-decay-refresh-sweep
└── All SERP positions dropped together? → likely algo volatility → monitor one more week
```

**Root-cause tree for rises:**

```
Rise confirmed?
├── Internal links recently added? → confirm; if not, call internal-linking-planner to consolidate
├── Backlinks acquired? (third-party tool check [verify])
└── Content refreshed? → note in digest as a playbook-confirmed win
```

**Output for this signal:** ranked table (query | URL | position this week | prior week | delta |
clicks delta | root-cause hypothesis | next action).

---

### Signal 2 — Index Coverage Issues

**What to export:** Index → Pages (Coverage) → full status breakdown. Group by reason.

**Severity tiers and response SLAs:**

| GSC Status | Severity | Response |
|---|---|---|
| Server error (5xx) | P0 — fix today | Check hosting, CDN, server logs |
| Redirect error | P0 — fix today | Audit redirect chain; max 1 hop |
| Submitted URL blocked by robots.txt | P1 — fix this week | Verify robots.txt intent; unblock if accidental |
| Submitted URL not found (404) | P1 — fix this week | Restore or 301 to closest live equivalent |
| Duplicate, Google chose different canonical | P2 — review | Confirm the chosen canonical is correct; strengthen if not |
| Crawled — currently not indexed | P2 — investigate | Thin content? Low PageRank? Consolidate or improve |
| Discovered — currently not indexed | P3 — monitor | Crawl budget issue or low priority; build internal links |
| Excluded by noindex | Expected | Confirm intentional; skip unless count changed |

**Key monitoring pattern — week-over-week delta:**
Flag any status whose count changed by ≥ 10% OR ≥ 50 pages compared to prior week. Stable
counts, even large ones, warrant a note not an alarm.

**Root-cause escalation:** If P0/P1 count > 20 pages, call `technical-seo-audit-fix-prioritizer`
with the list; don't try to diagnose a crawl-budget or redirect-loop problem inside this skill.

---

### Signal 3 — Manual Actions & Security Issues

**What to export:** Security & Manual Actions → Manual Actions tab + Security Issues tab.

**Protocol:**

```
Any active manual action?
├── YES → P0: surface the full action text verbatim in the digest; link the Recovery Guide [verify URL]
│         → Stop recommending content changes until resolved; the action suppresses everything else
└── NO  → confirm clean; note last checked date

Any security issue (hacked content / malware / deceptive pages)?
├── YES → P0: quarantine the affected URLs; escalate outside this skill
└── NO  → confirm clean
```

This signal runs first in the parallel stack; if a P0 fires here, all other signals are
secondary in the digest.

---

### Signal 4 — Low-CTR Opportunity Queries

**Definition:** queries where average position ≤ 20 AND CTR is ≤ 50 % of the expected
CTR benchmark for that position (benchmarks below). These are ranking wins leaking traffic.

**Position → expected CTR benchmarks** (organic, desktop+mobile blended — these are
indicative industry averages; [verify] against current Sistrix/AWR data for your vertical):

| Position | Expected CTR |
|---|---|
| 1 | ~28% |
| 2 | ~15% |
| 3 | ~11% |
| 4–5 | ~7–8% |
| 6–10 | ~3–5% |
| 11–20 | ~1–2% |

**Root-cause tree:**

```
CTR below threshold?
├── Check title tag: does it contain the exact query keyword?
│   └── No → add keyword near the front of the title (50–60 chars)
├── Check meta description: does it give a specific reason to click?
│   └── No → rewrite with the query's implied job-to-be-done
├── SERP feature eating clicks? (featured snippet, PAA, map pack)
│   └── Yes → note this in digest; optimize for snippet capture or accept lower CTR
├── Brand vs. non-brand? (non-brand CTR norms are lower)
│   └── Non-brand position 1 <20% may still be fine → recalibrate threshold
└── Title truncated in SERP? (> ~60 chars)
    └── Yes → shorten title; front-load keyword + value prop
```

**Per-URL fix:** for each flagged URL, call `on-page-seo-optimizer` inline if ≤ 3 URLs; if
more, note in the action CSV and batch separately.

---

### Signal 5 — URL Inspection Triage

**What to export:** URL Inspection API (or manual via GSC URL Inspection tool) for:
(a) any URL that appeared in Signal 1 drops, (b) any P0/P1 coverage URL from Signal 2,
(c) any URL the user flags explicitly.

**Read the inspection result for:**

| Field | What to check |
|---|---|
| Indexing allowed | Yes/No — if No, find the blocking rule |
| Canonical | GSC-selected vs. user-declared — mismatch = consolidation problem |
| Mobile usability | Any issues → pass to `technical-seo-audit-fix-prioritizer` |
| Last crawl date | > 30 days + important page = crawl budget or internal link deficit |
| Enhancements | Breadcrumbs / sitelinks / rich results valid or invalid → fix schema |
| Referring sitelinks | None on an important page → call `internal-linking-planner` |

**Output:** inspection table (URL | indexed? | canonical conflict? | last crawl | mobile OK?
| rich results status | recommended action).

---

## Digest format

```markdown
# GSC Weekly Digest — [brand] — Week [YYYY-WW] ([date range])
**Property:** [verified GSC property URL]  |  **Brand:** [slug, via brand-brain]

---
## 0. Alerts (act first)
[Manual actions / security issues — blank if clean]

## 1. Rank Movers
### Top Rises
[table: query | URL | Δpos | clicks delta | hypothesis]
### Top Drops (flagged)
[table + root-cause + recommended action]

## 2. Coverage Issues
**Delta from prior week:** [+X new errors / -Y resolved]
[table by severity tier — P0/P1 only in the digest; P2/P3 in the CSV]

## 3. Low-CTR Opportunities
[table: query | URL | position | actual CTR | expected CTR | Δ | recommended fix]

## 4. URL Inspection Notes
[table for flagged URLs only]

## 5. Recommended Actions (ranked by impact)
[numbered list, max 10, owner + effort estimate where possible]

---
*Data range: [start] – [end] | Generated: [date] | Next run: [date + 7 days]*
```

---

## Artifact persistence

Save after every run, never overwrite:

```
./gsc/
  YYYY-WW-digest.md          (full stakeholder digest)
  YYYY-WW-actions.csv        (headers: rank | signal | query_or_url | issue | action | owner | effort | status)
  YYYY-WW-raw-movers.csv     (full mover table before threshold filtering — for ad hoc analysis)
```

If `./gsc/` doesn't exist, create it and note this is the first run baseline (prior-week
deltas will be available from run 2 onward).

---

## Cascade logic — when to call sibling skills

| Trigger | Skill to call |
|---|---|
| P0/P1 coverage count > 20 pages | `technical-seo-audit-fix-prioritizer` — pass coverage export |
| Rank-drop cluster on 3+ URLs, content age > 12 months | `content-decay-refresh-sweep` — pass URLs + query data |
| Newly-risen query, 0 internal links from supporting pages | `internal-linking-planner` — pass new URL + top-traffic supporting pages |
| Low-CTR URL with clear title/meta fix, user says fix now | `on-page-seo-optimizer` — pass URL + target query |

Call cascades at the end of the run, after the digest is composed. Don't delay the digest
output to wait for cascade results; flag them as "additional tasks queued."

---

## Principles (Non-Negotiable)

- **Brand-brain first.** Domain filter and brand language come from the brain. Never assume a
  domain; confirm it.
- **Signal Stack is the skeleton.** Run all five signals every time — don't skip signals
  because the export seems clean.
- **Deltas over absolutes.** A stable 500-error count is less alarming than a 404 count that
  jumped 50 pages this week. Always report change, not just state.
- **Root-cause before action.** Never recommend an action without a hypothesis. "Rankings
  dropped — add more content" is not a root-cause; it is a guess.
- **Real data only.** Numbers from the export, or `[verify]`. Never invent rank positions,
  CTR figures, or impression counts.
- **SLA discipline.** P0 in the alerts section, every time; it should never be buried.

## What Not to Do

- Don't run a full site crawl — that's `technical-seo-audit-fix-prioritizer`.
- Don't write new content — that's `blog-post-drafting-engine` or `content-refresh-briefer`.
- Don't re-implement brand resolution — call `brand-brain`.
- Don't skip the coverage delta check because "nothing looks wrong" — a quiet coverage sheet
  that grew 200 'Crawled – currently not indexed' pages is a problem.
- Don't report all 500 low-CTR queries — use the threshold filter and cap the digest at the
  top 10 by impressions-at-risk.
- Don't merge multiple GSC properties into one digest without clearly labeling which data
  came from which property.

## Quality Checklist (self-review before delivering digest)

- `brand-brain` called; active brand domain confirmed; findings framed in brand language?
- All five Signal Stack layers run, even if some returned clean?
- Every flagged item has a root-cause hypothesis and a concrete next action (not "investigate further")?
- P0 items (manual actions, security issues, 5xx errors) appear in Section 0 / flagged first?
- Position deltas use a defined threshold (≥ 3 spots, ≥ 100 impressions) — not just any movement?
- Low-CTR analysis uses the benchmark table, not gut feel?
- Digest saved to `./gsc/YYYY-WW-digest.md`; action CSV saved to `./gsc/YYYY-WW-actions.csv`?
- Cascade calls noted at the end (not blocking the digest output)?
- No invented numbers; all unconfirmed benchmarks marked `[verify]`?
