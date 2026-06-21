---
name: meta-title-description-bulk-writer
description: >
  Takes a list of URLs and their target keywords and returns optimized meta titles and
  meta descriptions for every URL in one pass — at scale, without sacrificing SEO craft.
  Applies the Click-Through Optimization (CTO) framework: each title balances the primary
  keyword, a hook that raises the click-rate, and a character envelope that survives SERP
  truncation; each description closes with a value proposition and a call to action.
  Brand voice and banned words come from brand-brain (Layer-0); keyword clusters and
  search-intent signals can be piped in from keyword-research-clustering-suite or
  on-page-seo-optimizer. Output is a ready-to-import CSV plus an inline table, enabling
  a non-technical marketer to paste directly into Yoast, AIOSEO, RankMath, or any CMS
  that accepts bulk CSV. Use when someone says "write meta titles," "bulk meta tags,"
  "rewrite my page titles," "SEO titles for all my URLs," "meta descriptions at scale,"
  or pastes a URL list and asks for optimization.
---

# Meta Title & Description Bulk Writer

Give it URLs and keywords. Get a production-ready CSV of optimized meta titles and descriptions — every one on-brand, within the character envelope, and written to win the click, not just rank.

This skill handles volume without sacrificing craft. It does not crawl, audit, or rank-track. If you need to decide *which* pages need new meta tags first, run `seo-content-health-decay-audit` or `gsc-monitoring-suite` before this skill. If you need the keyword clusters to feed into this skill, run `keyword-research-clustering-suite` first.

---

## Skills this calls

- **`brand-brain`** (required) — loads voice adjectives, banned words, positioning line, and brand name casing before the first title is written.
- *(optional, upstream)* `keyword-research-clustering-suite` — provides primary + secondary keywords and intent tags; synthesize inline when absent.
- *(optional, upstream)* `on-page-seo-optimizer` — can hand off per-page intent signal and existing H1/copy for message-match; not required for bulk runs.
- *(optional, upstream)* `gsc-monitoring-suite` or `seo-content-health-decay-audit` — identifies which URLs are the highest-priority rewrite targets.

---

## How a run works

```
Step 0  Load the brand        ──► call brand-brain; receive voice + banned words
Step 1  Intake                ──► URL list + keywords + optional context (page type, existing title)
Step 2  Intent classification ──► assign intent type per URL (informational / navigational / commercial / transactional)
Step 3  Write in bulk         ──► apply CTO Framework per URL
Step 4  Truncation check      ──► flag any title >60 chars or description >158 chars
Step 5  Output                ──► inline table + CSV saved to ./seo/meta-tags-[slug]-[date].csv
```

---

## Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** before writing a single character. It returns the active brand's digest including voice adjectives, banned words, brand name + casing, and positioning line. Every title and description must honor those as hard overrides.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If none exists, ask the user for three things: brand name (exact casing), 2–3 voice adjectives, and any banned words. Then proceed.

---

## Step 1 — Intake

Minimum required per URL:
- **URL** (or slug)
- **Primary keyword** — the exact phrase to target

Optional (improves output significantly):
- **Secondary / supporting keywords** — worked in where natural
- **Page type** — blog post, product page, category page, landing page, homepage, comparison page
- **Existing title** — enables before→after output
- **H1 or first paragraph** — message-match anchor

If the user provides a spreadsheet, CSV, or markdown table, parse it. If just a URL list, ask for keywords before proceeding unless they're derivable from the slug.

---

## Step 2 — Intent classification

Classify each URL's **dominant search intent** before writing. Intent determines the right hook and CTA strategy — mismatching intent is the most common reason well-ranked pages don't earn the click.

| Intent type | What the searcher wants | Title hook style | Description CTA style |
|---|---|---|---|
| Informational | Learn / understand | "How to…", "What is…", question form | "Learn exactly how…", "Get the full guide" |
| Commercial | Compare / evaluate | "Best…", "X vs Y", "[Year] Review" | "Compare plans", "See which one wins" |
| Transactional | Buy / sign up / try | Offer + action verb | "Start free", "Get it today", urgency-safe |
| Navigational | Find a specific page | Brand name + destination | Short, direct, confirmatory |

Write the intent type into the output table — it's the audit trail that justifies the copy choice.

---

## Step 3 — The CTO Framework (Click-Through Optimization)

Every title and description passes through three lenses before it ships.

### Lens 1: Keyword placement (rank signal)
- Primary keyword in the **first 30 characters** of the title when possible — exact match or close variant preferred.
- Secondary keywords can appear in the description; keyword stuffing is disqualifying.
- Never sacrifice readability for keyword density.

### Lens 2: Hook (click signal)
Titles that rank but don't earn the click are wasted effort. Each hook is chosen to match the intent and outperform generic formulations.

**Hook patterns by intent:**

- **Informational:** year-stamp (`[2026]`), specificity (`"7 Steps"`), question form, "The [Guide/Playbook/Breakdown]"
- **Commercial:** `"Best [X] for [Audience]"`, `"[X] vs [Y]: Which Is Right for You?"`, `"[Year]'s Top Picks"`
- **Transactional:** benefit-led verb (`"Grow…"`, `"Automate…"`, `"Save…"`), offer-forward (`"Free trial"`, `"14 days free"`), social proof when real (`"Trusted by 25,000+ sites"` — only if confirmed)
- **Navigational:** clear destination + brand name; no hook needed

Proof and superlatives in titles must be confirmed or marked `[verify]`.

### Lens 3: Character envelope (display signal)
- **Title:** 50–60 characters ideal; hard cap at 60; under 40 is often a missed opportunity.
- **Description:** 140–158 characters ideal; hard cap at 160; anything under 120 leaves value on the table.

Flag, don't auto-truncate. Show the character count in the output table so the user can decide whether to compress.

---

## Description craft

A description does one job: convince the searcher that clicking this result is worth their time. It is not a keyword-stuffing surface.

**Structure (2-sentence pattern):**
1. **Setup sentence** — what the page delivers; ideally echoes the primary keyword and page type; addresses the searcher's core intent directly.
2. **Value + CTA sentence** — the unique reason to click this result over the others; close with a low-friction CTA matched to the intent.

**Voice rules (from brand-brain):**
- Honor voice adjectives and banned words absolutely.
- Write in second person (`"you"`, `"your"`) for most cases; first person for brand-preference comparison pages.
- No exclamation marks unless the brand explicitly permits them.
- Active verbs; avoid passive constructions.

**What not to do in descriptions:**
- Don't just restate the title — earn new ground.
- Don't list features when the reader wants benefits.
- Don't fabricate proof or statistics (`[verify]` anything unconfirmed).
- Don't end mid-sentence — Google truncates but the copy should be complete within the envelope.

---

## Output format

### Inline table (always shown)

```
| URL | Intent | Primary KW | Meta Title (chars) | Meta Description (chars) | Flags |
|-----|--------|------------|-------------------|--------------------------|-------|
```

Flags column: `OVER_TITLE`, `OVER_DESC`, `VERIFY_CLAIM`, `KEYWORD_MISSING`, `WEAK_HOOK`.

### CSV file (always saved)

Save to `./seo/meta-tags-[brand-slug]-[YYYY-MM-DD].csv` with columns:

```
url,intent,primary_keyword,meta_title,meta_title_chars,meta_description,meta_description_chars,flags
```

This format imports directly into Yoast SEO (bulk edit), AIOSEO's Post Bulk Editor, and RankMath's Bulk Edit. Note the target CMS in the output header so the user knows which tool to open.

If the user provides existing titles, add `existing_title` and `existing_description` columns and lead with a before→after diff note.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No title ships before the active brand's voice and banned words are loaded.
- **Intent before hook.** Classify intent before choosing any hook pattern — wrong intent match is the most expensive mistake in bulk rewrites.
- **Keyword in position, not density.** First 30 characters is the goal; natural language is the constraint. Never sacrifice fluency.
- **Envelope discipline.** Count every character. Flag overruns; never silently truncate.
- **Real proof only.** No invented stats or superlatives; unconfirmed claims are `[verify]`.
- **Message match.** The title should promise what the page delivers. If the page can't support the title, flag the page — not fix the copy.
- **CSV is the deliverable.** Bulk work has one job: get it into the CMS fast. The table is the review surface; the CSV is what ships.

---

## What not to do

- Don't write titles before `brand-brain` returns the active brand's banned words and voice.
- Don't reuse the same hook pattern across every URL in the batch — vary by intent type.
- Don't treat the description as a second meta title — it needs to advance the argument, not echo it.
- Don't invent social proof ("millions of users," "#1 rated") without a confirmed source.
- Don't ship a title over 60 characters without flagging it — Google will rewrite it and may rewrite it badly.
- Don't skip the intent column — it's the audit trail that justifies every copy decision.
- Don't run this skill without a keyword list; if the user provides only URLs, ask for keywords first (or explain the derivation from slugs and confirm).

---

## Quality checklist (self-review before presenting)

- `brand-brain` called; voice adjectives, banned words, and brand name casing loaded?
- Intent classified for every URL — not assumed from slug alone?
- Primary keyword appears in the first 30 characters of every title (or deviation explained)?
- Every title is 50–60 characters; every description is 140–158 characters; overruns flagged?
- Description follows 2-sentence setup + value/CTA pattern; no title echo?
- Proof claims confirmed or marked `[verify]`?
- Output table complete with Flags column; CSV saved to `./seo/meta-tags-[slug]-[date].csv`?
- CMS import note included (Yoast / AIOSEO / RankMath)?
