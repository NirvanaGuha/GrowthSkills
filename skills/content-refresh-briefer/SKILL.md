---
name: content-refresh-briefer
description: >
  Turns an underperforming URL — its current content plus its GSC data — into a surgical refresh brief
  a writer can execute in about two hours to recover lost traffic. Not a rewrite-from-scratch plan:
  it diagnoses WHY the page is decaying (intent drift, lost SERP features, thin sections, decayed
  freshness, cannibalization, missing entities) and returns an exact what-to-ADD / what-to-CUT /
  what-to-REWRITE / what-to-KEEP action list, prioritized by traffic upside vs. effort, mapped to the
  specific queries that should recover. It does NOT manage brand context itself — it calls the
  `brand-brain` skill to load the active brand's voice, ICP, positioning, proof, and banned words so
  the refreshed page stays on-message, and calls `proof-vault` and `cta-variant-generator` when the
  brief needs fresh proof or a stronger conversion ask. Use whenever the user says "refresh this post,"
  "this page is losing traffic," "content refresh brief," "update this article," "why is this URL
  decaying," "what should I add to rank again," or hands over a URL + GSC export and asks how to win the
  traffic back. It briefs the refresh — it does not write the article or publish it.
---

# Content Refresh Briefer

Hand it a slipping URL, its current content, and its Search Console data — get back a refresh brief that says exactly what to add, cut, rewrite, and leave alone, in priority order, mapped to the queries that should bounce back. The goal is a **two-hour recovery**: the highest-leverage edits to an existing asset, not a blank-page rewrite. Refreshing what already has authority and history beats starting over almost every time.

This skill briefs; it does not draft the article or publish it. If the page is beyond saving (wrong intent entirely, dead topic, better consolidated elsewhere), it says so and recommends consolidate / redirect / retire instead of papering over a lost cause with edits.

---

## Skills this calls

- **`brand-brain`** (required) — resolves and loads the active brand's voice, ICP, positioning, proof, and banned words. This skill does not implement brand scanning/interviewing/storage; that lives in `brand-brain`, once.
- **`proof-vault`** (when installed) — pull real, tagged proof (stats, customers, case-study lines) to slot into ADD/REWRITE items instead of inventing it.
- **`cta-variant-generator`** (when installed) — when the page's conversion ask is weak or off-stage, get a stronger on-voice CTA for the brief instead of hand-waving "improve the CTA."
- *(optional, when present)* a keyword/decay-audit sibling for cannibalization context. Synthesize inline if absent.

---

## How a run works

```
Step 0  Load the brand   ──► call the `brand-brain` skill (it bootstraps on first use)
Step 1  Gather the inputs ─► URL/current content · GSC queries+pages · target keyword · age
Step 2  Diagnose          ─► run the DECAY-7 scan → the dominant decay cause(s)
Step 3  Brief             ─► ADD / CUT / REWRITE / KEEP, prioritized, mapped to queries
Step 4  Self-review, present, offer to save
```

### Step 0 — Load the brand (always first)
**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the request and any named brand. It returns the digest — voice adjectives, banned words, positioning line, ICP + awareness tendency, real proof, offer + destinations — and the `brand.md` path. If the brand is new, it bootstraps first. **Do not brief until it returns.** Obey voice + banned-words as hard overrides; use only real proof (else `[verify]`).

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if none, ask the user to install `brand-brain` or answer a 3-question mini-setup (what it is · ICP + awareness · 3 voice adjectives + banned words), then proceed.

### Step 1 — Gather the inputs (ask only for what's missing)
You need four things; ask only for the ones not supplied:
1. **The URL + its current content** (paste, file, or fetch the live page).
2. **GSC data** — for this page: top queries with clicks, impressions, avg position, and CTR; ideally a before/after window (e.g. last 3mo vs. prior 3mo) showing the decline. The brand's `pushengage-analytics`/GSC tooling can pull this if available.
3. **The target keyword / primary intent** the page is meant to own.
4. **Page age / last meaningful update** — freshness matters for time-sensitive queries.

If GSC data is missing, say so plainly: the brief is far weaker without it (you're guessing at which queries to recover). Offer to proceed on content + keyword alone, clearly flagged as lower-confidence.

---

## The DECAY-7 diagnosis (the named method)

Don't refresh on vibes. Score the page against seven decay causes, evidenced from GSC + the content, and let the dominant one(s) drive the brief. A refresh that fixes the wrong cause wastes the two hours.

| # | Decay cause | Tell (from GSC + content) | Default fix lever |
|---|---|---|---|
| 1 | **Intent drift** | SERP now answers a different job than the page (queries shifted; impressions up, clicks/position down) | REWRITE intro + reframe; restructure to current intent |
| 2 | **Lost SERP feature** | Page used to win a featured snippet / PAA / video and lost it | ADD a crisp 40–55-word definition block, a direct-answer H2, FAQ schema-ready Q&As |
| 3 | **Thin / outcompeted sections** | Top results cover sub-topics the page omits; impressions on queries the page barely addresses | ADD the missing sections/entities mapped to those impression-only queries |
| 4 | **Freshness decay** | Time-sensitive query, stale year/stats/screenshots, "best X 2024" in a 2026 SERP | REWRITE dates/stats/examples; ADD a "what changed in [year]" note; refresh visuals |
| 5 | **Cannibalization** | Two+ of the brand's URLs split impressions/position for the same query | KEEP the winner, consolidate/redirect or differentiate the loser (flag — do not silently merge) |
| 6 | **CTR collapse at stable position** | Holds position 3–8 but CTR well below the position benchmark | REWRITE title + meta to match query + intent; add number/benefit; tighten the promise |
| 7 | **Authority / proof gap** | Ranks 8–20, content fine, but lacks depth, proof, originality the top results have | ADD real proof (call `proof-vault`), original data/examples, expert framing, internal links from strong pages |

**Position → opportunity read:** pos 5–15 with strong impressions = the fattest two-hour win (small lift, big click gain). Pos 1–3 losing clicks = CTR/feature problem, not content. Pos 20+ = depth/authority problem, maybe not a two-hour job — say so.

---

## The brief (what you return)

A refresh brief is an **edit list against the existing page**, not an outline. Lead with the diagnosis, then the prioritized actions, each tagged and mapped to the query it serves.

```
## Refresh Brief — [page title / URL]
Brand: [slug, via brand-brain]  ·  Target keyword: […]  ·  Est. effort: ~2h
Diagnosis (DECAY-7): [dominant cause(s) + the GSC evidence in one line each]
Verdict: REFRESH  |  CONSOLIDATE→[winner URL]  |  REDIRECT  |  RETIRE

### Action list (priority order — upside vs. effort)
| P | Action | Type | Where (section/H2) | Recovers (query) | Note |
|1| … | ADD/CUT/REWRITE/KEEP | … | "[query]" @ pos X | proof/voice note |

### ADD — new sections / entities / proof (mapped to impression-only queries)
### CUT — outdated, off-intent, or cannibalizing passages (say why)
### REWRITE — title, meta, intro, stale stats/dates (with the new angle)
### KEEP — what's working; don't touch (protect the ranking equity)

### Title & meta (if CTR is a cause)  — [new title] / [new meta], why it lifts CTR
### Conversion ask  — [on-stage CTA, via cta-variant-generator if weak]
### Internal links to add  — [from strong pages → this URL, with anchor]
### Success metric & re-check  — [the 2–3 queries to watch; re-pull GSC in 4–6 weeks]
```

Rules: every ADD/REWRITE maps to a real query or entity from the data. Title/meta only when CTR is an actual cause. Protect what ranks — name it in KEEP so the writer doesn't gut a winning section. Real proof only (else `[verify]`).

---

## Persistence

Offer to save the brief to `./briefs/[slug]-[page-slug]-refresh.md` (or the user's chosen path) — never inside the skill folder, never overwriting `brand.md`. The brief is the executable artifact the writer (or `article-writer`) picks up next, and the re-check note makes the loop closeable.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No brief before `brand-brain` returns. Its voice + banned-words override everything here.
- **Diagnose, then prescribe.** DECAY-7 names the cause before any edit is proposed — no shotgun rewrites.
- **Refresh the asset; don't restart.** Protect existing ranking equity (KEEP), edit surgically, scope to ~2 hours.
- **Every edit earns its place.** Each ADD/REWRITE maps to a real query/entity from the GSC + SERP evidence.
- **Honest verdict.** If it's cannibalization, consolidation, or a dead topic, say redirect/consolidate/retire — don't bill a doomed refresh.
- **Real proof only.** Pull from `proof-vault`; never invent stats, customers, or differentiators — mark unconfirmed `[verify]`.

## What Not to Do

- Don't brief before `brand-brain` returns the active brand.
- Don't reimplement brand scanning/interviewing/storage, or re-derive voice/ICP — call `brand-brain`.
- Don't write the article or publish it — this briefs the refresh; hand off to the writer.
- Don't propose a full rewrite when a section-level refresh recovers the traffic.
- Don't silently merge cannibalizing URLs — flag the consolidation and name the winner.
- Don't rewrite the title/meta when CTR isn't the problem, or change a section that's ranking well.
- Don't proceed silently without GSC data — flag the lower confidence.

## Quality checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded (or bootstrapped) before briefing?
- DECAY-7 run, dominant cause(s) named with GSC evidence, and a clear REFRESH/CONSOLIDATE/REDIRECT/RETIRE verdict?
- Action list is ADD/CUT/REWRITE/KEEP, priority-ordered by upside vs. effort, each mapped to a query/entity?
- KEEP protects what ranks; title/meta touched only if CTR is a real cause?
- Proof pulled from `proof-vault` (or `[verify]`); CTA via `cta-variant-generator` if the ask was weak?
- Voice + banned-words honored; scope realistically ~2 hours; save offered and re-check metric set?
