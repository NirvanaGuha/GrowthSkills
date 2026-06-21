---
name: product-feature-knowledge-base-curator
description: >
  Transforms raw product inputs — changelog entries, release notes, help docs, Notion feature specs,
  support tickets, or any product-team brain dump — into a governed, versioned feature knowledge base
  that every downstream marketing asset can cite without risk of contradiction. Uses the Feature
  Anatomy Framework: each feature gets a canonical record with a plain-language summary, jobs-to-be-done
  tag, ICP fit score, awareness-stage flag, proof anchor, and a "copy-safe" field that locks the single
  true claim writers are allowed to make. Outputs a structured markdown library (./feature-kb/[brand]-features.md)
  plus a CHANGELOG.diff showing what changed since the last run. Callers — blog posts, battle cards,
  email sequences, landing pages — query the KB instead of guessing at product facts. The result:
  content that can never contradict the product, and a junior marketer who writes like they shipped
  the feature. Use when the user says "build a feature library," "organize our changelog," "turn these
  release notes into marketing copy," "feature KB," "what does [feature] actually do," "update the
  product brain," or "sync our KB with the latest release notes."
---

# Product & Feature Knowledge Base Curator

Marketing copy that contradicts the product is worse than no copy. This skill turns raw product inputs into a single structured feature library that every other marketing skill in this library can query — so blog posts, battle cards, email sequences, and landing pages all draw from the same verified source of truth.

The job: ingest → normalize → classify → store → diff. Not a one-time task — each run updates the library incrementally so the KB ages gracefully as the product evolves.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's ICP, positioning, offer mechanics, and banned words. Feature copy-safe summaries must be in-voice and ICP-relevant.
- *(optional, when installed)* `offer-pricing-brain` — confirms which features are gated per tier before writing tier-flag metadata. If absent, tier data is pulled from brand.md's offer section or left `[verify]`.
- *(optional, when installed)* `proof-vault` — cross-checks any numeric claims in release notes against stored proof before marking them copy-safe. If absent, all unverified numbers are flagged `[verify]`.
- *(optional, when installed)* `icp-persona-builder` — supplies the persona slugs used in the ICP-fit field. If absent, uses the ICP summary from brand.md.

**Downstream callers** (skills that READ this KB, not call it):
`blog-post-drafting-engine`, `battlecard-objection-handler`, `landing-product-page-copy-writer`, `content-brief-builder`, `push-notification-copy-generator`, `campaign-brief-builder`, `in-app-microcopy-writer-auditor`, `case-study-customer-spotlight-production-suite`.

---

## How a run works

```
Step 0  Load the brand         ──► call brand-brain; get ICP, voice, banned words, offer mechanics
Step 1  Ingest inputs          ──► normalize every input into a raw feature list
Step 2  Deduplicate + cluster  ──► collapse same-feature entries across sources
Step 3  Classify each feature  ──► apply the Feature Anatomy Framework (6 fields per record)
Step 4  Diff against the KB    ──► new / updated / deprecated since last run
Step 5  Write & confirm        ──► write ./feature-kb/[brand]-features.md + CHANGELOG.diff
Step 6  Return query index     ──► summary table the caller can scan to find features by tag
```

---

## Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Receive the active brand's digest: ICP + awareness tendency, voice adjectives, banned words, offer mechanics, positioning line, real proof. Do not write any feature records until this returns.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If neither exists, ask the user for: brand name, ICP one-liner, voice adjectives, and any banned terms. Proceed once received.

---

## Step 1 — Ingest inputs

Accept any of the following (mixed is fine):

| Input type | What to extract |
|---|---|
| Changelog / release notes (text or MD) | Version tag, date, feature name, raw description, tier mention |
| Help docs / knowledge-base article | Feature name, how-it-works prose, prerequisite, screenshots described |
| Notion / Google Doc spec | Feature name, user story, acceptance criteria, launch date |
| Support ticket batch | Pain points, feature names mentioned, workarounds (signals gaps or confusion) |
| Product team brain dump | Any unstructured facts; extract by sentence |
| Existing `[brand]-features.md` | Treat as prior state; use as diff baseline |

If input is absent or ambiguous, ask one question: "What's the source — release notes, help docs, or a brain dump?" Then proceed.

---

## Step 2 — Deduplicate + cluster

Same feature appears under multiple names across input types (e.g., "smart segmentation," "audience segmentation," "segment builder"). Before classifying:

1. Group by semantic similarity — same capability = one canonical record.
2. Assign a `feature_slug` (lowercase-hyphenated, deterministic — e.g., `smart-segmentation`).
3. Mark the canonical name (prefer the marketing-facing name from the most authoritative source: marketing site > help docs > release notes > spec).
4. Note all aliases in the record's `aliases` field so search across inputs still finds it.

---

## Step 3 — Classify each feature (Feature Anatomy Framework)

Every feature record has exactly six classification fields. This is the opinionated framework that converts raw product facts into marketing-ready metadata.

```yaml
feature_slug: smart-segmentation          # lowercase-hyphenated, stable identifier
canonical_name: Smart Segmentation        # marketing-facing name
version_added: "4.2"                      # first version it shipped; "unknown" if not found
tier_flags: [Growth, Enterprise]          # which plan tiers include it; [verify] if unknown
summary_plain: >                          # ≤2 sentences, ICP-relevant, in brand voice, no jargon
  Automatically groups subscribers by behavior so you send the right message to the right
  segment without building rules manually. [verify — confirm "automatically" claim]
jtbd_tags:                                # jobs this feature serves (pick from controlled list below)
  - reduce-manual-work
  - improve-targeting-precision
icp_fit: high                             # high / medium / low relative to brand's primary ICP
awareness_stage: solution-aware           # unaware / problem-aware / solution-aware / product-aware / most-aware
copy_safe_claim: >                        # THE single pre-approved claim writers may use verbatim
  Smart Segmentation groups subscribers by behavior automatically — no rules to build.
contradictions_flag: none                 # "none" or a specific contradiction to resolve before use
proof_anchor: "[verify]"                  # cite if a real metric exists; [verify] if unconfirmed
aliases: [audience segmentation, segment builder, audience builder]
```

### JTBD tag controlled list (use these exact slugs — add new ones only when no existing tag fits)

`reduce-manual-work` · `improve-targeting-precision` · `increase-deliverability` · `drive-revenue-recovery` · `accelerate-onboarding` · `retain-subscribers` · `personalize-at-scale` · `enable-a-b-testing` · `surface-analytics-insights` · `integrate-third-party-stack` · `reduce-churn` · `simplify-compliance` · `automate-workflows` · `boost-conversion-rate` · `drive-feature-adoption`

### The `copy_safe_claim` rule

This field is the only claim downstream skills are permitted to use verbatim without further verification. A claim is copy-safe when:
- It makes no numeric assertion that isn't confirmed by `proof-vault` or a primary source.
- It uses no superlatives ("best," "only," "fastest") unless the brand can prove them.
- It contains no banned words from the brand's `brand.md`.
- It accurately describes what the feature does at GA — not roadmap, not beta-only.

If a claim can't clear all four gates, mark it `[verify]` and explain what's blocking.

---

## Step 4 — Diff against the prior KB

If a prior `[brand]-features.md` exists:

1. For each record in the new input: is this a **new feature**, an **update** (any field changed), or **unchanged**?
2. Flag **deprecated features** — present in prior KB but absent from new inputs (ask the user to confirm before removing; mark `status: deprecated` as a soft delete).
3. Output a `CHANGELOG.diff` block showing only the delta:

```
## KB CHANGELOG — [date]
### New (N)
- [feature_slug] — [one-line summary]
### Updated (U)
- [feature_slug] — field changed: [field] · was: [old] → now: [new]
### Deprecated (ask to confirm)
- [feature_slug] — absent from this input; confirm removal?
```

If no prior KB exists, the diff is "Initial build — all records new."

---

## Step 5 — Write & confirm

Output path: `./feature-kb/[brand-slug]-features.md` (relative to the user's CWD — never inside the skill folder).

File structure:

```markdown
# [Brand] Feature Knowledge Base
_Updated: [date] · Version: [semver, e.g. 1.3.0] · Features: [count] · Source: [input types used]_

## Index
| Slug | Name | Tier | ICP Fit | JTBD Tags | Status |
|------|------|------|---------|-----------|--------|
...

## Feature Records
[one YAML block per feature, in slug-alphabetical order]

## CHANGELOG
[cumulative diff appended at the bottom, newest on top]
```

Bump the KB version on every write: patch for updates/fixes, minor for new features, major for structural schema changes.

Confirm to the user in one line: *"KB written to `./feature-kb/[brand-slug]-features.md` — [N] new, [U] updated, [D] deprecated. Run again anytime to sync."*

---

## Step 6 — Return query index

After writing, return a compact query table so the caller (or user) can immediately find features by tag:

```
## Feature index — [brand]
| Slug | Copy-Safe Claim (preview) | JTBD | ICP Fit | Tier | Awareness Stage |
```

Also surface: features with unresolved `[verify]` items (writers must not use these), and features flagged `contradictions_flag: [anything other than "none"]`.

---

## Querying the KB (for downstream callers)

When another skill needs product facts, it reads `./feature-kb/[brand-slug]-features.md` and filters by:
- `jtbd_tags` — to find features relevant to a campaign angle
- `icp_fit: high` — to limit copy to highest-relevance features
- `awareness_stage` — to match the right feature to the reader's stage
- `copy_safe_claim` — to pull the one pre-approved claim verbatim
- `contradictions_flag: none` — to avoid features still under review

Callers should NEVER paraphrase `copy_safe_claim` beyond minor grammar adjustments. They should ALWAYS check `contradictions_flag` before use.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No feature records before `brand-brain` returns. Voice and banned words apply to `summary_plain` and `copy_safe_claim`.
- **Copy-safe is a gate, not a suggestion.** A claim with `[verify]` in `copy_safe_claim` is not usable copy. Do not soften this.
- **Canonical, not comprehensive.** One record per feature, one copy-safe claim per record. Ambiguity in the KB = contradiction risk in content.
- **Soft deletes only.** Deprecated features stay in the KB as `status: deprecated` until the user confirms removal. Never silently delete product history.
- **Aliases over duplicates.** Resist creating a new record for a renamed feature; add the new name as an alias, note the rename in CHANGELOG.
- **Diff is the deliverable.** For ongoing runs, the CHANGELOG.diff is as important as the KB itself — it is the signal that alerts content teams to update live assets.

---

## What Not to Do

- Don't invent features not present in the inputs. Gaps in the KB are honest; fabricated records corrupt downstream copy.
- Don't write a `copy_safe_claim` for roadmap features, deprecated features, or beta-only functionality unless explicitly scoped.
- Don't accept a numeric claim as copy-safe unless `proof-vault` confirms it or a primary source citation is in the record.
- Don't use brand.md's offer section to infer tier gates — if tier data isn't in the input, mark `[verify]`.
- Don't let banned words from `brand-brain` appear in `summary_plain` or `copy_safe_claim`. They will propagate to every downstream asset that reads the KB.
- Don't skip the diff step. An undiffed update silently loses the signal that live content needs refreshing.

---

## Quality Checklist (self-review before writing)

- `brand-brain` called and active brand loaded before any record was written?
- Every `copy_safe_claim` passes all four gates (no unconfirmed numbers, no superlatives, no banned words, GA-only)?
- Every unconfirmed metric marked `[verify]` — in `summary_plain`, `copy_safe_claim`, and `proof_anchor`?
- Duplicate/alias clustering done — no two records describe the same capability?
- CHANGELOG.diff produced and appended; deprecated features soft-deleted with confirmation pending?
- KB written to `./feature-kb/[brand-slug]-features.md` (not inside the skill folder)?
- Query index returned with `[verify]` and contradiction flags surfaced for the caller?
