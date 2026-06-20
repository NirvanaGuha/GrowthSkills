---
name: aeo-geo-llm-visibility-optimizer
description: >
  Restructures an existing article or landing page so AI answer engines — ChatGPT, Perplexity,
  Google AI Overviews / AI Mode, Claude, Gemini — can extract, quote, and cite it. Diagnoses where
  the page is invisible to retrieval-augmented answers, then rewrites passages into self-contained,
  extractable, entity-rich chunks, adds the question-led structure and schema that answer engines
  lift, and emits a prioritized fix list plus the FAQ/HowTo/schema to ship. It does NOT manage brand
  context — it calls `brand-brain` to load the active brand's voice, banned words, offer, ICP, and
  proof, so every rewrite stays on-voice and uses only real claims. Use whenever the user says
  "optimize for AI Overviews," "get cited by ChatGPT/Perplexity," "AEO," "GEO," "generative engine
  optimization," "LLM visibility," "answer engine optimization," "why isn't AI citing us," "make
  this quotable," or hands over a URL/article and asks to make it AI-extractable. It restructures
  one page for citation — it does not write a page from scratch (use the drafting engine) and it is
  not classic keyword SEO.
---

# AEO / GEO / LLM-Visibility Optimizer

Give it an existing page, get it back rewritten so an answer engine can lift a clean, self-contained passage and cite *you* as the source. This skill closes the gap between ranking and being *quoted* — the new battleground, where ChatGPT, Perplexity, and Google AI Overviews synthesize an answer and name a handful of sources instead of sending ten blue links.

It restructures one page at a time. It does not invent stats to make a claim "quotable," write a page from scratch, or chase keyword density — message-match and proof come from the brand, via `brand-brain`. If the page has no real expertise or proof to cite, it says so rather than fabricate authority.

---

## Skills this calls

- **`brand-brain`** (required) — resolves and loads the active brand: voice, banned words, ICP + awareness, offer mechanics + destination URLs, positioning, and **real proof**. This skill never re-derives brand context.
- **`proof-vault`** (optional, when installed) — pull confirmed stats, customer quotes, and named sources to make a passage citable. Absent → use only proof returned in the brand digest; everything else is `[verify]`.
- **`cta-variant-generator`** (optional) — for the single on-page CTA on a landing-page optimization. Don't hand-roll CTAs here.
- *(optional)* `competitive-intelligence-dossier` for the "who currently gets cited for this query" comparison; `editorial-style-guide` for entity naming/casing consistency.

Name what you actually called in the run.

---

## How a run works

```
Step 0  Load the brand     ──► call `brand-brain` (bootstraps on first use)
Step 1  Frame the query     ──► what AI question should this page win, and who cites it now?
Step 2  Audit with AEO-FRESH ──► score the page on the 6 extractability factors
Step 3  Restructure passages ──► rewrite to extractable chunks + question-led structure + schema
Step 4  Self-review + present ──► prioritized fix list, rewritten passages, schema, persist
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. Use the returned digest — voice, banned words, ICP + awareness, offer + destinations, positioning, real proof — as hard overrides. **Do not rewrite anything until it returns.** Mark any claim not backed by returned/`proof-vault` proof as `[verify]`.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if none, ask the user to install `brand-brain` or answer a 4-line mini-setup (what it is · ICP · offer/destination · voice + banned words), then proceed. Always prefer the call.

### Step 1 — Frame the target query

Before touching the page, name the **answer-engine question(s)** this page should win — phrased the way a person *asks an assistant*, not types into Google ("what's the best way to recover abandoned carts on Shopify?" not "abandoned cart recovery shopify"). Conversational, long-tail, intent-explicit. If the user gives a URL, fetch it; if they paste content, work from that. Optionally check who's cited today for that query (Perplexity/AI Overview) to set the bar.

---

## The AEO-FRESH framework (the audit + the fix)

Answer engines don't rank pages — they **retrieve passages, then synthesize and cite**. To get cited you must be (1) *retrievable* as a clean chunk, (2) *liftable* as a self-contained answer, (3) *trustable* as a source. AEO-FRESH scores the page on the six factors that drive all three, then each factor maps to a concrete rewrite.

| # | Factor | The question it answers | Score | Fix when low |
|---|---|---|---|---|
| **F** | **Findable chunks** | Can a passage be retrieved *out of context* — does each section stand alone? | 0–3 | Break walls of text into self-contained chunks; one idea per chunk; front-load the answer |
| **R** | **Restated question** | Does a heading/first line *restate the user's question* in natural language? | 0–3 | Convert headings to question form; lead the section with a 2–4 sentence direct answer |
| **E** | **Entities & specificity** | Are products, people, places, and numbers named explicitly (no "we," "it," "recently")? | 0–3 | Replace pronouns/vague refs with named entities; date claims; add concrete numbers |
| **S** | **Structure & schema** | Lists, tables, definitions, FAQ — and the markup (`FAQPage`/`HowTo`/`Article`) that exposes them? | 0–3 | Add scannable structures; emit JSON-LD schema matching the on-page content |
| **H** | **Hard evidence / authority** | Real stats, named sources, author expertise, citations *out* — is this quotable *and* trustworthy? | 0–3 | Attach real proof (via `proof-vault`); add author/E-E-A-T signals; cite primary sources |
| **F·R·E·S·H** | **Freshness & match** | Is it current, dated, and does it match how the question is actually asked today? | 0–3 | Add/update dates; align headings to conversational phrasing; refresh stale claims |

Score each 0–3 (0 absent · 1 weak · 2 decent · 3 best-in-class), total /18. Report the score, the two weakest factors, and lead the fix list with those.

### The Extractable Passage pattern (the core rewrite)

Every passage an engine can lift follows the same shape. This is the unit of work:

1. **Question heading** — the H2/H3 *is* the question, in the user's words.
2. **Direct answer first (the snippet).** Open with a self-contained 2–4 sentence answer that resolves the question *without* the surrounding page. This is the block that gets quoted. Lead with the conclusion, then qualify.
3. **Evidence + specifics.** Named entities, a real number or two, a named source — the why behind the answer.
4. **Detail / nuance** — edge cases, steps, exceptions, the human depth an LLM can't synthesize from thin pages.
5. **No orphan pronouns.** Each chunk re-states its subject so it survives being read alone.

> Inverted-pyramid, per section. The answer is the headline of the chunk; never bury it under a 150-word windup.

### What answer engines reward (and what they ignore)

- **Direct, declarative answers** over hedged, throat-clearing intros. Cut "In today's fast-paced world…" entirely.
- **Definitions, lists, tables, step sequences** — discrete, liftable units beat narrative prose for extraction.
- **Specific named entities + numbers** — "PushEngage recovered 18% of abandoned carts `[verify]`" beats "many users see big improvements."
- **Original data, frameworks, and first-hand expertise** — the things a model *can't* synthesize from everyone else's pages are what earn the citation.
- **Statistics + citations to primary sources** measurably lift citation rate; fluff, keyword stuffing, and pop-ups suppress it.

---

## Output

Lead with the verdict, then the fixes, then the rewritten assets. Don't dump a rewritten page with no diagnosis.

```
## AEO/GEO optimization — [page / URL]
Brand: [slug, via brand-brain]  ·  Target question(s): [conversational query]
AEO-FRESH score: __/18   (weakest: [factor], [factor])
[⚠ authority gate note, if the page has no real proof/expertise to cite]

### Priority fixes (highest leverage first)
| # | Factor | Issue | Fix | Effort |

### Rewritten passages (before → after)
[for each high-leverage section: the question heading + the extractable-passage rewrite, on-voice]

### Question-led structure
[proposed H2/H3 set as conversational questions; FAQ block of 3–6 Q&As]

### Schema to ship
```json
[ FAQPage / HowTo / Article JSON-LD matching the on-page content — no orphan schema ]
```

### Why this gets cited
[1–2 lines: which factors moved, what an engine can now lift]
```

Keep before→after tight; rewrite the passages that move the score, not the whole page. Offer to save the full report to `./reports/[slug]-aeo-[page].md` (never inside the skill folder, never touch `brand.md`).

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No rewrite before `brand-brain` returns; its voice + banned-words override everything here.
- **Optimize for being *quoted*, not just ranked.** The unit of value is the liftable passage, not the page.
- **Answer first, every section.** Front-load the self-contained answer; the inverted pyramid is per-chunk.
- **Real proof or `[verify]`.** Citability comes from *true* specifics — never invent a stat, source, or author credential to look authoritative.
- **Entities over pronouns.** Every chunk must survive being read out of context.
- **Schema must match the page.** No FAQ schema without a visible FAQ; orphan markup is a penalty risk, not a hack.
- **Restructure, don't rewrite the brand.** Preserve the page's real expertise and offer; surface it, don't replace it.

## What Not to Do

- Don't rewrite before `brand-brain` returns the active brand.
- Don't reimplement brand scanning/interviewing/storage — call `brand-brain`.
- Don't write a page from scratch (that's the drafting engine) or do classic keyword-density SEO.
- Don't fabricate stats, customers, author E-E-A-T, or "as cited by" claims to manufacture authority.
- Don't emit schema that doesn't match visible on-page content; don't keyword-stuff the new chunks.
- Don't bury the answer under a windup, or hand back narrative prose where a list/table/definition extracts better.
- Don't strip the brand's voice into flat, robotic "answer-ese" — extractable and on-voice are not in conflict.

## Quality checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded (or bootstrapped) before any rewrite?
- Target answer-engine question(s) named in conversational form?
- Page scored on all six AEO-FRESH factors with a total and the two weakest called out?
- Every rewritten passage follows the Extractable Passage pattern (question heading → answer-first → evidence → detail, no orphan pronouns)?
- Only real proof used (rest `[verify]`); authority gate raised if the page has nothing real to cite?
- Schema emitted matches visible on-page content (FAQ/HowTo/Article), nothing orphaned?
- Voice + banned-words honored; CTAs (if any) via `cta-variant-generator`?
- Prioritized fix list leads with highest-leverage items; report offered for save to `./reports/`?
