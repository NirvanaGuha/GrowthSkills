---
name: on-page-seo-optimizer
description: >
  Takes a draft article (or any page) plus a target keyword and rewrites the on-page SEO layer:
  the meta title, meta description, H1, the H2/H3 heading structure, and the placement of
  semantic/LSI terms and entities through the body. It optimizes copy that already exists — it does
  not invent the article, plan a topic cluster, or analyze the live SERP (those are sibling skills).
  It pulls voice and banned words from the active brand via the `brand-brain` skill so titles and
  metas sound like the brand and never keyword-stuff, and folds a real proof point into the meta
  description when one exists. Use whenever the user says "optimize this for SEO," "on-page SEO,"
  "rewrite my title and meta," "fix my headings," "add LSI/semantic terms," "optimize this draft
  for [keyword]," "improve my meta description," "is this page optimized," or hands over a draft and
  a keyword and asks to make it rank. It optimizes the on-page layer of one page — it does not write
  the article from scratch or do technical/site-wide SEO.
---

# On-Page SEO Optimizer

Hand it a draft and a target keyword; get back a rewritten, ship-ready on-page layer — title tag, meta description, H1, heading skeleton, and the semantic terms placed where they earn their keep. Every title and meta is written in the brand's real voice (loaded from `brand-brain`), so nothing reads like keyword soup and the meta description can carry a real proof point instead of a hollow promise.

This skill optimizes copy that already exists. It does **not** write the article, build the outline from nothing, plan a pillar/cluster, or run a live SERP scrape — if the draft is too thin to optimize, it says so and points you to the brief/SERP skills rather than papering over a weak body with a clever title.

---

## Skills this calls

- **`brand-brain`** (required) — resolves and loads the active brand's voice, banned words, ICP + awareness tendency, offer, and proof. This skill does not implement brand scanning, interviewing, or storage; that lives in `brand-brain`, once.
- **`proof-vault`** *(optional, when installed)* — when the meta description should carry a metric or testimonial line, pull a real, permission-cleared proof asset instead of inventing one. Synthesize inline from the brand digest only if absent.
- *(adjacent, not called — hand off if needed)* `serp-research` / SERP analysis for what's actually ranking, `brief-builder` / Content Brief Builder for the outline and intent before drafting, AEO/GEO optimizer for AI-answer extraction. This skill assumes the draft and keyword already exist.

---

## How a run works

```
Step 0  Load the brand   ──► call the `brand-brain` skill (it bootstraps on first use)
Step 1  Gather inputs     ──► draft + target keyword (+ secondary keywords, URL, page type)
Step 2  Audit the draft   ──► run the TICED pass, score what's there
Step 3  Rewrite the layer ──► title · meta · H1 · headings · semantic placement
Step 4  Self-review + present (offer to save the optimized layer)
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. It returns the active brand's digest — voice adjectives, banned words, ICP + awareness tendency, offer mechanics, real proof — and the path to `brand.md`. If the brand is new, `brand-brain` bootstraps it first; **do not rewrite any title or meta until it returns.** Obey the returned voice and banned-words as hard overrides, and use only real proof in the meta description (anything unconfirmed is `[verify]`).

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user to install `brand-brain` (preferred) or give you 2 inputs inline (3 voice adjectives + banned words, and the page's ICP/awareness stage), then proceed.

### Step 1 — Gather inputs

You need, at minimum: the **draft** (paste or path) and the **primary target keyword**. Ask once for any that are missing and useful: secondary/supporting keywords, the **search intent** (informational / commercial / transactional / navigational), the **page type** (blog post / landing / product / category), and the **live URL or slug** (so you respect existing rankings and don't propose a slug change lightly). Don't block on the optional ones — proceed with sensible defaults and flag assumptions.

---

## The framework — TICED (the on-page audit-and-rewrite pass)

Run the draft through five lenses, in order. Each lens scores the current state, then drives a rewrite. TICED keeps a junior from doing the one thing that tanks on-page work: optimizing the title and metas while leaving the body's structure and semantics untouched.

| Lens | Question it answers | What it governs |
|---|---|---|
| **T — Title & meta** | Will this earn the click in the SERP? | Title tag, meta description, slug |
| **I — Intent match** | Does the page answer the query the keyword represents? | Angle, H1 promise, intro |
| **C — Content structure** | Can a skimmer and a crawler both parse it? | H1/H2/H3 hierarchy, sections, featured-snippet target |
| **E — Entities & semantics** | Does it cover the topic, not just repeat the phrase? | LSI/related terms, entities, internal-link anchors |
| **D — Distribution & density** | Is the keyword placed naturally, never stuffed? | Keyword placement, density, cannibalization check |

### T — Title tag & meta description

- **Title tag:** keep it **≤ ~60 characters / ~575px** so it doesn't truncate. Put the **primary keyword near the front**, then a differentiator or benefit (year, number, outcome, the brand's wedge). One title, written in brand voice — not a keyword glued to a brand name. Distinguish the title tag from the on-page H1; they can differ (title optimizes for the SERP, H1 for the reader).
- **Meta description:** **~150–160 characters.** It's ad copy, not a summary — lead with the payoff, include the primary keyword once (Google bolds the match), end with an implicit reason to click. Fold in a **real proof point** from `brand-brain` / `proof-vault` when it fits ("trusted by 10k+ stores" only if true; else `[verify]`). Google rewrites metas often — write the best one anyway; it sets the framing.
- **Slug:** short, lowercase, hyphenated, keyword-bearing, no stop-word filler. **Only propose changing a live slug if the gain clearly beats the redirect cost** — flag it, don't silently rename.

### I — Intent match

Name the **search intent** behind the keyword and check the draft delivers it. A transactional query answered with a 2,000-word think-piece won't rank no matter how clean the title is. The **H1 and the first 100 words must make the same promise the title made** (message match) and confirm to the reader they're in the right place. If the draft's angle fights the intent, say so — that's a body problem, not a meta problem.

### C — Content structure

- **One H1**, containing the primary keyword naturally (or a close variant).
- **H2s for major sections, H3s nested beneath** — a logical, scannable outline a crawler reads as a topic map. No skipped levels (no H3 without a parent H2), no headings used for styling.
- Work the **primary keyword or a clear variant into 2–3 headings**, secondary keywords into others — naturally, never forced.
- Identify one section to shape as a **featured-snippet / "People Also Ask" target**: a question-form H2 answered in a tight 40–60-word paragraph or a list/table directly under it.
- Flag walls of text: long sections want sub-headings, lists, or a table so both skimmer and crawler can parse them.

### E — Entities & semantics

Search engines rank on topic coverage, not phrase repetition. Build a short **semantic map**, then place terms where they're missing:

- **LSI / related terms** — the words that co-occur with the topic (e.g. for "abandoned cart": *recovery email, checkout, conversion rate, win-back, cart value*). Place them in body copy and subheads where they fit the sentence.
- **Entities** — the named things the topic implies (tools, standards, competitors, methods). Naming them signals genuine coverage.
- **Questions** — the sub-questions a searcher also asks; each can become an H2/H3.
- **Internal-link anchors** — note 2–4 places where a descriptive, keyword-relevant internal link belongs (suggest the anchor text and the kind of page; don't invent URLs).

Place terms **only where they read naturally.** A term forced into a sentence hurts more than a term left out.

### D — Distribution & density

- **Placement beats count.** The primary keyword should land in: title, meta, H1, the first 100 words, at least one H2, and the conclusion — then wherever else it's natural. Hitting those positions matters far more than a target percentage.
- **Density as a guardrail, not a goal:** keep the exact-match phrase in a natural band (roughly **0.5–1.5%**) and lean on variants/synonyms for the rest. If a rewrite trips a banned word or reads stuffed, the rewrite is wrong.
- **Cannibalization check:** if the brand likely has another page targeting the same keyword, flag it — two pages fighting for one query split the rankings. Recommend consolidating or re-targeting one, don't optimize both into the same fight.

---

## Output

Default output is a single, paste-ready block. Lead with the rewrites, then the reasoning.

```
## On-page SEO — [page / target keyword]
Brand: [slug, via brand-brain]  ·  Intent: [type]  ·  Page type: [type]

### Title tag        [chars]
[recommended title]  (alt: [one alternate])
### Meta description  [chars]
[recommended meta — real proof or [verify]]
### Slug
[recommended-slug]   [⚠ change vs. keep note if live]

### Heading structure  (before → after)
H1: [optimized H1]
  H2: …
    H3: …
[★ = featured-snippet target]

### Semantic / LSI placements
| Term/entity | Where to add it | Why |
### Internal-link anchors
| Anchor text | Link to (page type) |

### Score & fixes
TICED: T_/I_/C_/E_/D_  — top 3 fixes that move the needle
[⚠ flags: thin body / intent mismatch / cannibalization / weak offer]
```

Keep a **before → after** for the title, meta, and H1 so the user sees exactly what changed and why. If the draft is too thin to rank regardless of on-page work, say so plainly and hand off to the brief/SERP skills.

**Persist on request:** offer to save the optimized layer to `./seo/[slug]-onpage.md` (or a path the user gives). Never write inside the skill folder, and never edit the draft file in place unless the user asks.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No title or meta before `brand-brain` returns. Its voice + banned-words override every SEO instinct here.
- **Optimize the existing draft — don't ghost-write it.** If the body can't rank, flag it; don't hide a thin article behind a strong title.
- **Intent over phrase.** Match what the searcher wants; a perfectly placed keyword on the wrong intent still loses.
- **Placement beats density.** Hit the high-value positions; treat density as a guardrail, never a quota.
- **Topic coverage, not repetition.** Win on entities and semantics, not on saying the keyword more times.
- **Readers first, crawlers second.** Every rewrite must read naturally to a human; if it doesn't, it's wrong.
- **Truth in the meta.** Real proof or `[verify]` — never fabricate a stat, customer count, or claim to earn the click.

## What Not to Do

- Don't write titles/metas/headings before `brand-brain` returns the active brand.
- Don't reimplement brand scanning/interviewing/storage — call `brand-brain`.
- Don't write the article from scratch, plan a topic cluster, or run a SERP scrape — hand off to the sibling skills.
- Don't keyword-stuff, force LSI terms into unnatural sentences, or use a banned word to hit a keyword.
- Don't exceed the title (~60 char) or meta (~160 char) limits, or skip heading levels.
- Don't silently change a live slug, or optimize two cannibalizing pages into the same fight.
- Don't invent proof, stats, or competitor claims to make the meta more clickable.

## Quality checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded (or bootstrapped) before any rewrite?
- Voice + banned-words honored; meta proof is real (or `[verify]`)?
- Title ≤ ~60 chars, keyword near front, brand voice — not keyword + brand name?
- Meta ~150–160 chars, keyword once, leads with payoff, real proof if any?
- Intent named and the H1 + intro deliver it (message match with the title)?
- One H1, clean H2/H3 hierarchy, ≥1 featured-snippet target, keyword in 2–3 headings naturally?
- Semantic/LSI terms + entities placed where they read naturally; internal-link anchors suggested?
- Keyword hits the high-value positions; density in a natural band; cannibalization checked?
- Before → after shown for title/meta/H1; thin-body / intent-mismatch flagged not hidden; save offered?
