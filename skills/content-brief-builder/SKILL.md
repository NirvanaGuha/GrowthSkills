---
name: content-brief-builder
description: >
  Turns a target keyword plus SERP/research data into a complete editorial brief — the single
  hand-off a writer needs to draft a winning article without re-deciding strategy. It pins down
  search intent and the SERP's dominant content type, sets the differentiated angle (how this beats
  the top ten, not just matches them), builds a heading-by-heading outline mapped to the questions
  searchers actually ask, lists the entities/subtopics the page must cover for topical completeness,
  recommends a defensible word count from the SERP, and maps internal links + the on-page SEO targets
  (title, meta, H1, URL). It does NOT manage brand context itself — it calls the `brand-brain` skill
  for voice, ICP, offer/proof, and positioning so the brief points the writer at the brand's real
  story, and it calls sibling skills (proof-vault, cta-variant-generator) for the assets the article
  will need. Use whenever the user says "build a content brief," "write a brief for [keyword]," "SEO
  brief," "editorial brief," "brief the writer," "turn this SERP research into a brief," "outline this
  article," or hands over a keyword + SERP notes and asks what to write. It produces the plan, not the
  draft — drafting is the blog-post engine's job.
---

# Content Brief Builder

Give it a keyword and what's ranking; get back a brief a junior writer can execute into a senior article. The brief makes every strategic decision *once* — intent, angle, structure, coverage, length, links, SEO targets — so the writer spends their energy on prose, not on re-litigating strategy mid-draft. A great brief is the difference between an article that ranks and an article that merely exists.

This skill plans; it does not draft. It will not write the body copy, and it will not invent the SERP — if no real SERP/research data is provided, it says so and offers to run the upstream research first, rather than guessing what's ranking.

---

## Skills this calls

- **`brand-brain`** (required) — resolves and loads the active brand's voice, banned words, ICP + awareness tendency, offer/destinations, positioning, and real proof. The brief steers the writer toward the brand's real story; it does not re-derive brand context here.
- *(optional, when installed / when the brief naturally needs them)*
  - **`serp-research` / SERP Analysis Report** — if no SERP data was provided, this is the upstream step; call it (or ask the user to) before briefing.
  - **`topic-cluster-pillar-architect`** — to slot this article into its cluster and pull the right internal-link targets (pillar ↔ spokes).
  - **`proof-vault`** — to pull real stats/quotes/case studies the article should cite (so the brief names *which* proof goes *where*).
  - **`cta-variant-generator`** — to specify the closing CTA in the brand's voice and offer.
  - **`positioning-messaging-architect`** / **`icp-persona-builder`** — already folded into `brand.md` via brand-brain; call directly only for deeper pillars/persona language.

Synthesize a section inline only when a needed component is unavailable.

---

## How a run works

```
Step 0  Load the brand   ──► call the `brand-brain` skill (it bootstraps on first use)
Step 1  Gather inputs    ──► keyword + SERP/research data (or trigger upstream research)
Step 2  Build the brief  ──► run the IDEALS framework, section by section
Step 3  Self-review against the checklist, then present and offer to persist
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the request and any named brand. It returns the digest — voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning line, ICP + awareness tendency — and the `brand.md` path. The brief obeys voice + banned-words as hard overrides, points the writer only at **real** proof (anything else marked `[verify]`), and frames the angle through the brand's positioning. **Do not build the brief until brand-brain returns.**

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user to install `brand-brain` or answer a 4-question mini-setup (what it is · ICP + awareness · offer + destination · 3 voice adjectives + banned words), then proceed.

### Step 1 — Gather the inputs

You need two things before you can brief: the **target keyword** and **real SERP/research data** (the top-ranking pages' structure, formats, gaps, and common patterns). If the user hands over a SERP research dossier, use it. If they hand over only a keyword, **do not invent the SERP** — offer to run `serp-research` (or ask them to paste the SERP) first. Also capture, if available: secondary keywords, the cluster/pillar this belongs to, and any deadline or length constraint.

---

## The IDEALS framework

Every brief answers six questions, in order. Earlier answers constrain later ones — get intent wrong and the whole brief is wrong.

| Letter | Question the brief answers | Output |
|---|---|---|
| **I — Intent** | What does this searcher actually want, and where are they in the journey? | Intent type + awareness stage + the job-to-be-done |
| **D — Differentiated angle** | Why will *this* page win, not just match the SERP? | A one-sentence angle + the specific wedge over the top ten |
| **E — Entities & coverage** | What must the page cover to be topically complete? | Entities/subtopics + must-answer questions (PAA/related) |
| **A — Architecture** | How is it structured so it's scannable and answer-ready? | Heading-by-heading outline with intent-per-section notes |
| **L — Links & assets** | What does it connect to and cite? | Internal links (anchor + target) + proof + CTA |
| **S — SEO targets** | What are the exact on-page metadata + specs? | Title, meta, H1, URL slug, word count, primary/secondary KWs |

### I — Intent (decide this first)

Classify the keyword's dominant intent from the SERP, not from the words alone — **the SERP is the ground truth for intent.** If the top ten are listicles, Google has decided this is informational-list; a 3,000-word ultimate guide will not rank there.

| Intent | SERP signal | What the brief demands |
|---|---|---|
| Informational | guides, how-tos, listicles, definitions | Teach completely; answer-first; lower commercial CTA |
| Commercial | comparisons, "best/top," reviews, alternatives | Decision-support; honest comparison; mid-funnel CTA |
| Transactional | product/pricing/category pages | Conversion-led; thin on theory; strong primary CTA |
| Navigational | brand/login/known-destination pages | Usually don't brief content for these |

Pin the **awareness stage** (from the brand's ICP tendency + the keyword) and the **JTBD** — what the reader is hiring this article to do. These set the CTA ceiling later (don't ask a problem-aware reader to buy).

### D — Differentiated angle (the part juniors skip)

A brief that just re-skins the SERP produces a page that ranks #11. Use the research dossier's **gaps** and **common patterns** to choose a wedge:

- **Cover what every result omits** — the gap they all share (a step, an objection, a real number, a use case).
- **Beat them on format** — original data, a real example, a tool/template, a decision table, expert POV — where the SERP is all thin rehash.
- **Frame through the brand's positioning** — the angle should let the brand's real differentiator and proof show up *naturally*, not as a bolt-on pitch. Mark the moment in the outline where the brand's POV earns its place.

Write the angle as **one sentence the writer can hold in their head**: *"Unlike the listicles, this is the only guide that [wedge], proven with [real proof]."* If you can't find a genuine wedge, say the topic may not be worth writing — don't manufacture a fake differentiator.

### E — Entities & coverage (topical completeness)

List the **entities, subtopics, and terms** the page must mention to be recognized as comprehensive (pull from the SERP's shared vocabulary, related searches, and the brand's domain — not a generic LSI dump). Then list the **must-answer questions** (People-Also-Ask + the real questions the ICP asks). Coverage is a floor, not a keyword-stuffing quota: every entity must earn a place in the outline.

### A — Architecture (the outline)

Build a **heading-by-heading outline (H1 → H2 → H3)** where every section has a job:

```
## [H2 working heading]
   Intent of this section: [what the reader gets here]
   Must cover: [entities/questions assigned to this section]
   Proof/asset: [stat, quote, example, or "[verify]"]  ·  Internal link: [anchor → target]
   Target length: [~words]
```

Rules: lead with an **answer-first** opening for informational/featured-snippet plays (a 40–55-word direct answer the page can win the snippet with); order sections by searcher logic, not by what's easy to write; assign each must-answer question to exactly one section; note where a table, list, or image belongs. Map the closing CTA section to the brand's offer and awareness ceiling.

### L — Links & assets

- **Internal links:** specific anchor text → specific destination URL, pulled from the cluster (call `topic-cluster-pillar-architect` if present). Always include the path *up* to the pillar and *across* to sibling spokes. Note 2–5 high-value internal links minimum.
- **Proof:** name the real stat/quote/case study per section (call `proof-vault`); anything unconfirmed is `[verify]` — never a placeholder number presented as fact.
- **CTA:** specify the closing call to action (call `cta-variant-generator`) matched to the article's intent and the reader's awareness stage.

### S — SEO targets

Spell out the exact specs so the writer (or the On-Page SEO skill) has no guesswork:

- **Primary keyword** + 2–4 secondary/semantic variants and where each lands.
- **SEO title** (≤~60 chars, keyword near the front, on-voice).
- **Meta description** (≤~155 chars, promises the payoff + a reason to click).
- **H1** (can differ from the title; reads naturally).
- **URL slug** (short, keyword, lowercase-hyphenated).
- **Word count** — derived from the SERP's ranking range, not a wish. State it as a target range with the rationale ("top results run 1,800–2,400; target ~2,200 to match depth without padding").
- **Schema/format flags** if relevant (how-to, FAQ, comparison table for snippet).

---

## Output shape

```
# Content Brief — [keyword]   ·   Brand: [slug, via brand-brain]
Intent: [type · awareness stage · JTBD]
Angle: [one-sentence differentiated angle]
Word count: [range + rationale]  ·  Primary KW: […]  ·  Secondary: […]

## SEO targets
SEO title · Meta · H1 · URL slug · schema/format flags

## Entities & must-answer questions
[coverage list]  ·  [PAA / ICP questions]

## Outline (H1 → H2 → H3, with intent + coverage + proof + link + length per section)
…

## Internal links
[anchor → URL]  ×N

## Proof to cite   ·   ## Closing CTA
[real proof per section, [verify] flags]   ·   [CTA + destination]

## Writer notes
voice reminders · banned words · what to avoid · open [verify] items
```

Keep it tight enough to act on and complete enough to never need a second conversation.

---

## Persistence

If the user wants it saved, write the brief to `./briefs/[slug]-[keyword].md` (create `./briefs/` if needed) — **never** inside the skill folder, and never touch `brand.md`. Offer it; don't assume. A saved brief is the artifact the drafting engine reads next.

---

## Principles

- **Brand-brain first.** No brief before brand-brain returns; its voice + banned-words override everything here.
- **The SERP decides intent and length.** Match the content type and depth Google is already rewarding; don't fight the SERP with a format it isn't ranking.
- **Every brief needs a wedge.** A re-skin of the top ten is a #11 page. If there's no honest differentiator, say so.
- **Plan, don't draft.** This skill makes strategic decisions and hands them off; it does not write the article.
- **Coverage is a floor, not a stuffing quota.** Entities earn their place in the outline or they're cut.
- **Truth only.** Real proof, real numbers, or `[verify]`. Never a fabricated stat or differentiator in a brief that a writer will treat as fact.
- **One decision, one place.** Every choice the writer would otherwise re-make mid-draft is made here, once.

## What not to do

- Don't build a brief before `brand-brain` returns the active brand.
- Don't invent the SERP — if there's no real research, run/request `serp-research` first.
- Don't reimplement brand resolution/scanning/interviewing — call `brand-brain`.
- Don't write the article body, headlines as final copy, or the meta as deathless prose — this is a plan.
- Don't dump generic LSI keywords or pad word count; don't bolt the brand pitch onto an informational page.
- Don't fabricate proof, customers, stats, or differentiators; don't present `[verify]` items as confirmed.

## Quality checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded (or bootstrapped) before anything else?
- Real SERP/research data used (not invented); intent + length derived from the SERP?
- Intent, awareness stage, and JTBD pinned — and the CTA ceiling matches the awareness?
- A genuine one-sentence differentiated angle stated, framed through the brand's real positioning?
- Outline assigns every must-answer question and every entity to exactly one section, with intent + length per section?
- Internal links (anchor → real URL) and proof named per section, with `[verify]` on anything unconfirmed?
- SEO targets complete: title, meta, H1, slug, primary/secondary KWs, word-count range + rationale?
- Voice + banned-words honored; persistence to `./briefs/` offered (never the skill folder, never `brand.md`)?
```
