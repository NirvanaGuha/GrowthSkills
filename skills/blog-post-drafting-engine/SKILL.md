---
name: blog-post-drafting-engine
description: >
  Turns an APPROVED editorial brief into a complete, publish-ready article draft — answer-first
  intro, the brief's exact H2/H3 architecture, fully written body, conclusion, and the on-page meta
  (title, description, slug). Pure execution against the plan: it follows the brief's intent, angle,
  outline, coverage entities, internal-link map, and SEO targets literally instead of re-deciding
  strategy mid-draft, so a junior operator ships a senior article and never faces a blank page. It
  does NOT manage brand context itself — it calls the `brand-brain` skill for voice, banned words,
  ICP + awareness, offer/destinations, positioning, and real proof, and it calls sibling skills
  (proof-vault for the cited stats, cta-variant-generator for the closing CTA, editorial-style-guide
  for house mechanics) rather than redoing their work. Use whenever the user says "draft the post,"
  "write the article," "turn this brief into a draft," "execute the brief," "write from the brief at
  [path]," "fill in the outline," or hands over an approved brief and asks for the full draft. It
  writes the draft; it does not invent the SERP, rebuild the brief, or do the post-draft SEO/AEO
  rewrite or editorial review — those are sibling skills.
---

# Blog Post Drafting Engine

Hand it an approved brief; get back a full article that already says what the brief promised — in the brand's voice, citing only real proof, structured exactly as the outline dictates. This skill is the **execution layer** of the content pipeline: every strategic decision (intent, angle, structure, coverage, length, links, SEO targets) was made once in the brief, so the draft spends its energy on prose and evidence, not on re-litigating strategy.

It drafts; it does not plan. It will not invent the SERP, override the brief's angle, fabricate a stat, or do the post-draft optimization pass. If the brief is missing or not approved, it stops and routes back — a great draft off a bad plan is still a bad article.

---

## Skills this calls

- **`brand-brain`** (required) — resolves and loads the active brand's voice adjectives, banned words, ICP + awareness tendency, offer mechanics + destination URLs, positioning, and real proof. The draft obeys voice + banned-words as hard overrides and uses only real proof; brand context is never re-derived here.
- *(optional, when installed / when the draft naturally needs them)*
  - **`content-brief-builder`** — the **upstream** step. If no approved brief exists, the draft can't start; offer to build one (or have the user paste it) first.
  - **`proof-vault`** — pull the exact real stat/quote/case-study the brief assigns to each section, with its permission status. Anything unconfirmed stays `[verify]` — never a placeholder number presented as fact.
  - **`cta-variant-generator`** — write the closing CTA in the brand's voice + offer at the right awareness ceiling, instead of inventing one.
  - **`editorial-style-guide`** — apply house mechanics (capitalization, number/date format, product-name casing, banned phrases) so the draft is consistent on the first pass.
- *(handoff — downstream, do not run them here)* `on-page-seo-optimizer`, `aeo-geo-llm-visibility-optimizer`, and the article reviewer take the finished draft next. This skill drafts to spec and stops.

Synthesize a section inline only when a needed component is unavailable.

---

## How a run works

```
Step 0  Load the brand   ──► call the `brand-brain` skill (it bootstraps on first use)
Step 1  Load + gate brief ──► require an APPROVED brief; refuse to draft without one
Step 2  Draft to spec    ──► run the DRAFT framework, section by section, off the outline
Step 3  Self-review against the checklist, then present and offer to persist
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the request and any named brand. It returns the digest — voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning line, ICP + awareness tendency — and the `brand.md` path. Obey voice + banned-words as hard overrides, cite only the returned real proof (mark anything else `[verify]`), and let the brand's POV surface where the brief marks it. **Do not draft until brand-brain returns.**

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user to install `brand-brain` or answer a 4-question mini-setup (what it is · ICP + awareness · offer + destination · 3 voice adjectives + banned words), then proceed. Always prefer the call.

### Step 1 — Load the brief and gate on approval

The brief is the contract. Read it from the path the user gives (typically `./briefs/[slug]-[keyword].md`) or from what they paste. **This skill refuses to draft without an APPROVED brief** — that gate is intentional, not pedantry:

- **No brief at all** → offer to run `content-brief-builder` (which itself needs SERP research first), or ask the user to paste one. Don't draft from a bare keyword.
- **Brief exists but isn't marked APPROVED** → say so and route to the reviewer; don't draft an unapproved plan.
- **Brief approved** → extract its fields: intent, differentiated angle (the one-sentence wedge), heading-by-heading outline with per-section intent + assigned must-cover entities/questions, coverage list, internal-link map (anchor → URL), proof-per-section assignments, and the SEO targets (title, meta, H1, slug, word count, primary/secondary KWs). The draft must honor every one.

---

## The DRAFT framework

Execute the brief in five passes. The brief already decided *what*; DRAFT decides *how the prose lands* — and it never silently changes the plan.

| Letter | Pass | What it produces |
|---|---|---|
| **D — Direct answer first** | Write the intro | A hook + a 40–55-word direct answer to the head query (snippet/AI-extractable), framed by the brief's angle |
| **R — Run the outline verbatim** | Build the body | Each H2/H3 from the brief, fully written, in the order the brief set — no added or dropped sections |
| **A — Anchor every claim** | Place proof + links | The assigned real proof in its section (via `proof-vault`); internal links on the brief's exact anchor text → URL |
| **F — Finish with the offer** | Write the conclusion + CTA | A payoff that restates the transformation, then the closing CTA at the right awareness ceiling (via `cta-variant-generator`) |
| **T — Tune to brand + spec** | Final voice/SEO pass | Voice + banned-words enforced, meta written, word count + KW placement checked against the brief's targets |

### D — Direct answer first (the intro that earns the scroll)

Open with the reader's problem in their words (use the ICP's language from `brand.md`), then deliver the **answer-first paragraph**: a self-contained 40–55-word direct answer to the primary query, written so it can win the featured snippet and be quoted by an AI answer engine. State the article's promise (the brief's angle) in one line. No "In today's fast-paced world" throat-clearing, no defining the obvious. The intro is the contract the rest of the piece pays off.

### R — Run the outline verbatim (the body)

Write each section the brief specified, in the brief's order, at the brief's H-level. **Do not reorganize, merge, add, or drop sections** — if the outline is genuinely wrong, flag it and route back to the brief, don't quietly "improve" it mid-draft.

Per section, render the assigned job:

| Section job (from brief) | How the draft handles it |
|---|---|
| Must-answer question | Lead the section with a direct, scannable answer, then expand |
| Assigned entities/subtopics | Cover each naturally and once — coverage is a floor, not a stuffing quota |
| "Add a table/list/image here" | Build the table or list; mark `[image: …]` where one belongs |
| The brand's POV moment | Let the brand's real differentiator surface *here* (where the brief marked it), as insight — never a bolted-on pitch |

Default to scannable prose: short paragraphs (2–4 sentences), an H3 or list when the reader is comparing or stepping through, an answer-first sentence under each heading. Write at the ICP's sophistication level from `brand.md` — don't over-explain to experts or under-explain to novices.

### A — Anchor every claim (proof + links)

Every claim that *should* carry evidence gets the **specific real proof the brief assigned** to that section — call `proof-vault` for the exact stat/quote/case study and its permission status. Truth discipline is absolute: a number is real and attributable, or it is written `[verify]`. Never present an illustrative figure as fact, never invent a customer, a result, or a differentiator.

Place internal links on the brief's exact anchor text → destination URL (the path up to the pillar and across to sibling spokes). Link with descriptive anchors, not "click here." External links only to authoritative sources the brief named.

### F — Finish with the offer (conclusion + CTA)

Close by restating the transformation the reader now has — not a flabby summary of what they "learned." Then the **single closing CTA**, matched to the audience's awareness stage (a blog-post end is usually a soft, low-commitment ceiling): call `cta-variant-generator` for the on-voice, on-offer line and its destination URL from `brand.md`. One CTA, one action.

### T — Tune to brand + spec (the final pass)

- **Voice + banned words:** enforce the brand's adjectives and strip every banned word/phrase (hard override). Apply `editorial-style-guide` mechanics if present.
- **Meta:** write the title, meta description (fold in a real proof point if one fits), H1, and slug — to the brief's SEO targets, never keyword-stuffed.
- **Spec check:** hit the brief's word-count range; primary keyword in H1, intro, and one H2; secondary keywords placed naturally; every must-cover entity present; every must-answer question answered exactly once.

---

## Output shape

```
# [H1 from brief]
**Meta title:** … · **Meta description:** … · **Slug:** … · **Target word count:** [n] · [actual: n]
**Brand:** [slug, via brand-brain] · **Brief:** [path] · **Angle:** [one line]

[Intro — hook + 40–55-word answer-first paragraph + promise]

## [H2 …]
[body … with inline proof + internal links]
   ### [H3 …]

[… every section from the outline …]

## [Conclusion]
[transformation restatement]
[Closing CTA — label + microcopy + destination URL, via cta-variant-generator]

---
Draft notes: [any [verify] items · any outline issues flagged back to the brief · entities/questions coverage map]
```

Keep `[verify]` items and any flagged outline issues visible at the end so the reviewer and the user can act on them.

---

## Persistence

Offer to save the finished draft to `./drafts/[slug]-[keyword].md` (create `./drafts/` if needed) — **never** inside the skill folder, and never touch `brand.md` or the brief. Offer it; don't assume. A saved draft is the artifact the on-page SEO / AEO optimizers and the reviewer read next.

---

## Principles (Non-Negotiable)

- **Brand-brain first, brief second, prose third.** No drafting before brand-brain returns and an APPROVED brief is loaded.
- **The brief is the contract.** Follow its intent, angle, outline, coverage, links, and SEO targets literally. Flag a bad plan; don't quietly rewrite it.
- **Answer first.** Lead the piece, and each section, with a direct answer — not preamble.
- **Truth discipline is absolute.** Real, attributable proof or `[verify]`. Never invent stats, customers, results, or differentiators.
- **Coverage is a floor, not a quota.** Cover every assigned entity once and naturally; never keyword-stuff.
- **The brand's POV is insight, not a pitch.** Let the differentiator surface where the brief marked it.
- **One closing CTA, one action,** at the reader's awareness ceiling.

## What Not to Do

- Don't draft before `brand-brain` returns or without an APPROVED brief — route back instead.
- Don't reimplement brand resolution, SERP analysis, or brief-building here — call the upstream skills.
- Don't reorder, add, drop, or merge the brief's sections without flagging it back to the brief.
- Don't run the post-draft on-page-SEO/AEO rewrite or the editorial review here — those are downstream siblings.
- Don't invent proof, fabricate a `[verify]` away, or present an illustrative number as fact.
- Don't open with throat-clearing ("In today's fast-paced world"), pad to hit word count, or bolt on a sales pitch.
- Don't use emojis or exclamation marks unless the brand allows them.

## Quality checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded (or bootstrapped) before any prose?
- An APPROVED brief was loaded; the draft follows its intent, angle, and outline section-for-section (none added/dropped/reordered without a flag)?
- Intro leads with a 40–55-word answer-first paragraph and states the brief's promise?
- Every assigned proof point placed via `proof-vault` and real/attributable; everything else marked `[verify]`?
- Internal links on the brief's exact anchor text → URL; closing CTA via `cta-variant-generator` at the right awareness ceiling?
- Voice + banned-words enforced; style-guide mechanics applied; meta (title/description/H1/slug) written to the brief's SEO targets, not stuffed?
- Word count in range; primary KW in H1 + intro + an H2; every must-cover entity present and every must-answer question answered once?
- Persistence to `./drafts/` offered (never the skill folder, never `brand.md` or the brief)?
