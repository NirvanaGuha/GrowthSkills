---
name: multi-source-deep-research-report-builder
description: >
  Any topic, market, competitor, or strategic question → a fact-checked, fully cited multi-source
  research report with a clear executive summary, layered evidence hierarchy, adversarial
  verification pass, and actionable so-what synthesis. This is the library's general-purpose
  research horsepower: deeper and more rigorous than a quick web scan, more structured than
  raw notes, purpose-built to feed every downstream skill that needs credible intelligence as
  its input. Three output modes: Snapshot (rapid 3–5 source brief), Standard (10–15 source full
  report), and Deep Dive (20+ sources, multi-wave with adversarial check). Invokes brand-brain
  to anchor framing in the active brand's ICP, positioning, and competitive landscape — so
  findings land as strategy, not raw facts. Use whenever the user says "research," "deep
  dive on," "what do we know about," "compile everything on," "competitor deep-dive," "market
  landscape," "fact-check this claim," "I need a brief on," "build me a research report," or
  hands over a topic and asks for cited intelligence.
---

# Multi-Source Deep Research Report Builder

Give it a question, market, competitor, or topic. Get a structured, cited report where every claim traces to a source, every weak signal is flagged, and the final section tells you what to do with it — anchored to your brand's actual ICP and competitive context.

This skill produces reports, not raw search dumps. It runs a named methodology (TRAAP + adversarial verification), levels sources by authority, and produces a deliverable another skill — or a human decision-maker — can act on immediately.

---

## Skills this calls

- **`brand-brain`** (required first) — loads the active brand's ICP, positioning, competitive landscape, and banned claims. Research framing is shaped to what matters *for this brand*, not a generic audience. Does not re-derive brand context.
- **`competitive-intelligence-dossier`** — if the research subject is a specific competitor or competitive landscape, call this instead of duplicating competitor profiling here. This skill synthesizes multi-source intelligence; CI Dossier owns the deep competitor teardown.
- **`voice-of-customer-mining-pipeline`** — when the research corpus includes reviews, forum threads, support tickets, or interview transcripts, delegate VoC extraction there; fold the returned theme hierarchy into this report's Evidence section.
- **`analytical-reasoning-toolkit`** — for strategic questions (market entry, pricing tier, positioning pivot), call this after the research pass to run consequence mapping, EV calc, and bias audit on the top findings.
- **`content-brief-builder`** — if the output is destined for a content asset, hand this report's findings to content-brief-builder as the research input rather than repeating web searches.

---

## How a run works

```
Step 0  Load brand context  ──► call brand-brain (ICP, positioning, competitive lens)
Step 1  Scope the question  ──► clarify scope, depth, output mode
Step 2  Source planning     ──► map source types; assign authority tiers
Step 3  Evidence gathering  ──► multi-wave research with WebSearch + WebFetch
Step 4  Adversarial pass    ──► stress-test top claims; surface contradictions
Step 5  Synthesize          ──► structure the report; write executive summary last
Step 6  Save artifact       ──► ./research/<slug>.md
```

### Step 0 — Brand context (always first)

Invoke `brand-brain` before any research. Use the returned digest to: (a) anchor the executive summary to the brand's ICP pain/gain vocabulary; (b) pre-populate the competitive lens with the brand's known rivals; (c) flag any brand claims that need fact-checking against what the scan turns up. If brand-brain is absent, read `~/.brandbrain/brands/.active` and its `brand.md`; if none, ask the user for ICP description and top 2–3 competitors before proceeding.

### Step 1 — Scope the question

Before researching, resolve three things:

| | Question | Default if not stated |
|---|---|---|
| **Scope** | Exact research question (not just topic) | Restate the topic as the most useful research question |
| **Depth** | Snapshot / Standard / Deep Dive | Standard |
| **Deliverable intent** | Who reads it and what decision it feeds | State as "to inform" + most likely downstream skill |

If the question is underspecified (e.g., "research push notifications"), narrow it to the most actionable framing given the brand context and confirm with the user in one sentence before proceeding.

---

## The research methodology: TRAAP + Adversarial

**TRAAP** is the evidence quality lens applied per source. Every cited source gets a TRAAP label inline.

| Criterion | What it checks | Mark |
|---|---|---|
| **T**imeliness | Is it current enough for the claim? (< 12 months for market data; < 24 for evergreen) | ✓ / ⚠ dated |
| **R**elevance | Does it directly address the research question? | ✓ / ⚠ tangential |
| **A**uthority | Who published it? (primary > analyst > trade press > blog > forum) | tier 1–4 |
| **A**ccuracy | Is it verifiable? Are numbers traceable? | ✓ / `[verify]` |
| **P**urpose | What's the author's intent? (inform vs. sell vs. advocate) | note bias |

**Adversarial pass** (Step 4): after gathering evidence, run a structured challenge:
1. Pick the 3–5 claims most load-bearing for the report's conclusions.
2. For each: search for a counter-claim or disconfirming source.
3. If found: present both sides, weight by TRAAP authority, and note the tension.
4. If not found after a genuine search: say so — that's evidence too.

Never suppress disconfirming evidence. If the data contradicts the user's assumption, surface it clearly before the synthesis.

---

## Output modes

### Snapshot (3–5 sources, ~400–600 words output)
Quick calibration for a meeting, a brief, or a fast editorial decision. Skips multi-wave sourcing; skips adversarial pass on low-stakes claims. Best for: "what is X," orientation questions, or feeding a downstream skill that just needs the landscape.

### Standard (10–15 sources, ~800–1200 words output)
Full TRAAP review, single adversarial pass on top 3 claims, executive summary + findings sections + so-what. Default mode. Best for: strategic decisions, content briefs, positioning reviews, competitor snapshots.

### Deep Dive (20+ sources, ~1500–2500 words output)
Multi-wave sourcing: Wave 1 (broad scan) → Wave 2 (targeted fill-in on gaps) → Wave 3 (adversarial). Primary sources required (original data, transcripts, filings) alongside secondary. Adversarial pass on all load-bearing claims. Best for: market entry, M&A diligence context, major positioning pivots, annual planning inputs.

---

## Report structure

```markdown
# [Research Question]
**Brand lens:** [active brand slug] · **ICP:** [one line] · **Date:** [today] · **Mode:** [Snapshot/Standard/Deep Dive]

## Executive Summary
[3–5 bullets: the most actionable findings. Written last. No hedging — state what is true, 
what is uncertain, and what the brand should do about it.]

## Findings
### [Theme 1]
[Claim with inline citation and TRAAP tier. ⚠ flags for dated or tangential sources. 
`[verify]` for unconfirmed numbers.]

### [Theme 2] ...

## Contradictions & Open Questions
[Tensions between sources. Disconfirming evidence. Claims that could not be verified.
Anything the user should not treat as settled.]

## Source Index
| # | Source | Type | Authority Tier | TRAAP notes |
|---|---|---|---|---|

## So What (Brand Action Layer)
[Translate findings to implications for the active brand's ICP, offer, positioning, or content.
Specific. No generic advice. Point to the downstream skill that should act on each implication.]
```

---

## Source authority tiers

| Tier | Examples | Weight |
|---|---|---|
| 1 — Primary | Original research, SEC/Companies House filings, peer-reviewed studies, official product docs, first-party interviews | Highest — use as bedrock claims |
| 2 — Analyst | Gartner, Forrester, IDC, CB Insights, G2 crowd data, Statista with sourced data | High — cite with the underlying survey date |
| 3 — Trade press | Industry publications, known editorial outlets with editorial standards | Medium — triangulate with tier 1/2 |
| 4 — Aggregated / blog | Marketing vendor blogs, LinkedIn posts, forum threads, Reddit, review sites | Low — useful for VoC signal and trend direction, not for stating facts |

Never cite a tier-4 source for a factual claim without tier-1/2 corroboration. Use tier-4 for VoC texture and delegate the extraction to `voice-of-customer-mining-pipeline` when the corpus is large.

---

## Principles

- **Brand-brain first.** Research framing is anchored to the brand's ICP and competitive reality before the first search runs.
- **Question over topic.** Always restate the input as a specific, answerable research question before gathering sources.
- **TRAAP every claim.** No fact lands in the report without a source, a tier, and a currency check.
- **Surface disconfirming evidence.** Adversarial pass is mandatory on Standard and Deep Dive. Finding nothing is itself a result — state it.
- **So What is the real deliverable.** Findings without a brand-anchored action implication are trivia. The So What section is what makes research a strategic input.
- **Compose, don't duplicate.** Competitor profiling → `competitive-intelligence-dossier`. VoC extraction → `voice-of-customer-mining-pipeline`. Strategic reasoning → `analytical-reasoning-toolkit`. This skill orchestrates and synthesizes; it does not reimplement those functions.

## What Not to Do

- Don't start research before `brand-brain` returns the active brand digest.
- Don't state a number without a citation; unverified figures get `[verify]`.
- Don't suppress contradictions or inconvenient findings — surface tensions clearly.
- Don't produce a source dump; always synthesize into themes with a So What.
- Don't run Deep Dive on a vague question — scope it first or default to Standard.
- Don't reimplement competitor teardowns, VoC theme extraction, or reasoning frameworks — call the sibling skill.
- Don't exceed the scope: this skill produces research reports, not content drafts, landing pages, or campaign briefs. Name the downstream skill that should act on the output.

## Quality Checklist

- `brand-brain` called; ICP + competitive lens loaded before any search?
- Research question stated precisely before sourcing began?
- Every cited claim has a source, a TRAAP tier, and a currency flag?
- At least 3 load-bearing claims stress-tested in the adversarial pass (Standard/Deep Dive)?
- Contradictions and disconfirming evidence surfaced — not buried?
- Source Index complete with tier labels?
- Executive Summary written last, states what is true/uncertain/actionable?
- So What section names brand-specific implications and the downstream skill to act on each?
- Artifact saved to `./research/<slug>.md`?
