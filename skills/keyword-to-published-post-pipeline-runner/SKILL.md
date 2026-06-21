---
name: keyword-to-published-post-pipeline-runner
description: >
  Flagship orchestrator that turns one target keyword into a publish-ready, SEO- and AI-optimized
  blog post by chaining the library's content skills end-to-end — SERP research → brief → draft →
  on-page SEO → answer-engine optimization → QA → publish handoff — into ONE bundled deliverable a
  buyer can run cold. It does NOT re-implement any stage: it invokes each sibling skill via the Skill
  tool and manages the handoff, gates, and revision loops between them. It loads brand context FIRST
  by calling `brand-brain`, so every stage is on-voice, ICP-aligned, and uses only real proof. Use
  when the user says "take this keyword and write me a post," "run the full content pipeline,"
  "keyword to published post," "draft and publish an article on X," "give me a finished blog post for
  [keyword]," "do the whole SEO content workflow," or hands over a keyword (with optional intent /
  priority / brand) and wants a finished article rather than one stage of it. For a single stage
  (just research, just a draft, just SEO), call that stage's skill directly — this runs the whole line.
---

# Keyword-to-Published-Post Pipeline Runner

Point it at a keyword. Get back a publish-ready post — title, meta, body, schema, internal links, and a QA verdict — produced by running the library's best content skills in order and carrying each stage's output into the next. This is the "growth team in a box" for SEO content: it replaces a junior marketer's week of research, briefing, drafting, optimizing, and QA with one supervised run.

This skill is a **conductor, not a performer.** It does not write briefs, draft prose, or optimize pages itself — every one of those is a sibling skill that already exists and does the job better. The runner's whole value is the **chain**: loading the brand once, ordering the stages, passing the right artifact from each stage to the next, gating on quality, looping when a stage comes back weak, and pausing for the human at the two moments that matter (brief approval, publish). Run cold by a buyer, it should feel like handing a keyword to a senior content lead and getting a finished post back.

---

## Skills this calls

In invocation order. Each is invoked with the **Skill tool** — never reimplemented here.

1. **`brand-brain`** (required, Layer-0) — loads the active brand's voice, banned words, ICP, offer + destinations, positioning, and real proof. Every later stage inherits this context.
2. **`serp-analysis-report`** — analyzes the live SERP for the keyword: top-result structure, intent, content gaps, common patterns.
3. **`content-brief-builder`** — turns the SERP dossier into a structured brief: angle, outline, differentiation map, internal-link plan, SEO targets. **Human approval gate here.**
4. **`blog-post-drafting-engine`** — drafts the full article strictly to the approved brief and brand voice.
5. **`on-page-seo-optimizer`** — optimizes the draft: title tag, meta description, H-structure, keyword placement, internal/external links, image alt, slug.
6. **`aeo-geo-llm-visibility-optimizer`** — restructures passages for AI answer engines (AI Overviews, ChatGPT, Perplexity) and emits FAQ/HowTo/Article schema.
7. **`content-qa-reviewer`** — final quality gate: returns **Approve / Revise / Reject** with specific fixes. Drives the revision loop.

**Optional, when installed and the keyword/run calls for it (invoke; synthesize inline only if absent):**
- `keyword-research-clustering-suite` / `topic-cluster-pillar-architect` — when the keyword needs validation or cluster placement before research.
- `headline-hook-generator` — stronger title/H2 options fed into the draft or SEO stage.
- `cta-variant-generator` — the end-of-post conversion CTA, on-voice.
- `proof-vault` — real proof/stat microcopy when the draft needs evidence.
- `de-slop-humanize-pass` — a humanizing pass if QA flags AI-tells.
- `content-repurposer-atomizer` — post-publish atomization into social/email (offered, not auto-run).
- `wordpress-publisher` (or the user's CMS skill) — the actual publish action at the handoff.

---

## How a run works

```
Stage 0  Brand          brand-brain ──► brand digest + brand.md path  (always first)
Stage 1  Research        serp-analysis-report ──► SERP dossier (gaps, intent, patterns)
Stage 2  Brief           content-brief-builder ──► structured brief   ── HUMAN APPROVES ──┐
Stage 3  Draft           blog-post-drafting-engine ──► full draft (to brief + voice)       │
Stage 4  On-page SEO     on-page-seo-optimizer ──► draft + title/meta/links/slug           │
Stage 5  AEO/GEO         aeo-geo-llm-visibility-optimizer ──► extractable passages + schema │
Stage 6  QA gate         content-qa-reviewer ──► Approve / Revise / Reject ───loop on Revise┘
Stage 7  Publish handoff assemble bundle ──► HUMAN APPROVES ──► publish (or hand to CMS skill)
```

### Stage 0 — Load the brand (always first)
**Invoke `brand-brain`** (Skill tool, `skill: brand-brain`), passing the request and any named brand. It returns the active slug, the `brand.md` path, and a digest (voice adjectives, banned words, ICP + awareness tendency, offer mechanics + destination URLs, positioning, real proof) plus pointers to companion files (`competitors.md`, `proof.md`, `style-guide.md`). **Produce nothing downstream until it returns.** Pass this digest into every subsequent stage so each skill stays on-voice and uses only real claims.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user to install `brand-brain` (preferred) or answer a 4-question mini-setup (what it is · ICP + awareness · offer + destination URL · 3 voice adjectives + banned words), then proceed. Never write `brand.md` yourself — always prefer the call.

### Stage 1 — SERP research
Invoke `serp-analysis-report` with the keyword + brand digest. **Handoff out:** a SERP dossier — search intent, the shape of the top results, the gaps none of them cover, and the patterns all of them share. Carry this verbatim into Stage 2; do not summarize away the gaps (they are the differentiation fuel).

### Stage 2 — Brief  ·  *human approval gate*
Invoke `content-brief-builder` with the SERP dossier + brand digest. **Handoff out:** a structured brief — angle, working title, section outline, differentiation map, internal-link targets, primary/secondary keywords, and SEO targets. **Stop and show the user the brief.** This is the cheapest place to course-correct: an approved brief is the contract the draft is held to. Do not draft until the human approves (or auto-approve only if the user explicitly said "run the whole thing without stopping").

### Stage 3 — Draft
Invoke `blog-post-drafting-engine` with the **approved** brief + brand digest. It writes to the brief literally — angle, outline, differentiation, SEO targets. **Handoff out:** the full draft. (If the brief skill exposes a reviewer/approval flag, respect it: a brief marked not-approved should block drafting.)

### Stage 4 — On-page SEO
Invoke `on-page-seo-optimizer` with the draft + brief SEO targets + brand digest. **Handoff out:** the draft plus the meta layer — title tag, meta description, slug, H-structure, keyword placement, internal/external links, image alt text. This stage edits the existing post; it does not rewrite from scratch.

### Stage 5 — Answer-engine optimization
Invoke `aeo-geo-llm-visibility-optimizer` with the optimized draft + brand digest. **Handoff out:** passages restructured into self-contained, quotable, entity-rich chunks, a question-led structure, and ready-to-ship FAQ/HowTo/Article schema. This makes the post extractable by AI Overviews / ChatGPT / Perplexity without changing its on-voice meaning.

### Stage 6 — QA gate  ·  *the loop*
Invoke `content-qa-reviewer` with the full assembled post (body + meta + schema) + brief + brand digest. It returns one of:
- **Approve** → proceed to Stage 7.
- **Revise** → it returns specific fixes. **Route each fix to the stage that owns it** (voice/structure → drafting engine; meta/links → SEO; extractability/schema → AEO/GEO; AI-tells → `de-slop-humanize-pass`), apply, then **re-run QA.** Cap at **2 revision loops**; if it still fails, stop and surface the blockers to the human rather than shipping a weak post or looping forever.
- **Reject** (wrong angle / off-brief / thin) → do not patch. Return to Stage 2, revise the brief with the human, and re-run from there.

### Stage 7 — Publish handoff  ·  *human approval gate*
Assemble the **bundled deliverable** (below) and present it. **Get explicit human go-ahead to publish.** Then either hand the bundle to the user's CMS/publisher skill (e.g. `wordpress-publisher`) if installed and authorized, or deliver the publish-ready package for the user to paste. Confirm the saved path and, if the post went live, the URL. Offer (don't auto-run) `content-repurposer-atomizer` to atomize the post into social/email.

---

## Bundled deliverable

Save to a **project-relative** path (leading `./` = the user's CWD/project, **not** the skill folder):
`./content/<keyword-slug>/` containing:

- `00-brief.md` — the approved brief (the contract)
- `01-serp-dossier.md` — SERP analysis (intent, gaps, patterns)
- `02-post.md` — the final publish-ready article (body + title + meta + slug)
- `03-schema.json` — FAQ/HowTo/Article schema from the AEO/GEO stage
- `04-qa-report.md` — the QA verdict + what changed across revision loops
- `RUN.md` — one-page run log: brand slug, stages run, gates passed, loop count, open `[verify]` items, publish status/URL

If the user only wants the post, deliver `02-post.md` + its meta inline and still save the bundle. Always tell the user the absolute path written.

---

## Principles

- **Brand-brain first, always.** No research, no draft, no copy before `brand-brain` returns. Its voice + banned-words override every stage.
- **Conduct, don't perform.** Each stage is a sibling skill invoked via the Skill tool. Never reimplement research, briefing, drafting, SEO, AEO, or QA inline — that defeats the library and drifts from each skill's craft.
- **Carry artifacts forward intact.** Each stage's output is the next stage's input. Pass the real artifact (dossier, brief, draft), not a lossy paraphrase. The brief is the contract; the draft is held to it; QA checks against it.
- **Gate at the two moments that matter.** Human approves the brief (cheap to fix) and approves publish (irreversible). Everything between runs autonomously unless the user opts into full auto.
- **Loop on weak output, but bounded.** Revise routes fixes to the owning stage and re-runs QA; cap at 2 loops, then escalate to a human. Reject reopens the brief. Never ship a post QA hasn't approved.
- **Truth discipline.** Use only the brand's real proof and real differentiators; mark every unconfirmed number or claim `[verify]`. Never invent stats, customers, quotes, or rankings — and never claim the post is "indexed" or "ranking," which the runner cannot verify.
- **One keyword, one post per run.** Cluster work routes to `topic-cluster-pillar-architect`, not here.

## What not to do

- Don't produce any stage's output before `brand-brain` returns the active brand.
- Don't reimplement a stage — call its skill. If a skill is missing, say so and synthesize a minimal stand-in only as a flagged fallback.
- Don't skip the brief-approval gate or the publish-approval gate (unless the user explicitly asked for an unattended run).
- Don't draft against an unapproved or rejected brief; don't publish a post QA returned as Revise/Reject.
- Don't loop QA more than twice — escalate the blockers to the human instead.
- Don't write to the skill folder or to a brand's `brand.md`; bundle to the project-relative `./content/<keyword-slug>/`.
- Don't claim live/indexed/ranked status; report only what you can verify (saved path, and the publish action if a CMS skill confirmed it).
- Don't invent proof, internal-link targets, competitors, or schema fields; carry only what the brand/SERP stages actually provided.

## Quality checklist (self-review before declaring the run done)

- `brand-brain` called first; brand resolved (or bootstrapped); digest passed into every stage?
- All seven stages invoked **as skills** (not reimplemented), in order, with the real artifact handed off at each step?
- Brief approved by the human before drafting; publish approved by the human before going live?
- Draft demonstrably matches the approved brief (angle, outline, differentiation, SEO targets)?
- On-page SEO complete (title, meta, slug, headings, links, alt) and AEO/GEO applied (extractable chunks + valid schema)?
- `content-qa-reviewer` returned **Approve** (not Revise/Reject); revision loops ≤ 2; any unresolved blockers surfaced to the human?
- Voice + banned-words honored throughout; only real proof used; everything unconfirmed marked `[verify]`?
- Bundle saved to `./content/<keyword-slug>/`; absolute path reported; publish status stated honestly (no unverifiable "indexed/ranking" claims)?
