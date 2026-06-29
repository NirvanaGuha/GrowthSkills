---
name: crm-data-hygiene-dedup-runner
description: >
  Takes a CRM contact or company export — CSV, pasted rows, or described schema — and returns
  a deduplicated, standardized, import-ready list alongside a structured audit: flagged duplicates
  with merge confidence scores, field-standardization changes, stale records (no activity over a
  configurable threshold), blank mandatory fields, and an unused-field report that surfaces columns
  nobody is writing to. Works against any CRM (HubSpot, Salesforce, Pipedrive, ActiveCampaign,
  custom) by adapting field logic to whatever schema is present. Applies four of the DAMA
  data-quality dimensions (completeness, uniqueness, validity, timeliness) as its scoring spine. Produces three
  output artifacts: a cleaned CSV ready to re-import, a hygiene audit report, and a field-usage
  memo. Data-quality gate runs through `data-qa-measurement-gotcha-checker` before any output is
  declared clean. Use when the user says "clean my CRM," "deduplicate contacts," "merge duplicates,"
  "standardize CRM fields," "find stale records," "CRM hygiene," "audit my contact list,"
  "unused CRM fields," or hands over a CSV/export and asks what's wrong with it.
---

# CRM Data Hygiene & Dedup Runner

Dirty CRM data compounds every downstream problem — bad segmentation, inflated unsubscribe rates, broken lead routing, double-outreach to the same account. This skill takes a raw export and returns three things: a **cleaned, import-ready file**, a **structured hygiene audit**, and a **field-usage memo**. It doesn't guess at your CRM's field schema — it reads what you give it and adapts.

The scoring spine is **four of the DAMA data-quality dimensions**: Completeness, Uniqueness, Validity, Timeliness. (DAMA-DMBOK defines six core dimensions in all — these four are the ones this runner scores against.) Every finding maps back to one of those four. That gives you a defensible quality score you can track over time and bring to a RevOps conversation.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's context (ICP, sales process, CRM conventions) so field-standardization rules, segment tags, and the stale-record threshold are calibrated to the actual business. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their CRM platform name, the mandatory fields for their sales process, and their definition of a "stale" record (days since last activity) before proceeding.
- **`data-qa-measurement-gotcha-checker`** (required, quality gate) — runs a structured data-quality gate over the cleaned output before it is declared ready to re-import. Any gotcha flagged here must be addressed or explicitly acknowledged before the artifacts are finalized.
- **`segmentation-rfm-strategy-builder`** (optional, compose) — if the user also wants segment assignments after the clean, pass the cleaned CSV to this skill to apply RFM or behavioral segmentation logic.
- **`lead-scoring-routing-model-designer`** (optional, compose) — if routing logic needs to be re-derived once the cleaned data reveals its true shape, compose here.
- **`tracking-plan-taxonomy-builder-auditor`** (optional, compose) — if the unused-field audit surfaces systemic data-capture gaps, hand the field-usage memo to this skill to redesign the event/field taxonomy upstream.

---

## How a run works

```
Step 0  Load brand context        ──► brand-brain (CRM conventions, ICP fields, stale threshold)
Step 1  Ingest + profile          ──► read schema, detect CRM platform, count rows/fields
Step 2  Deduplicate               ──► DAMA Uniqueness pass, merge-confidence scoring
Step 3  Standardize               ──► DAMA Validity pass, normalize fields to canonical form
Step 4  Flag completeness gaps    ──► DAMA Completeness pass, mandatory vs. optional fields
Step 5  Flag stale records        ──► DAMA Timeliness pass, configurable inactivity threshold
Step 6  Unused-field audit        ──► identify columns with >90% null/blank
Step 7  Data-quality gate         ──► data-qa-measurement-gotcha-checker
Step 8  Produce artifacts         ──► cleaned CSV + hygiene audit report + field-usage memo
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. The returned digest tells you the ICP definition (critical for deciding which fields are "mandatory"), the CRM platform in use (field name conventions differ materially between HubSpot, Salesforce, and Pipedrive), and any existing stale-record policy. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their CRM platform name, the mandatory fields for their sales process, and their definition of a "stale" record (days since last activity) before proceeding.

### Step 1 — Ingest and profile

Before touching any record, produce an ingestion summary:

```
File:       [filename or "pasted data"]
Rows:       [n contacts / n companies]
Fields:     [n columns]
Platform:   [detected or stated]
Date range: [earliest to latest activity/created date if present]
```

If the input is a schema description rather than live data, acknowledge this and adapt — produce a hygiene spec (rules to apply) rather than a cleaned CSV, and note that in the output header.

---

## DAMA Quality Dimensions — the four-pass framework

### Pass 1 — Uniqueness (deduplication)

Use a **weighted-field matching model** (not just email equality):

| Signal | Weight |
|---|---|
| Email (normalized) | High |
| Phone (E.164 normalized) | High |
| First + Last name (Jaro-Winkler ≥ 0.92) | Medium |
| Company name (normalized, token-sorted) | Medium |
| Domain extracted from email | Supporting |

**Merge confidence tiers:**

| Tier | Criteria | Action |
|---|---|---|
| Certain (≥95%) | Email match OR phone match + name match | Auto-merge; flag in audit log |
| Probable (80–94%) | Name + company + domain match; no email conflict | Flag for human review; suggest merge |
| Possible (60–79%) | 2 of 3 soft signals match | Flag as possible duplicate; do not auto-merge |

For each Certain merge, retain the record with the most recent activity date; promote non-null field values from the absorbed record (never silently overwrite a non-null value with a null). List every merge decision in `./crm-hygiene/dedup-log.csv` with columns: `record_id_kept`, `record_id_absorbed`, `confidence_tier`, `match_signals`, `fields_promoted`.

### Pass 2 — Validity (standardization)

Normalize to canonical form without altering substance:

- **Email:** lowercase; remove trailing spaces; flag malformed (no `@`, invalid TLD pattern)
- **Phone:** strip non-numeric, reformat to E.164 where country is determinable; flag ambiguous country codes
- **Name:** title-case; trim whitespace; flag all-caps or all-lowercase as probable scrape artifacts
- **Job title:** map obvious variants to a controlled vocabulary (e.g., "VP of Mktg" → "VP Marketing") only if the brand's `brand.md` contains a job-title taxonomy; otherwise flag, don't invent
- **Country / State:** ISO 3166 where unambiguous; flag free-text entries that don't match a known country name
- **Company name:** trim, de-duplicate legal-suffix variants (Inc/LLC/Ltd) only when the domain confirms identity; flag otherwise
- **Date fields:** normalize to ISO 8601 (YYYY-MM-DD); flag non-parseable strings

Every change is logged in `./crm-hygiene/standardization-log.csv`: `record_id`, `field`, `original_value`, `normalized_value`, `change_type`.

### Pass 3 — Completeness (mandatory-field gaps)

Classify fields as **mandatory**, **recommended**, or **optional** using:
1. The brand's ICP fields from `brand.md` (e.g., industry, company size, lifecycle stage)
2. Standard sales-process minimums: email OR phone, first name OR company name
3. Any segmentation fields active in the brand's ESP / CRM workflows

Score each record:

```
completeness_score = filled_mandatory_fields / total_mandatory_fields
```

Flag records with a score below 0.6 as **high-gap**; include the specific missing fields. Summarize at the file level: `% records with all mandatory fields`, `top 5 most-blank mandatory fields`.

### Pass 4 — Timeliness (stale records)

Default stale threshold: **no activity in 18 months** (configurable; use the brand's policy if present in `brand.md`, or ask).

"Activity" = the most recent of: last email open, last email click, last note/task, last deal stage change, last login, last CRM update — whichever fields are present in the export.

Flag stale records with a staleness tier:
- **Cold (18–36 months):** candidate for a re-engagement campaign (compose with `win-back-re-engagement-campaign-builder` if the user wants one)
- **Frozen (>36 months):** candidate for archival; surface to user before deletion

Do not auto-delete anything. Produce a stale-record list; the user decides.

---

## Unused-field audit

After all four passes, scan for columns where ≥90% of values are null, blank, or a single repeated default (e.g., "Unknown", "N/A", "—").

Produce a field-usage memo:

```
Field             | Fill rate | Verdict
------------------|-----------|--------
[field_name]      | 3%        | Unused — consider removing from import template
[field_name]      | 91%       | Well-used
...
```

Unused fields with 0% fill rate and no apparent purpose from field name → recommend removing from the CRM layout to reduce cognitive load on reps. If `tracking-plan-taxonomy-builder-auditor` is installed, note that it can redesign upstream capture to fill the gaps in important-but-empty fields.

---

## Data-quality gate

Before declaring artifacts ready, **invoke the `data-qa-measurement-gotcha-checker` skill**. Pass it:
- the cleaned CSV schema
- the dedup-log and standardization-log summaries
- the completeness and timeliness scores

If the gate flags issues (e.g., a merge that changed a field value without logging it, a normalization that looks like data loss, a stale-threshold miscalibration), resolve or explicitly acknowledge each flag before presenting the final artifacts.

---

## Output artifacts

All artifacts save to `./crm-hygiene/` relative to the user's CWD:

| File | Contents |
|---|---|
| `cleaned-contacts.csv` | Import-ready; deduped, standardized, all changes logged |
| `hygiene-audit-report.md` | DAMA scores per dimension, top findings, merge decisions summary, completeness gaps, stale breakdown |
| `field-usage-memo.md` | Fill-rate table per field, unused-field recommendations, upstream capture gaps |
| `dedup-log.csv` | Every merge decision with confidence tier and signals |
| `standardization-log.csv` | Every field normalization with before/after |

Present a **summary table** inline before listing the file paths:

```
DAMA Dimension   | Score  | Key Finding
-----------------|--------|-----------------------------------
Uniqueness       | [n]%   | [n] duplicates found; [n] auto-merged, [n] flagged for review
Validity         | [n]%   | [n] normalization changes; [n] malformed emails flagged
Completeness     | [n]%   | Top gap: [field] missing in [n]% of records
Timeliness       | [n]%   | [n] cold records; [n] frozen records
```

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No standardization rules applied until `brand-brain` returns, so mandatory fields and stale thresholds match the actual business — not generic defaults.
- **Log every change.** Nothing is silently altered. Every merge and every normalization is written to a log file. The user can audit and revert.
- **Never auto-delete.** Stale records are flagged and handed back with a recommendation; the user or RevOps owns the delete decision.
- **Never overwrite non-null with null.** When merging, a blank absorbs a value; a value never absorbs a blank.
- **Confidence tiers, not binary.** The world has near-duplicates. Probable and Possible matches go to a human review list; only Certain merges execute automatically.
- **DAMA is the spine.** Every finding maps to Completeness, Uniqueness, Validity, or Timeliness. This keeps the audit defensible and comparable over time.
- **Data-quality gate is mandatory.** `data-qa-measurement-gotcha-checker` runs before artifacts are declared ready. It is not optional.

## What Not to Do

- Don't delete records — flag and surface them.
- Don't overwrite a confirmed field value with a normalized variant without logging it.
- Don't invent a job-title taxonomy the brand hasn't confirmed; flag instead.
- Don't merge Possible duplicates automatically — the false-positive cost (merging two real people) is worse than the false-negative cost (one lingering duplicate).
- Don't produce a cleaned CSV before the data-quality gate passes (or all flags are explicitly acknowledged).
- Don't skip the unused-field audit — it's often the most valuable finding for RevOps.
- Don't reimplement brand resolution; call `brand-brain`.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and brand context loaded before any field rules were applied?
- Ingestion summary produced (row count, field count, platform)?
- All four DAMA passes completed and mapped to findings?
- Dedup: merge confidence tiered; Certain-only auto-merged; Probable/Possible in review list?
- Standardization: every change logged; no silent overwrites?
- Completeness: mandatory fields defined from brand context, not invented?
- Timeliness: stale threshold sourced from brand.md or confirmed with user?
- Unused-field audit completed with fill-rate table?
- `data-qa-measurement-gotcha-checker` gate run and any flags resolved or acknowledged?
- All five output artifacts written to `./crm-hygiene/`; inline DAMA summary table presented?
