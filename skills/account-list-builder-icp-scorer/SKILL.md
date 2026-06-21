---
name: account-list-builder-icp-scorer
description: >
  Turns a raw prospect or partner list into a clean, scored, Tier 1/2/3 account file ready for CRM
  import and downstream personalization. Accepts any combination of seed criteria (industry, headcount,
  ARR, tech stack, intent signals, geography), a pasted or uploaded CSV, and an ICP definition (loaded
  via brand-brain or supplied inline). Cleans the list — deduplicates, normalizes fields, flags
  missing critical data — then enriches each record with firmographic signals, tech-stack indicators,
  and buying-readiness proxies before scoring every account against a weighted ICP rubric and
  assigning it to Tier 1 (best-fit, highest priority), Tier 2 (partial fit, nurture-ready), or
  Tier 3 (weak fit, qualify-first or hold). The output CSV maps cleanly to standard CRM fields; the
  tiering rationale is documented so reps understand why an account ranks where it does. Use when the
  user says "build my target account list," "score these accounts," "clean up this prospect list,"
  "tier these companies for ABM," "import-ready list," "which accounts should we focus on," or hands
  over a CSV and asks which accounts matter.
---

# Account List Builder & ICP Scorer

Raw leads in, prioritized account tiers out. This skill cleans what you have, enriches what is missing, and scores every account against your real ICP so reps and ABM campaigns always hit the best targets first.

The score is only as good as the ICP definition. That definition lives in `brand-brain` — not here, not improvised per run. This skill builds the list; `brand-brain` supplies the target profile.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's ICP (firmographics, technographics, buyer persona, awareness tendency, deal profile). Do not start scoring before `brand-brain` returns. If absent, read `~/.brandbrain/brands/.active` and the brand's `brand.md` directly; if neither exists, run a 5-question inline ICP interview (segment · size/ARR · must-have tech · geography · business model) and document it as a temporary profile with `[unconfirmed]`.
- **`icp-persona-builder`** — call when the brand has no ICP section in `brand.md` or when the user asks to define or refresh the ICP before scoring.
- **`competitive-intelligence-dossier`** — call when the user wants to flag accounts currently using a named competitor (competitive displacement tier).
- **`account-dossier-builder`** — call after tiering to build a deep profile on individual Tier 1 accounts before outreach.
- **`cold-outreach-sequence-architect`** — the natural next step after a Tier 1 list is ready.

---

## How a run works

```
Step 0  Load brand + ICP  ──► brand-brain (always first)
Step 1  Ingest + clean    ──► normalize, deduplicate, flag missing fields
Step 2  Enrich            ──► firmographic + technographic + intent signals
Step 3  Score             ──► weighted ICP rubric → raw score 0–100
Step 4  Tier              ──► T1 / T2 / T3 assignment + rationale column
Step 5  Output            ──► CRM-ready CSV + tier summary table
```

---

## Step 0 — Load brand + ICP (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`) before touching any list. It returns the active brand's ICP: industry/vertical filters, firmographic bands (headcount, ARR, funding stage), required technographic signals, geography scope, deal type (self-serve vs. enterprise), and any known disqualifiers. These fields directly map to scoring dimensions below. **Do not score a single account until this returns.**

---

## Step 1 — Ingest and clean

Accept input in any form: pasted CSV, a table in chat, a column of company names, or a list of URLs.

**Normalization pass:**
- Standardize company names (remove "Inc.," "LLC," "Ltd." suffixes for deduplication; keep the canonical version).
- Deduplicate — same company/URL appearing more than once → merge, flag the duplicate count.
- Normalize column names to the standard schema (below).
- Flag rows missing critical fields: `company`, `website`, `industry`, or `headcount_range` — these need enrichment before scoring.

**Standard input schema (map incoming columns to these):**

| Field | Required | Notes |
|---|---|---|
| `company` | yes | Canonical company name |
| `website` | yes | Used as dedup key + enrichment anchor |
| `industry` | yes | Normalize to a consistent taxonomy |
| `headcount_range` | yes | Bands: 1–10, 11–50, 51–200, 201–500, 501–2000, 2000+ |
| `estimated_arr` | recommended | Use bands if exact ARR unknown |
| `hq_country` | recommended | For geo-filter |
| `tech_stack` | optional | Pipe-delimited; enriched in Step 2 |
| `funding_stage` | optional | Seed / Series A–D / PE-backed / Bootstrapped / Public |
| `intent_signal` | optional | G2 intent, job posting keywords, recent news flag |
| `source` | auto | Where the record came from |

---

## Step 2 — Enrich

For any missing critical field, use every available signal before marking a record as `[unconfirmed]`. Enrichment priority order:

1. **Website URL** → infer industry, HQ country, product type from the homepage and About page (use `WebFetch` if available).
2. **LinkedIn URL** (if in the CSV) → headcount band, recent funding announcements, job-posting signals.
3. **Tech stack proxies** → infer from job postings mentioning specific platforms (e.g., "Shopify Plus" in a job ad = eCommerce tech-stack signal). Mark inferred values `[inferred]`.
4. **Intent signals** → flag if a job posting, recent funding round, or press mention correlates with a buying event (hiring a VP of Marketing = growth signal; "migrating from X" = displacement signal).

**Never fabricate firmographic data.** If a field cannot be confirmed from any source, write `[unconfirmed]` and weight that dimension zero in scoring — do not guess.

---

## Step 3 — Score (weighted ICP rubric)

Score every account 0–100 using a weighted rubric derived from the brand's ICP. Default weights (adjust per brand if the ICP specifies different emphasis):

| Dimension | Default weight | Rationale |
|---|---|---|
| Industry / vertical fit | 30% | Wrong vertical = disqualifier |
| Headcount / company size | 20% | Proxy for deal size and complexity |
| Estimated ARR band | 20% | Willingness + ability to pay |
| Tech stack alignment | 15% | Integration readiness + displacement risk |
| Geography match | 10% | GTM coverage and support capacity |
| Intent / buying-readiness signals | 5% | Recency signals (hiring, funding, migration) |

**Scoring per dimension (0 / 0.5 / 1.0 multiplier applied to weight):**
- **1.0** — confirmed exact match to ICP criteria.
- **0.5** — partial match or `[inferred]` data.
- **0.0** — confirmed mismatch, `[unconfirmed]` data, or a known disqualifier (e.g., a banned vertical from the brand's ICP).

**Hard disqualifiers** (from the brand's ICP if present) immediately set the score to 0 and mark the account `T3-DQ` regardless of other dimensions.

**Document the raw score AND the per-dimension breakdown** — reps need to see why an account scored 72, not just that it did.

---

## Step 4 — Tier assignment

| Tier | Score range | Meaning | Default action |
|---|---|---|---|
| **T1 — Ideal** | 75–100 | Strong multi-dimensional ICP fit; prioritize now | Direct outreach + personalized dossier |
| **T2 — Qualified** | 45–74 | Partial fit; worth nurturing; one or two gaps | Nurture sequence; revisit on qualifying event |
| **T3 — Weak / Hold** | 0–44 | Significant mismatch or insufficient data | CRM hold; do not invest outreach budget |
| **T3-DQ** | Hard disqualifier | Confirmed out-of-scope | Remove or archive; note the reason |

Add a `tier_rationale` column to the output: one sentence naming the top reason for the tier and the biggest gap (e.g., "T2: vertical and size match but ARR band is below threshold; revisit post-Series A").

---

## Step 5 — Output

**Deliver two artifacts, saved to `./outreach/account-lists/`:**

1. **`[brand-slug]-account-list-[date].csv`** — CRM-ready, one row per account, fields:
   `company, website, industry, headcount_range, estimated_arr, hq_country, tech_stack, funding_stage, intent_signal, icp_score, tier, tier_rationale, data_confidence, source`

2. **`[brand-slug]-tier-summary-[date].md`** — the inline summary table:

```
## Account List: [brand] — [date]
ICP loaded from: brand.md (brand-brain)
Total accounts processed: N  |  T1: X  |  T2: Y  |  T3: Z  |  DQ: W

| Tier | Count | Avg ICP Score | Top Industry | Notes |
| T1   |       |               |              |       |
| T2   |       |               |              |       |
| T3   |       |               |              |       |

### Data quality flags
- N records missing headcount (marked [unconfirmed] — weight to 0)
- N records with inferred tech stack — confirm before outreach

### Suggested next step
T1 list → account-dossier-builder (deep profiles) → cold-outreach-sequence-architect
T2 list → usage-triggered-message-sequencer or lead-nurture-drip-builder on qualifying trigger
```

Always present the tier summary table inline in chat so the user sees the split immediately, even before the files are written.

---

## ICP interview (fallback — no brand-brain)

If `brand-brain` is unavailable and no `brand.md` exists, ask these five questions before scoring anything:

1. **Segment** — what industry/vertical and buyer title are you targeting?
2. **Size** — what headcount and ARR bands are ideal vs. too small vs. too large?
3. **Tech** — what 1–3 tools in their stack signal a good fit (or a red flag)?
4. **Geography** — any regions you cover today vs. out of scope?
5. **Disqualifiers** — any verticals, deal types, or funding stages you never touch?

Document the answers as a temporary ICP block at the top of the tier summary and flag it `[unconfirmed — install brand-brain to persist]`.

---

## Principles

- **Brand-brain first.** No scoring before the ICP loads. The scoring rubric is derived from the brand's ICP, not invented per run.
- **Confirmed data only.** Every `[unconfirmed]` or `[inferred]` field is weighted to zero; never score up to a guess.
- **Hard disqualifiers are hard.** A confirmed DQ criterion overrides every other positive signal. Don't let a 30% tech-stack match rescue a wrong-vertical account.
- **Rationale is mandatory.** A tier without a rationale column is useless to a rep. Always explain the score in one sentence per account.
- **Tier boundaries are defaults, not doctrine.** Document any threshold changes in the tier summary header so future runs stay consistent.
- **Enrich before abandoning.** Before marking a field `[unconfirmed]`, try every available signal (website, job postings, LinkedIn, press). Only give up when genuinely nothing confirms it.

---

## What not to do

- Do not score accounts before `brand-brain` returns the ICP.
- Do not invent headcount, ARR, or tech-stack data — use `[inferred]` or `[unconfirmed]` and reduce the weight.
- Do not create a flat list with no tiering — the job is to prioritize, not just to clean.
- Do not produce a tier without a rationale column — reps will ignore a list they cannot interpret.
- Do not write brand data to `brand.md` yourself — that is `brand-brain`'s job.
- Do not pass a T3 list to `cold-outreach-sequence-architect` — flag it as a hold and say why.
- Do not conflate "many accounts" with "many Tier 1 accounts" — a small T1 list is a better output than an inflated one.

---

## Quality checklist

- `brand-brain` called and ICP loaded (or fallback interview completed) before any scoring?
- Input list normalized: company names deduplicated, columns mapped to standard schema?
- Missing critical fields enriched from available signals; remaining gaps marked `[unconfirmed]`?
- Score is 0–100 with per-dimension breakdown documented (not just a final number)?
- Hard disqualifiers applied before weighting — DQ accounts marked T3-DQ regardless of partial positive signals?
- Tier rationale column present for every account (one-sentence explanation of the tier + biggest gap)?
- Output CSV maps cleanly to CRM import fields; tier summary table presented inline in chat?
- Files saved to `./outreach/account-lists/` with brand slug + date in filename?
- Next-step suggestions offered: T1 → `account-dossier-builder`, T2 → nurture, T3 → hold?
