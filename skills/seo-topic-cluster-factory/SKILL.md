---
name: seo-topic-cluster-factory
description: >
  Flagship orchestrator: pillar keyword + brand brief → a complete, ship-ready topic-cluster build
  kit that establishes topical authority. It chains the library's SEO skills end-to-end —
  keyword-research-clustering-suite to map the space, topic-cluster-pillar-architect to design the
  hub-and-spoke, then per spoke serp-analysis-report → content-brief-builder, and finally
  editorial-calendar-builder to sequence the whole build — so a buyer points it at one keyword and
  gets back a pillar outline, 8–12 writer-ready spoke briefs, a bidirectional internal-linking plan,
  and a publishing calendar. It does NOT re-implement any stage and does NOT manage brand context: it
  invokes the `brand-brain` skill first so every cluster decision is on-voice, ICP-relevant, and
  conversion-aware. Use whenever the user says "build a topic cluster," "topic cluster factory,"
  "full cluster for [keyword]," "pillar plus supporting briefs," "establish topical authority for X,"
  "build out a content hub," "give me the whole cluster, briefs and all," or hands over a pillar
  keyword and a brand and asks for the complete build kit — not just a keyword list, not a single
  article. It plans and briefs the entire cluster; it does not draft the articles (hand briefs to the
  blog-post engine) and it does not publish them.
---

# SEO Topic-Cluster Factory

Point it at one pillar keyword. Get back a complete cluster build kit: a pillar-page outline, 8–12 writer-ready spoke briefs, a bidirectional internal-linking plan, and an editorial calendar that sequences the build so links exist before they're needed. This is the "growth team in a box" for topical authority — the week of work a junior SEO, a strategist, and a content lead would split between them, run as one orchestrated pass.

This skill is an **orchestrator**, not a do-er. It does not invent keywords, read SERPs, or write briefs itself — it **chains sibling skills** that each already do one job well, and its value is the *run order*, the *handoff* between stages, and the *gates* that stop a weak stage from poisoning everything downstream. Every cluster decision is anchored to the real brand via `brand-brain`, so the output is differentiated and conversion-aware, never a generic high-volume keyword dump.

It plans and briefs the whole cluster. It does **not** draft the articles (that's `blog-post-drafting-engine`, fed by these briefs) and it does **not** publish.

---

## Skills this calls

In pipeline order. Each already exists — **invoke it via the Skill tool; never re-implement its work.**

1. **`brand-brain`** (required, Layer-0) — loads the active brand's voice, ICP, positioning, offer, and proof. Run first; everything downstream inherits this context.
2. **`keyword-research-clustering-suite`** — maps the keyword space around the pillar into scored clusters + a quick-win list. Defines *what to own*.
3. **`topic-cluster-pillar-architect`** — turns the chosen cluster into the hub-and-spoke blueprint: pillar page + mapped spokes, each with URL, title, target keyword/intent, and the bidirectional internal-linking logic.
4. **`serp-analysis-report`** *(per spoke + the pillar)* — reads the live top-10 for each target keyword: table-stakes vs. differentiator split, SERP features, the angle to win on.
5. **`content-brief-builder`** *(per spoke + the pillar)* — turns each keyword + its SERP report into a complete, writer-ready brief (intent, angle, heading outline, entities, word count, internal links, on-page SEO targets).
6. **`editorial-calendar-builder`** — sequences the pillar + spokes into a publishing schedule with owners, dates, and dependency order so internal links land in the right sequence.

**Optional, when installed** — fold in if present, synthesize a lightweight note inline if absent:
`aeo-geo-llm-visibility-optimizer` (make the pillar AI-Overview / LLM-citable), `on-page-seo-optimizer` (title/meta/schema polish per page), `content-qa-reviewer` (quality gate on each finished brief), `cta-variant-generator` + `proof-vault` (conversion assets the briefs reference). These are already wired *inside* the brief/SERP siblings — call them at the factory level only when the user asks for the extra polish pass.

---

## How a run works

```
Stage 0  Brand        ── brand-brain ───────────────► brand digest + brand.md path
Stage 1  Keyword map  ── keyword-research-clustering-suite ─► scored clusters + quick-wins
                                       │  [GATE: pick the cluster]
Stage 2  Architecture ── topic-cluster-pillar-architect ──► pillar + 8–12 spokes + link plan
                                       │  [HUMAN APPROVES the cluster shape]
Stage 3  Per-spoke loop (for the pillar and each spoke):
            serp-analysis-report ──► content-brief-builder
                                       │  [GATE: thin SERP? merge/cut the spoke]
Stage 4  Calendar     ── editorial-calendar-builder ─────► sequenced build schedule
Stage 5  Compile      ── assemble the build kit folder, self-review, present
```

### Stage 0 — Load the brand (always first)
**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the request and any named brand. It returns the active brand's digest — voice, banned words, ICP + awareness tendency, positioning line, offer mechanics + destination URLs, real proof — and the path to `brand.md`. If the brand is new, it bootstraps (scan + a few questions) before returning. **Do not start keyword work until it returns.** Pass this digest to every later stage so none of them re-derive brand context. Never write `brand.md` here — that is `brand-brain`'s job alone.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user to install `brand-brain` (preferred) or answer a 4-question mini-setup (what it is · ICP + awareness · offer + destination · 3 voice adjectives + banned words), then proceed. Always prefer the call.

### Stage 1 — Map the keyword space
Invoke **`keyword-research-clustering-suite`** with the pillar keyword/seed + brand digest (and any Ahrefs/Semrush/GSC export the user has). It returns scored clusters and a quick-win list. **Handoff →** carry forward the single cluster best matched to the pillar intent and ICP, plus its quick-win spokes, into Stage 2.

> **GATE — is there a cluster worth owning?** If volume is negligible, intent is purely transactional with no informational spine, or the space is owned by a wall of high-authority incumbents with no gap, say so and recommend a different pillar or an AEO-only play. Don't manufacture a 12-spoke cluster out of three keywords.

### Stage 2 — Design the architecture
Invoke **`topic-cluster-pillar-architect`** with the chosen cluster + brand digest. It returns the pillar page plan and 8–12 mapped spokes (each: URL, working title, target keyword, intent) plus the bidirectional internal-linking logic. **Handoff →** this is the spec the per-spoke loop and the calendar both execute against.

> **HUMAN APPROVAL POINT.** Present the cluster shape — pillar + spoke list + link map — and get a yes before spending the per-spoke loop. This is the cheap moment to cut a weak spoke, merge two overlapping ones, or re-aim the pillar. Approving here is approving the *scope of the build*.

### Stage 3 — SERP → brief, per spoke (and the pillar)
For the pillar and **each** approved spoke, in dependency order (pillar context first where the architect flagged it):
1. Invoke **`serp-analysis-report`** for that keyword → table-stakes vs. differentiator, SERP features, the winning angle.
2. **Handoff →** feed that SERP report + brand digest + the spoke's role in the cluster straight into **`content-brief-builder`** → a complete writer-ready brief (the architect's internal-link map is passed in so each brief's links point at real sibling spokes, not placeholders).

> **GATE — thin or mismatched SERP.** If a spoke's SERP shows the keyword is actually the same intent as another spoke (cannibalization), or the top 10 is all one different format, or there's no realistic gap → flag it: **merge** it into its sibling, **re-scope** the angle, or **cut** it and note why. Update the link map so no brief links to a dropped page. A factory that ships 11 honest briefs beats one that ships 12 with a dud.
>
> **REVISE LOOP (optional, when `content-qa-reviewer` is installed).** Run each finished brief through `content-qa-reviewer`. On **Revise**, feed its specific notes back into `content-brief-builder` for that spoke and re-run — max one extra pass per brief, then surface the unresolved flags to the human rather than looping forever.

### Stage 4 — Sequence the build
Invoke **`editorial-calendar-builder`** with the full spoke set, the architect's dependency/link order, the brand digest, and the user's cadence (ask if unknown — e.g. "2 posts/week"). It returns a publishing schedule with owners, due-dates-before-publish-dates, and the order that guarantees a link's target page exists before the linking page goes live. **Handoff →** the calendar is the last artifact in the kit.

### Stage 5 — Compile the build kit
Assemble every stage's output into one folder (below), run the Quality checklist, and present a short index: what the cluster is, the pillar, the spoke count, where the briefs live, and the recommended publish order. Offer the optional polish passes (AEO on the pillar, on-page SEO targets, QA) if not already run.

---

## Bundled deliverable

Save the whole kit to a **project-relative** path (leading `./` = the user's CWD / project, never the skill folder):

```
./seo-clusters/<cluster-slug>/
  00-cluster-overview.md      # pillar keyword, chosen cluster, ICP fit, the win thesis, spoke index
  01-keyword-map.md           # from keyword-research-clustering-suite (clusters + quick-wins)
  02-cluster-architecture.md  # from topic-cluster-pillar-architect (pillar + spokes + link map)
  briefs/
    pillar-<slug>.md          # pillar brief (serp-analysis-report → content-brief-builder)
    spoke-01-<slug>.md ... spoke-NN-<slug>.md   # one brief per surviving spoke
  03-internal-linking-plan.md # the bidirectional link map, reconciled after any merge/cut
  04-editorial-calendar.md    # from editorial-calendar-builder (sequenced, owned, dated)
```

`<cluster-slug>` is the kebab-cased pillar topic. Confirm the path before writing; if the user names a folder, use it. One run = one cluster kit.

---

## Principles

- **Brand-brain first, always.** No keyword, SERP, or brief work before `brand-brain` returns. Its voice and banned-words are hard overrides; only its real proof is usable.
- **Orchestrate, don't re-implement.** Every stage is a sibling skill invoked via the Skill tool. The factory owns the *order, the handoffs, and the gates* — never the inner work.
- **Carry context forward; never re-derive it.** Pass the brand digest and the cluster spec into each stage so no sibling re-scans the brand or re-decides the architecture.
- **Honest cluster math.** 8–12 spokes only if 8–12 honestly exist. Cut, merge, or re-scope thin spokes at the gate — don't pad to hit a number.
- **Links before pages.** The calendar sequences so every internal link's target exists before the linking page publishes.
- **Truth discipline.** Every unconfirmed volume, difficulty, or ranking number is marked `[verify]`. No invented competitors, SERP features, or proof — if a sibling couldn't read live data, say so.
- **One human gate that matters.** The architecture approval (Stage 2) is where scope is set cheaply. Respect it; don't burn the per-spoke loop before it.

## What not to do

- Don't write any output before `brand-brain` (or the documented fallback) returns the active brand.
- Don't re-implement keyword research, SERP reading, brief-building, or calendaring inline — invoke the sibling. If a sibling is missing, say so and offer to install it; don't silently fake its output.
- Don't draft or publish the articles — this skill stops at writer-ready briefs + calendar. Hand briefs to `blog-post-drafting-engine`.
- Don't pad the cluster to 8–12, link to spokes that got cut, or let two spokes cannibalize the same keyword.
- Don't skip the Stage-2 human approval to "save time" — a wrong architecture wastes the whole loop.
- Don't invent metrics, competitors, or proof; don't write `brand.md` yourself.
- Don't loop the QA revise step more than once per brief — escalate unresolved flags to the human.

## Quality checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded (or bootstrapped) before any stage ran?
- Each pipeline stage was an actual Skill-tool invocation of the named sibling, fed the prior stage's output (not re-derived)?
- Stage-1 cluster gate applied (is this space worth owning?) and Stage-2 architecture **approved by the human** before the per-spoke loop?
- Every surviving spoke has both a SERP report and a complete, writer-ready brief; every cut/merged spoke is removed from the link map?
- Internal-linking plan is bidirectional and references only pages that still exist in the kit?
- Calendar sequences so each link's target publishes before its source; cadence + owners set?
- Brand voice + banned-words honored across all briefs; only real proof used; every unconfirmed number marked `[verify]`?
- Kit saved to a `./`-relative path with the full folder structure; overview index presented with the recommended publish order and offered next step (drafting)?
