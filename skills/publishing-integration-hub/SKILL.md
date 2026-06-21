---
name: publishing-integration-hub
description: >
  Takes an approved draft + its metadata and publishes it correctly to WordPress, Webflow, Notion,
  or Google Docs — with every SEO field set explicitly, every platform formatting rule respected, and
  a post-publish QA pass confirming the live output matches the brief. Never uses template tokens like
  #post_title; every field (SEO title, meta description, focus keyphrase, OG title/description, slug,
  category, tags, featured image alt-text) is written out in full. Works as the terminal step in any
  content pipeline — it does not write or brief copy, it ships what's already approved. Use when the
  user says "publish this," "push to WordPress / Webflow / Notion," "set the SEO fields," "format for
  CMS," "finalize for publish," "send to Docs," or hands over a finished draft and asks it to go live.
  Also use as the final orchestrated step when called by keyword-to-published-post-pipeline-runner or
  any campaign orchestrator that needs a publish step.
---

# Publishing Integration Hub

Approved draft in → correctly published, fully SEO-tagged, platform-formatted post out. Every field explicit. Every platform's real constraints respected. A QA pass confirms the live output before the job is done.

This skill does not draft, brief, or rewrite content. It receives a finished, approved artifact and handles the mechanical precision of getting it published correctly. If the draft needs work, it routes to the right sibling before touching a CMS.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's voice, SEO defaults (focus keyphrase patterns, CTA URLs), and any CMS-specific house rules. Does not reimplement brand resolution. **Fallback if `brand-brain` is absent or returns no brand:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for CMS house rules, keyphrase pattern defaults, and banned words inline before proceeding.
- **`on-page-seo-optimizer`** (conditional) — if SEO fields are absent or weak, calls this to generate an explicit SEO title, meta description, focus keyphrase, slug recommendation, and OG fields before publish.
- **`meta-title-description-bulk-writer`** (conditional) — for batch publishes (5+ URLs/posts), delegates meta-field generation here rather than writing each inline.
- **`internal-linking-planner`** (conditional) — if the CMS is WordPress and internal links were not pre-baked into the draft, calls this to generate bidirectional link inserts before final upload.
- **`content-qa-reviewer`** (Step 4, always) — runs a structured pre-publish QA pass against the brand voice and completeness checklist; blocks publish on any hard failure.
- **`editorial-style-guide`** (reference) — consulted for platform-specific formatting rules if not already embedded in the brand brain.

---

## How a run works

```
Step 0  Load the brand          ──► call brand-brain; surface CMS-specific house rules
Step 1  Receive + triage         ──► intake draft, platform target, metadata, SEO fields
Step 2  Fill gaps               ──► call on-page-seo-optimizer / internal-linking-planner
                                     if any required fields are absent
Step 3  Format for platform      ──► apply platform's real field map and formatting rules
Step 4  QA pass                 ──► call content-qa-reviewer; block on hard failures
Step 5  Publish                 ──► write to CMS or produce the publish-ready artifact
Step 6  Confirm + log           ──► verify live output; write publish log
```

---

## Platform field maps (non-negotiable)

Use these exact mappings. Never use template tokens (`#post_title`, `%%title%%`, placeholder slugs).

### WordPress (with AIOSEO or Yoast)

| Field | Rule |
|---|---|
| Post Title | Written headline — the exact title the post uses |
| Slug | Lowercase, hyphen-separated, keyword-first, ≤60 chars, no stop words |
| SEO Title | Explicit string: `Primary Keyphrase | Brand Name` — max 60 chars |
| Meta Description | Explicit string: benefit-led, includes keyphrase near the front, 140–155 chars |
| Focus Keyphrase | Single phrase, exact match to the primary keyword the post targets |
| OG Title | May match SEO title or use a click-optimized variant; explicit string |
| OG Description | Explicit string — often matches meta description; may be conversational |
| Category | Exact match to an existing taxonomy; create new only if briefed to |
| Tags | 3–6 descriptive tags; no stop-word-only tags |
| Featured Image | File uploaded; Alt text = explicit keyword-descriptive string, not "featured image" |
| Author | Set to correct profile; not left as admin default |
| Publish Status | Draft → Scheduled (if date given) → Publish (if immediate) |

**Hard rule:** every AIOSEO / Yoast field must be a written-out value. No template tokens, no empty fields, no "auto-generate." If a field's value is unknown, call `on-page-seo-optimizer` before proceeding.

### Webflow (CMS Collection)

| Field | Rule |
|---|---|
| Name / Title | The post headline; drives the auto-slug — confirm or override |
| Slug | Validate after autogeneration; override to keyword-first if needed |
| SEO Title tag | Explicit string in SEO Settings tab; 50–60 chars |
| SEO Meta Description | Explicit string; 140–155 chars |
| OG Title | Explicit; set in Social tab |
| OG Description | Explicit; set in Social tab |
| OG Image | Upload + confirm dimensions (1200×630 px) |
| Rich Text Body | Verify heading hierarchy (H1 once → H2 → H3); remove Webflow-inserted divs that break reading order |
| Category / Tag | Match existing reference items; do not create phantom references |
| Published | Staged unless `--publish-now` flag is given |

### Notion (publishing via public page or integration)

| Field | Rule |
|---|---|
| Page Title | Exact draft headline |
| Body Formatting | H1 = article title, H2 = major sections, H3 = sub-sections; toggle blocks for asides |
| Cover Image | Set; alt context added in caption if Notion supports it |
| Properties | Fill every database property specified in the brief (Status, Author, Publish Date, Tags, Category) |
| SEO / Embed | If using a Notion-to-web integration (Super, Potion), verify meta tag passthrough in the integration's panel |
| Share Settings | Public link enabled if the brief calls for it; icon + cover set |

### Google Docs (for handoff or embed-publish flows)

| Field | Rule |
|---|---|
| Document Title | `[BRAND] Post Title — PUBLISH-READY` naming convention |
| Heading Styles | Use built-in Heading 1/2/3 styles — never Bold + Large manually |
| SEO Fields Block | Insert a table at the top: SEO Title / Meta Description / Focus Keyphrase / Slug / OG Title / OG Description — all explicit strings |
| Comments | Clear all resolved comments; leave one top-level note: "Approved for publish — [date]" |
| Sharing | Set to the briefed access level; never leave at "Anyone with the link can edit" unless explicitly instructed |

---

## The EXPLICIT SEO FIELDS rule (applies to all platforms)

Every publish must have five fields written in full before the post goes live:

1. **SEO Title** — the string that appears in the `<title>` tag; includes keyphrase + brand name; max 60 chars.
2. **Meta Description** — 140–155 chars; benefit-led; contains the focus keyphrase.
3. **Focus Keyphrase** — the single phrase this post competes on; used by AIOSEO/Yoast to score the post.
4. **OG Title** — the social share headline; may differ from the SEO title for click optimization.
5. **OG Description** — the social share description; pulled from meta or written fresh.

If any of these are absent on intake, **do not publish** — call `on-page-seo-optimizer` first, then proceed.

---

## Pre-publish QA pass (Step 4, via content-qa-reviewer)

Call `content-qa-reviewer` before writing to any CMS. Block publish if any of these hard failures come back:

- Missing or template-token SEO fields
- H1 absent or used more than once in the body
- Brand-banned words present
- CTA URLs pointing to a dead or wrong destination
- Featured image absent (WordPress/Webflow)
- Slug contains stop words only or duplicates an existing post
- Post contains `[verify]`-tagged claims not yet confirmed

Surface soft warnings (thin meta, tag count outside 3–6, alt text generic) as fixes to apply before shipping, not blockers — unless the brand brain flags them as hard rules.

---

## Batch publish mode

Triggered when the user provides a list of drafts (5+), a content calendar export, or a CSV of posts to stage.

1. Run `meta-title-description-bulk-writer` to generate SEO fields for all URLs in one pass.
2. Run `internal-linking-planner` across the batch to surface cross-link opportunities before any post goes live.
3. Apply the platform field map to each post; flag any that fail the QA gate separately.
4. Produce a publish log CSV: `post-slug, platform, seo-title, meta-desc, keyphrase, publish-status, qa-flags`.
5. Save the log to `./publish-logs/[YYYY-MM-DD]-batch.csv`.

---

## Publish log (all modes)

After every successful publish, write a one-line entry to `./publish-logs/[YYYY-MM-DD]-publish-log.md`:

```
| [datetime] | [slug] | [platform] | [SEO title] | [keyphrase] | [status: live/scheduled/staged] |
```

This gives the content ops audit trail that `content-decay-refresh-sweep` and `seo-content-health-decay-audit` read when they age-grade content.

---

## Principles

- **Explicit fields, always.** No template tokens. No auto-generate. No empty SEO fields. If it's unknown, get it — do not skip it.
- **Platform rules are real.** WordPress ≠ Webflow ≠ Notion ≠ Docs. Apply the correct field map; never assume fields transfer.
- **QA before publish, not after.** Call `content-qa-reviewer` at Step 4; blocked posts do not go live until failures are resolved.
- **Brand-brain first.** CMS house rules, keyphrase patterns, CTA destinations, and banned words come from the brand — not guessed.
- **Compose, don't duplicate.** SEO field generation lives in `on-page-seo-optimizer`; internal links live in `internal-linking-planner`; voice/brand lives in `brand-brain`. This skill orchestrates the final mile; it does not rebuild those capabilities.
- **Log every publish.** The ops log is not optional — it feeds downstream audits and decay sweeps.

---

## What not to do

- Do not rewrite, restructure, or "improve" an approved draft — if it needs changes, flag and route to the right writer skill before this step.
- Do not use `#post_title`, `%%title%%`, or any dynamic token as a final SEO Title or Meta Description value.
- Do not publish without a QA pass, even when the user says "just push it."
- Do not invent or assume a focus keyphrase — if it is not in the brief, call `on-page-seo-optimizer` to derive one from the content.
- Do not skip the featured image alt text — "featured image" is not a valid alt string.
- Do not create new CMS taxonomy items (categories, tags, reference fields) unless the brief explicitly instructs it.
- Do not leave the publish log empty — the audit trail is required.

---

## Quality checklist

- `brand-brain` called; active brand's CMS house rules, keyphrase patterns, and banned words loaded?
- All five explicit SEO fields present and written out in full (no tokens, no blanks)?
- Correct platform field map applied (WordPress / Webflow / Notion / Google Docs)?
- Featured image uploaded with an explicit, keyword-descriptive alt text?
- H1 present exactly once in the body; heading hierarchy correct?
- `content-qa-reviewer` called; zero hard failures remaining?
- `internal-linking-planner` called if WordPress and links were absent from the draft?
- Post published / staged / scheduled per the brief's instruction (not left as draft by default)?
- Publish log entry written to `./publish-logs/`?
- Batch mode: log CSV saved; any QA-blocked posts flagged separately?
