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

Approved draft in → correctly published, fully SEO-tagged, platform-formatted post out. The mechanism is the **Publish Readiness Gate**: a five-state machine — INTAKE → GAP-FILL → FORMAT → QA → SHIP — where a draft cannot reach the next state until the current gate's hard-fail conditions clear. Each gate either routes the fix to a sibling skill or blocks the run. Every field explicit, every platform's real constraints respected, the live output confirmed before the job is done.

This skill does not draft, brief, or rewrite content. It receives a finished, approved artifact and runs it through the gate. If a gate condition can't be met from the artifact, it routes to the right sibling before touching a CMS — it never papers over a failure to ship faster.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's voice, SEO defaults (focus keyphrase patterns, CTA URLs), and any CMS-specific house rules. Does not reimplement brand resolution. **Fallback if `brand-brain` is absent or returns no brand:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for CMS house rules, keyphrase pattern defaults, and banned words inline before proceeding.
- **`on-page-seo-optimizer`** (conditional) — if SEO fields are absent or weak, calls this to generate an explicit SEO title, meta description, focus keyphrase, slug recommendation, and OG fields before publish.
- **`meta-title-description-bulk-writer`** (conditional) — for batch publishes (5+ URLs/posts), delegates meta-field generation here rather than writing each inline.
- **`internal-linking-planner`** (conditional) — if the CMS is WordPress and internal links were not pre-baked into the draft, calls this to generate bidirectional link inserts before final upload.
- **`content-qa-reviewer`** (Step 4, always) — runs a structured pre-publish QA pass against the brand voice and completeness checklist; blocks publish on any hard failure.
- **`editorial-style-guide`** (reference) — consulted for platform-specific formatting rules if not already embedded in the brand brain.

---

## How a run works — the Publish Readiness Gate

This skill runs the **Publish Readiness Gate** (house model): a five-state machine where a draft cannot reach the next state until the current gate's hard-fail conditions are clear. Each gate either **routes** the draft to a sibling skill to fix the failure or **blocks** the run until a human resolves it. Nothing ships until it has passed every gate left-to-right.

```
INTAKE ──► GAP-FILL ──► FORMAT ──► QA ──► SHIP
  │           │            │        │       │
  └ triage    └ fill SEO/  └ apply  └ block └ publish
    + brand     links via    field    on      + log
    load        siblings     map      fail
```

### The gate table (read this top to bottom)

| Gate | What it checks | Hard-fail condition | Routes to / Outcome |
|---|---|---|---|
| **INTAKE** | Draft is approved; platform target + metadata present | Draft not marked approved, or no platform named | **BLOCK** — ask the user; never publish an unapproved draft |
| **INTAKE** | Brand context loaded | `brand-brain` not yet called | Call `brand-brain` (Step 0); on absence use the fallback line below |
| **GAP-FILL** | Five explicit SEO fields present | Any of SEO title / meta / keyphrase / OG title / OG desc missing or weak | Route to `on-page-seo-optimizer` (single) or `meta-title-description-bulk-writer` (5+); **ALLOW** once filled |
| **GAP-FILL** | Internal links present (WordPress only) | Links absent from the draft | Route to `internal-linking-planner`; **ALLOW** once inserted |
| **GAP-FILL** | Focus keyphrase derivable | No keyphrase in brief and none derivable | Route to `on-page-seo-optimizer` to derive from body; **BLOCK** if still none |
| **FORMAT** | Platform field map applied | Any field unmapped or carrying a template token | Apply the field map below; **BLOCK** on any `#post_title`-style token |
| **FORMAT** | Heading hierarchy valid | H1 absent or used >1× in body | Fix in place (mechanical), re-check; **BLOCK** if structure is broken |
| **QA** | Pre-publish review clean | Any hard failure from `content-qa-reviewer` (see blocklist) | Route back to the relevant gate; **BLOCK** until zero hard failures |
| **SHIP** | Live output matches brief | Rendered title/meta/slug ≠ intended values | **BLOCK** — fix and re-publish; never log a mismatched publish |
| **SHIP** | Audit trail written | Publish log entry missing | Write the log; the run is not done until it exists |

A gate that **routes** hands off, waits for the sibling, then re-evaluates its own condition. A gate that **blocks** stops the run and surfaces the failure — it does not silently skip ahead. The five-field rule, the field maps, and the QA blocklist below are the gate criteria this skill owns; the fixes they trigger live in the siblings.

---

## Cross-platform SEO field equivalence (the FORMAT gate's spine)

One canonical field per row. The columns are where it lives on each platform, the limit to respect, and the gotcha that breaks it if you forget. Read across a row before you touch a field — the same canonical value lands in a differently-named box on each platform, and each box has its own failure mode.

| Canonical field | WordPress (AIOSEO/Yoast) | Webflow | Notion | Google Docs | Limit | Gotcha |
|---|---|---|---|---|---|---|
| **SEO Title** | AIOSEO "SEO Title" field | SEO Settings → Title Tag | Integration meta panel (Super/Potion) | SEO-fields table, top of doc | ≤60 chars | WP defaults to a `#post_title #separator #site_title` token — **must be overwritten with a literal string** |
| **Meta Description** | AIOSEO "Meta Description" | SEO Settings → Meta Description | Integration meta panel | SEO-fields table | 140–155 chars | Webflow truncates silently in preview; Notion-to-web only passes it if the integration's meta toggle is on |
| **Focus Keyphrase** | AIOSEO/Yoast "Focus Keyphrase" | No native field — track in brief | No native field — track in brief | SEO-fields table | 1 phrase | Only WP/Yoast *scores* against it; on Webflow/Notion it's a discipline, not a setting — never assume the CMS enforces it |
| **Slug** | Permalink slug | Auto-generated from Name → override | Page URL (integration) | n/a (handoff) | ≤60 chars, keyword-first | Webflow auto-slugs from the title and keeps the first version on rename — **validate after every title edit** |
| **OG Title** | AIOSEO Social tab | Designer → Open Graph → Title | Integration social panel | SEO-fields table | ≤60 chars | All three CMSs fall back to SEO Title if blank — set it explicitly even when it matches |
| **OG Description** | AIOSEO Social tab | Designer → Open Graph → Description | Integration social panel | SEO-fields table | 140–155 chars | Same silent fallback to meta description — set it on purpose, don't inherit by accident |
| **OG / Featured Image** | Featured Image + alt text | Open Graph image (1200×630) | Cover image | n/a | 1200×630 px | Alt text ≠ "featured image"; Webflow OG image is separate from the body hero — both must be set |

**Why WordPress carries the explicit-fields rule hardest:** AIOSEO and Yoast ship live template tokens (`#post_title`, `%%sep%%`) in the SEO Title and Meta Description boxes by default. A post can look "filled in" while every field is a placeholder that renders dynamically. This is the user's standing house rule — on PushEngage WP posts, **every AIOSEO field is set explicitly, never via a template token** — and it is the single most common silent publish failure across all four platforms.

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

## The EXPLICIT SEO FIELDS rule (the GAP-FILL gate's pass condition)

The GAP-FILL gate does not open until all five fields are written in full. A token in any one of them is a FORMAT-gate hard fail later, so resolve it here.

1. **SEO Title** — the string that appears in the `<title>` tag; includes keyphrase + brand name; max 60 chars.
2. **Meta Description** — 140–155 chars; benefit-led; contains the focus keyphrase.
3. **Focus Keyphrase** — the single phrase this post competes on; used by AIOSEO/Yoast to score the post.
4. **OG Title** — the social share headline; may differ from the SEO title for click optimization.
5. **OG Description** — the social share description; pulled from meta or written fresh.

If any of these are absent on intake, the gate **routes** to `on-page-seo-optimizer` (single post) or `meta-title-description-bulk-writer` (5+) and re-checks before opening. It does not publish past a blank field.

---

## The QA gate's blocklist (executed by content-qa-reviewer)

The QA gate is the last checkpoint before SHIP. This skill owns the blocklist — the exact conditions that hold a post back — and routes the *review* to `content-qa-reviewer`. Any hard failure sends the draft back to the gate that owns the fix; none of these reach SHIP:

| Hard failure | Sends back to |
|---|---|
| Missing or template-token SEO fields | GAP-FILL → `on-page-seo-optimizer` |
| H1 absent or used more than once in the body | FORMAT (fix in place) |
| Brand-banned words present | INTAKE → flag to writer; do not fix here |
| CTA URLs pointing to a dead or wrong destination | FORMAT (correct destination) |
| Featured image absent (WordPress/Webflow) | GAP-FILL (upload + alt text) |
| Slug is stop-words-only or duplicates an existing post | FORMAT (rewrite slug) |
| Post contains `[verify]`-tagged claims not yet confirmed | INTAKE → BLOCK; confirm before publish |

Surface soft warnings (thin meta, tag count outside 3–6, generic alt text) as fixes to apply before shipping, not blockers — unless the brand brain flags them as hard rules.

---

## Batch publish mode

Triggered when the user provides a list of drafts (5+), a content calendar export, or a CSV of posts to stage. The same five gates run, batched per stage — every post clears one gate before the batch advances to the next.

1. **GAP-FILL (batched):** run `meta-title-description-bulk-writer` to generate SEO fields for all URLs in one pass; run `internal-linking-planner` across the batch to surface cross-link opportunities before any post goes live.
2. **FORMAT:** apply the platform field map to each post.
3. **QA:** run the blocklist per post; pull any post that hard-fails into a separate flagged list — the clean batch ships, the flagged posts hold.
4. **SHIP:** produce a publish log CSV: `post-slug, platform, seo-title, meta-desc, keyphrase, publish-status, qa-flags`.
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

- **The gate runs left to right.** INTAKE → GAP-FILL → FORMAT → QA → SHIP. A draft never skips a gate, and a gate either routes the fix to a sibling or blocks the run. No back-door publishes.
- **Explicit fields, always.** No template tokens. No auto-generate. No empty SEO fields. If it's unknown, the gate gets it — it does not skip it.
- **Platform rules are real.** WordPress ≠ Webflow ≠ Notion ≠ Docs. Read across the equivalence table; never assume a field name or limit transfers.
- **QA before publish, not after.** The QA gate runs `content-qa-reviewer` before any CMS write; a hard failure routes back, it does not get waved through.
- **Brand-brain first.** CMS house rules, keyphrase patterns, CTA destinations, and banned words come from the brand — not guessed.
- **Own the criteria, route the fix.** This skill owns the gate conditions, the field maps, and the blocklist; the repairs they trigger live in `on-page-seo-optimizer`, `internal-linking-planner`, `content-qa-reviewer`, and `brand-brain`. It orchestrates the final mile; it does not rebuild those capabilities.
- **Log every publish.** The SHIP gate is not closed until the ops log entry exists — it feeds downstream decay sweeps.

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

## Quality checklist (walk the gates)

- **INTAKE:** draft confirmed approved and platform named; `brand-brain` called and the brand's CMS house rules, keyphrase patterns, and banned words loaded?
- **GAP-FILL:** all five explicit SEO fields present and written out in full (no tokens, no blanks); `internal-linking-planner` called if WordPress and links were absent; featured image uploaded with explicit, keyword-descriptive alt text?
- **FORMAT:** correct platform field map applied; every field read across the equivalence table for the right name/limit/gotcha; H1 present exactly once with correct hierarchy; zero template tokens?
- **QA:** `content-qa-reviewer` called; every blocklist item clear; routed-back failures re-checked, not waved through?
- **SHIP:** rendered title/meta/slug match intended values; post published / staged / scheduled per the brief (not left as draft by default); publish log entry written to `./publish-logs/`?
- **Batch:** the five gates ran batched; log CSV saved; any QA-blocked posts pulled into the flagged list, not shipped with the clean batch?
