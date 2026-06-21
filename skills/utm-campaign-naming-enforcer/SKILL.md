---
name: utm-campaign-naming-enforcer
description: >
  Audits UTM parameters and campaign naming conventions across any input format — raw UTM CSV, active
  campaign lists, ad platform exports (Google Ads, Meta, LinkedIn), or a pasted URL batch — and returns
  a flagged-violations table with corrected values plus a clean, validated UTM URL sheet ready to
  activate. Enforces a single naming schema (field order, case, separator, allowed values, required
  fields) so attribution never breaks because a trafficker typed "Email" instead of "email" or used a
  space instead of an underscore. Also generates a governance artifact: a locked naming taxonomy
  the team can reference on every campaign build. Calls `brand-brain` for brand slug + channel list;
  calls `data-qa-measurement-gotcha-checker` as a data-quality gate; calls `tracking-plan-taxonomy-
  builder-auditor` when a full taxonomy needs to be built or compared. Use whenever the user says
  "audit my UTMs," "clean up campaign names," "enforce UTM naming," "my UTM data is messy," "build a
  UTM naming convention," "UTM taxonomy," "campaign naming schema," "validate UTM URLs," "fix UTM
  parameters," or pastes a URL or CSV and asks why attribution is broken.
---

# UTM & Campaign Naming Enforcer

Dirty UTMs break attribution silently — "Email" and "email" look the same in a dashboard until you try to aggregate them and find two rows. This skill audits every parameter in a batch, flags each violation with its rule, emits a corrected value, and outputs a validated URL sheet. It also gives you the governance artifact — a canonical naming taxonomy — so the next trafficker doesn't invent a new convention.

The underlying framework is Google Analytics' own UTM taxonomy best-practice layer: five required/optional parameters, a strict field-value contract, and a naming schema aligned to GA4's channel-grouping rules so your traffic doesn't land in "(other)."

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand slug (used as the campaign prefix convention), confirmed channel list, and any brand-specific naming rules already captured. Does not reimplement brand resolution or scanning.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for the brand slug, their canonical channel list (e.g. email, paid_search, organic_social), and any existing naming rules before proceeding.
- **`data-qa-measurement-gotcha-checker`** (required) — runs as a data-quality gate on the corrected UTM sheet before output, catching common GA4 measurement gotchas (self-referrals, cross-domain leakage, inconsistent medium values that break channel grouping).
- **`tracking-plan-taxonomy-builder-auditor`** (called when building or comparing a full taxonomy) — if the user has no existing convention or asks for a full audit against their tracking plan, invoke this to surface the event taxonomy alongside the UTM schema so they stay aligned.

---

## How a run works

```
Step 0  Load the brand        ──► call brand-brain
Step 1  Ingest the input      ──► URL batch | CSV | ad export | "build from scratch"
Step 2  Detect or define the schema
Step 3  Audit every row       ──► violations table
Step 4  Emit corrected values ──► validated UTM sheet
Step 5  Data-quality gate     ──► call data-qa-measurement-gotcha-checker
Step 6  Governance artifact   ──► naming taxonomy + lock doc (on request or new build)
```

---

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`) with the user's request and any named brand. It returns the active brand slug, confirmed channel list, and brand-specific naming rules. Use the slug as the default campaign-name prefix (`[slug]_[quarter]_[initiative]`). Do not proceed until brand-brain returns.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for the brand slug, canonical channel list, and any existing naming rules before proceeding.

---

### Step 1 — Ingest the input

Accept any of:
- **URL batch** — one URL per line, pasted inline
- **CSV** — columns may include `source`, `medium`, `campaign`, `term`, `content`, plus a `url` column
- **Ad platform export** — Google Ads, Meta Ads Manager, or LinkedIn Campaign Manager CSV; map platform fields to UTM params automatically
- **"Build from scratch"** — no input, just a campaign brief; skip to Step 6 to define the schema, then generate URLs

If input is ambiguous (e.g., a partial CSV with no headers), ask one clarifying question before continuing.

---

### Step 2 — Detect or define the naming schema

#### The Canonical UTM Schema (default)

Unless the user provides their own convention, enforce these rules:

| Parameter | Required | Allowed values / pattern | GA4 channel-grouping impact |
|---|---|---|---|
| `utm_source` | Always | Lowercase, no spaces, underscores only. Canonical values: `google`, `facebook`, `instagram`, `linkedin`, `twitter`, `tiktok`, `newsletter`, `partner_[slug]`, `direct` | Maps to Source |
| `utm_medium` | Always | Must match GA4 default-channel definitions exactly (case-sensitive lowercase): `cpc`, `email`, `organic`, `referral`, `social`, `push`, `sms`, `affiliate`, `display`, `video` | A wrong medium lands traffic in "(other)" — highest-severity flag |
| `utm_campaign` | Always | Format: `[brand-slug]_[YYYYQQ]_[initiative]` or `[brand-slug]_[YYYYMMDD]_[initiative]` for one-off sends. Lowercase, underscores, no spaces, no special chars except hyphens within `[initiative]`. Max 80 chars. | Campaign dimension |
| `utm_term` | Paid search only | Keyword, underscored, lowercase | Keyword dimension |
| `utm_content` | A/B / creative differentiation | `[format]-[variant]`, e.g. `banner-blue`, `cta-v2`. No spaces. | Ad content dimension |

**Schema customization:** if the brand already has a schema (detected in `brand.md` or `tracking-plan-taxonomy-builder-auditor` output), honor it over the default. Surface any divergences from the default and note the GA4 impact.

---

### Step 3 — Audit every row

For each URL or row, check every parameter against the schema. Produce a violations table:

```
## UTM Audit — [brand slug] — [date]
Rows audited: N   Violations: V   Clean rows: C

| Row | Parameter | Raw value | Violation | Severity | Corrected value |
|-----|-----------|-----------|-----------|----------|-----------------|
| 3   | utm_medium | Email     | Wrong case — GA4 maps "Email" to "(other)" | HIGH | email |
| 7   | utm_campaign | Q2 Summer Sale | Spaces + missing brand prefix | MED | [slug]_2026Q2_summer-sale |
| 12  | utm_source | (none)    | Missing required parameter | HIGH | [ask user] |
```

**Severity levels:**
- **HIGH** — breaks channel grouping in GA4 or loses the parameter entirely (missing required field, wrong-case medium, space in value, reserved char in URL)
- **MED** — inconsistency that fragments the campaign dimension without breaking routing (caps mismatch in campaign, non-standard separator, no brand prefix)
- **LOW** — style violation (over-long value, undocumented source, missing utm_content on A/B creative)

Flag HIGH violations prominently. Do not silently correct — always show the raw value and the rule it violates.

---

### Step 4 — Emit the validated UTM sheet

After the audit table, output a corrected sheet the team can use directly:

```
## Validated UTM URLs
| # | Final URL | utm_source | utm_medium | utm_campaign | utm_term | utm_content | Status |
```

- Apply all HIGH and MED corrections automatically, marking each cell `[auto-corrected]`.
- For missing required parameters where the correct value cannot be inferred, insert `[FILL: describe what's needed]` and mark the row `INCOMPLETE`.
- For any correction that changes meaning (e.g., inferring a source from context), add a `[confirm]` flag.
- Save the sheet to `./utm/[brand-slug]-utm-validated-[YYYYMMDD].csv` if the input was a file or batch of ≥5 URLs.

---

### Step 5 — Data-quality gate

Invoke `data-qa-measurement-gotcha-checker` on the corrected sheet before presenting final output. Key checks:
- `utm_medium=organic` on paid URLs (medium/source mismatch)
- Self-referral UTMs (source = own domain)
- `utm_source=direct` set explicitly — direct traffic must not be tagged; remove and flag
- Cross-domain scenarios where UTM params would be stripped (document, don't auto-fix)
- `utm_campaign` values that will exceed GA4's 100-char event-parameter limit after URL-encoding

Surface any data-quality issues as an addendum to the violations table.

---

### Step 6 — Governance artifact (naming taxonomy)

Generate when: (a) building from scratch, (b) the user asks for a "naming convention," "taxonomy," or "lock doc," or (c) `tracking-plan-taxonomy-builder-auditor` surfaces a gap.

```markdown
# UTM Naming Taxonomy — [Brand Slug]
Generated: [date]   Owner: [user-supplied or blank]   Version: 1.0

## Source registry
| Source value | Platform | Notes |
…

## Medium registry (GA4 default-channel aligned)
| Medium value | GA4 channel | Usage rule |
…

## Campaign naming formula
[slug]_[YYYYQQ or YYYYMMDD]_[initiative]

## Initiative tag library
(Named initiatives the team uses — seed from input campaigns, expand over time)
| Tag | Definition | Example |
…

## Content naming formula
[format]-[variant]  (e.g., banner-blue, email-v2, cta-above)

## Governance rules
- All new UTMs built against this doc before activation
- Source + medium changes require team-lead sign-off (GA4 channel grouping impact)
- This doc is the source of truth; update it before introducing a new source, medium, or initiative tag
- Review quarterly with brand-brain refresh
```

Save taxonomy to `./utm/[brand-slug]-utm-taxonomy.md`. On subsequent runs, diff the input against the existing taxonomy before writing.

---

## The GA4 Channel-Grouping Alignment Rule (why medium is your highest-severity field)

GA4's default channel grouping is not configurable in most properties. It matches `utm_medium` against a fixed lookup:

| medium value | GA4 default channel |
|---|---|
| `cpc` | Paid Search / Paid Shopping |
| `email` | Email |
| `social`, `social-media` | Organic Social |
| `paid-social` | Paid Social |
| `referral` | Referral |
| `organic` | Organic Search |
| `display` | Display |
| `affiliate` | Affiliates |
| anything else | **(Other)** |

A typo in `utm_medium` moves entire campaigns into "(Other)" — permanently (historical data cannot be reprocessed). Always flag wrong-medium values as HIGH severity and call out the GA4 impact by name.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No audit runs before brand-brain returns. Brand slug anchors the campaign naming formula.
- **Show, don't silently fix.** Every correction must display the raw value, the rule violated, and the corrected value side-by-side. Silent auto-correction hides recurring process failures.
- **Medium is your highest-stakes field.** Wrong case or wrong value on `utm_medium` routes traffic to "(Other)" irrecoverably. Always flag HIGH and name the GA4 channel it maps to.
- **Data-quality gate is mandatory.** Call `data-qa-measurement-gotcha-checker` before final output; do not skip even for small batches.
- **Governance artifact, not just a one-time fix.** Every run either references an existing taxonomy or creates one. One-off fixes without a locked schema will repeat.
- **Real values or `[verify]`.** Never invent a corrected source or medium value; if the right value cannot be inferred, flag `[FILL]` and explain what's needed.

---

## What Not to Do

- Don't silently rewrite parameters without showing the before/after and the violated rule.
- Don't implement brand scanning, voice, or proof logic — call `brand-brain`.
- Don't implement data-quality measurement checks inline — call `data-qa-measurement-gotcha-checker`.
- Don't assume a medium value is correct because it "looks reasonable" — check it against the GA4 channel-grouping lookup.
- Don't use `utm_source=direct`; direct traffic must never be tagged. Flag and remove.
- Don't exceed GA4's 100-char event parameter limit on `utm_campaign` after URL-encoding — flag it.
- Don't generate a validated sheet with INCOMPLETE rows without clearly marking them and listing what's needed to complete them.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and brand slug returned before any audit work?
- Every HIGH violation explicitly flagged with its GA4 channel-grouping impact named?
- Corrected sheet shows raw → corrected for each auto-fix, and `[FILL]` / `[confirm]` for ambiguous cases?
- `data-qa-measurement-gotcha-checker` called; its findings included as an addendum?
- Governance taxonomy created or referenced — not just a one-time fix?
- Output file saved to `./utm/` (for batches ≥5 URLs)?
- No invented source/medium values; only `[verify]` or `[FILL]` for unknowns?
