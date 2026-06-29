---
name: google-sheets-excel-power-tools-builder
description: >
  Sheet task description → working VLOOKUP/INDEX-MATCH/SUMIFS/QUERY formula, pivot table spec,
  or Google Apps Script / Excel VBA automation for any marketer living in spreadsheets. Three
  modes: Formula (instant, correct, paste-ready formula with explanation), Pivot (buildable pivot
  spec with exact field placement and calculated fields), and Script (Apps Script or VBA that
  actually runs — not pseudocode). Covers every recurring marketer pain: attribution lookups,
  multi-condition aggregations, dynamic dashboards, campaign-data cleansing, scheduled email
  reports, and budget vs. actuals reconciliation. Outputs are live-ready artifacts: paste the
  formula, follow the pivot spec step-by-step, or drop in the script. Never advice-only.
  Brand context is loaded first so outputs match the brand's sheet conventions, naming, and
  data schema. Use whenever the user says "write me a formula," "VLOOKUP/XLOOKUP this," "how do
  I SUMIFS," "build a pivot," "automate this sheet," "Apps Script to," "VBA to," "how do I
  pull data from," "clean up this column," "send a report from Sheets," or describes any
  repetitive spreadsheet task.
---

# Google Sheets & Excel Power Tools Builder

Sheet task in → working artifact out. Formula, pivot spec, or automation script — paste-ready, not advice-only. Brand context loads first so column names, naming conventions, and data schema match the real environment. This skill never describes what a formula does in the abstract; it writes the exact formula for the stated data layout.

This is an L21 Growth Engineering skill. It produces real output that runs, not guidance on what to Google.

---

## Skills this calls

- **`brand-brain`** (required first) — loads the active brand's context: sheet naming conventions, data schema fields, known tool integrations (e.g., HubSpot, GA4 BigQuery exports), and any banned column-name aliases. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their sheet's column headers, data source, and what result cell or output they need before proceeding.
- **`data-qa-measurement-gotcha-checker`** — called as a gate before finalizing any formula or script that aggregates revenue, attribution, or funnel metrics; flags common gotchas (blank-vs-zero, duplicate rows, timezone mismatches, filtered-range drift). Call inline; present its flags as a "Watch out" section in the output.
- **`looker-studio-live-dashboard-builder`** — compose when the user needs a live connected dashboard downstream of the sheet work (Sheets → Looker Studio pipeline).
- **`tracking-plan-taxonomy-builder-auditor`** — compose when the sheet is a UTM/event taxonomy tracker that needs governance rules.
- **`utm-parameter-bulk-builder`** — compose when the sheet task involves building or validating UTM URLs at scale.

---

## How a run works

```
Step 0  Load the brand          ──► brand-brain (voice, schema, conventions)
Step 1  Classify the task       ──► Formula | Pivot | Script
Step 2  Gather the minimum      ──► ask only what's missing for the task type
Step 3  Build the artifact      ──► working output using the LEGO framework
Step 4  Data-quality gate       ──► data-qa-measurement-gotcha-checker for metric aggregations
Step 5  Present + save          ──► inline + offer to save to ./sheets/[brand-slug]-[task].md
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Use the returned digest for: sheet naming conventions (e.g., "campaign name" vs. "Campaign Name" vs. "campaign_name"), known data sources (HubSpot exports, GA4 BigQuery, Shopify orders), and any field-alias conflicts already documented.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their sheet's column headers, data source, and what result cell or output they need before proceeding.

### Step 1 — Classify the task

| Mode | Trigger signals |
|---|---|
| **Formula** | "formula," "VLOOKUP," "SUMIFS," "pull," "calculate," specific function mentioned |
| **Pivot** | "pivot," "summarize by," "group by," "cross-tab," "breakdown" |
| **Script** | "automate," "Apps Script," "VBA," "macro," "scheduled," "send email from," "loop through rows" |

Default to Formula when ambiguous. Offer the others at the end.

### Step 2 — Gather the minimum (ask before building)

**Formula needs:** source sheet/tab name, relevant column letters or headers, target cell, exact result wanted, any conditions. Do not ask for data the brand-brain already supplied.

**Pivot needs:** source range or named table, rows/columns/values/filters desired, any calculated field logic.

**Script needs:** trigger (time-driven? button? edit event?), input range, output target (email address, new sheet, Slack webhook URL), and the loop logic (what decision does the script make per row?).

Ask all missing inputs in one batch. Never ask for something deducible from the brand's schema.

---

## LEGO build order (our working model for formulas/scripts)

LEGO = **Layer, Exact-reference, Gate, Output.** Every artifact follows this build order:

1. **Layer** — decompose the task into the smallest independent functions; build each layer before nesting.
2. **Exact-reference** — use explicit sheet names, locked `$` references, and named ranges where available; never use vague `A:A` column references in production formulas.
3. **Gate** — wrap with IFERROR / IFNA; for scripts, add a try/catch and a dry-run flag.
4. **Output** — present the final formula/spec/script in a code block, then explain what each argument does in one line each. Never flip the order (explanation before the formula leads to copy errors).

---

## Formula mode

Produce formulas for Google Sheets (default) or Excel (when specified); flag incompatibilities.

**Canonical hierarchy by task:**

| Task | Preferred function | Why |
|---|---|---|
| Single-condition lookup | `XLOOKUP` (Sheets/365) or `INDEX/MATCH` | VLOOKUP breaks on column inserts; use only when user is locked to an older format |
| Multi-column lookup | `INDEX/MATCH/MATCH` | |
| Conditional aggregation | `SUMIFS` / `COUNTIFS` / `AVERAGEIFS` | |
| Cross-sheet filtered aggregation | `QUERY` (Sheets) / `SUMPRODUCT` (Excel) | QUERY is SQL-like; document the syntax |
| Dynamic array / dedup | `UNIQUE` + `SORT` + `FILTER` (Sheets) / `UNIQUE` (365) | |
| Text parsing | `REGEXEXTRACT` (Sheets) / `TEXTSPLIT` (365) | |
| Date bucketing | `EOMONTH`, `WEEKNUM`, `TEXT(date,"YYYY-MM")` | |
| Attribution window | `SUMPRODUCT` with date-range array | |

**Output format:**

```
=YOUR_FORMULA_HERE
```

Arguments:
- `arg1` — what it references
- `arg2` — the condition / match value
- `arg3` — what it returns

Watch out: [data-qa flags, if any]

Paste this into [cell reference]. Named range version (if applicable): `=NAMED_RANGE_VARIANT`

---

## Pivot mode

Deliver a buildable spec — not a screenshot description. The user should be able to follow it step-by-step without guessing.

```
## Pivot spec — [task description]
Source range: [Sheet!A1:G500 or named table]
Insert → Pivot Table → [same sheet / new sheet: specify]

Rows:    [field name] — [sort: descending by Values]
Columns: [field name or "none"]
Values:  [field name] — [SUM / COUNTA / AVERAGE] — Show as: [% of row / grand total / default]
Filters: [field name] = [condition]

Calculated field (if needed):
  Name: [display name]
  Formula: = [field A] / [field B]   ← Sheets: use field names without $; Excel: use field names in quotes

Conditional format: Values column → Color scale [low=red, mid=yellow, high=green]
```

Always include the slicer recommendation when multiple pivots share the same source.

---

## Script mode

**Google Apps Script (default) or Excel VBA (when specified).**

Every script ships with:
1. A `DRY_RUN` constant at the top (`true` by default) — flipping it to `false` activates writes/sends.
2. A `try/catch` (Apps Script) or `On Error GoTo` (VBA) block.
3. Inline comments on every non-obvious line.
4. Trigger setup instructions (time-driven: Edit → Apps Script → Triggers; button: Insert → Drawing → Assign script).

**Common marketer automation patterns (handle without asking for spec details if the intent is clear):**

| Task | Pattern |
|---|---|
| Scheduled email digest from a sheet | `MailApp.sendEmail` + `SpreadsheetApp.getActiveSheet().getRange()` → body string |
| Row-by-row CRM enrichment | Loop rows, call `UrlFetchApp.fetch(apiEndpoint)`, write result to column |
| Duplicate removal + sort | `removeDuplicates()` on a named range + `sort()` |
| Auto-populate date stamp on edit | `onEdit(e)` trigger, check column, write `new Date()` |
| Budget vs. actuals alert | Sum a range, compare threshold, `MailApp.sendEmail` if exceeded |
| Import JSON API response to sheet | `UrlFetchApp.fetch` → `JSON.parse` → `setValues` |

**Output format:**

```javascript
// Google Apps Script — [task name]
// Trigger: [Time-driven daily 8am / onEdit / Button]
// Dry-run mode: set DRY_RUN = false to activate

const DRY_RUN = true;

function runTask() {
  try {
    // ... working code with inline comments
  } catch (e) {
    Logger.log('Error: ' + e.message);
  }
}
```

After the code block: setup steps (where to paste, how to set the trigger, how to test in dry-run).

---

## Principles (Non-Negotiable)

- **Working output only.** Never deliver pseudocode, "the logic would be…", or a formula skeleton. If you need more info, ask — then build.
- **LEGO order every time.** Layer → Exact-reference → Gate → Output. Never nest before validating the inner layer.
- **Exact references, not vague columns.** Always use `'Sheet Name'!$A$2:$A` or a named range. `A:A` is a beginner error that silently breaks on row insertions.
- **XLOOKUP / INDEX-MATCH over VLOOKUP.** VLOOKUP is fragile; use it only when the user is locked to a format that doesn't support the alternatives.
- **DRY_RUN by default.** Scripts that write, send, or delete must ship with dry-run mode on.
- **Data-quality gate for metric aggregations.** Always run `data-qa-measurement-gotcha-checker` before finalizing any formula or script that touches revenue, attribution, or funnel counts.
- **Brand schema first.** Column names and sheet tab names from the brand digest override any guesses.

## What Not to Do

- Don't explain what VLOOKUP is in general — write the formula for the specific data layout.
- Don't deliver a script without a DRY_RUN flag and error handling.
- Don't use `.getRange("A:A")` open-ended column references in production scripts — they scan the entire sheet and time out on large data.
- Don't invent column names; if the schema is unknown after brand-brain, ask.
- Don't output a pivot "description" — output a buildable spec with exact field placements.
- Don't skip the data-quality gate for aggregation formulas touching revenue or funnel metrics.
- Don't produce Excel output when the user is on Sheets (and vice versa) — flag function incompatibilities explicitly.

## Quality Checklist (self-review before presenting)

- `brand-brain` called; column names and schema from the digest used (not invented)?
- Fallback path present for brand-brain absence?
- Artifact classified correctly (Formula / Pivot / Script)?
- LEGO order followed: layers validated before nesting?
- Formula: XLOOKUP/INDEX-MATCH preferred; `$`-locked references; IFERROR wrapper; argument explanation provided?
- Pivot: source range, row/column/value/filter fields, and calculated-field formula all specified?
- Script: DRY_RUN constant present; try/catch present; trigger setup instructions included?
- `data-qa-measurement-gotcha-checker` called for any metric aggregation; flags surfaced in "Watch out"?
- Output is paste-ready — no placeholders left that require the user to reverse-engineer intent?
