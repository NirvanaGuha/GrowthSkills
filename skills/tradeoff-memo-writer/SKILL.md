---
name: tradeoff-memo-writer
description: >
  Takes any decision with competing priorities — channel, budget, pricing, team structure, tool
  consolidation, launch timing, build-vs-buy, or any fork in the road — and produces a tight
  one-page structured memo: a crisp framing of the decision, 2–4 discrete options with their
  real tradeoffs (cost, risk, opportunity cost, reversibility), a reasoned recommendation, and
  a mandatory dissenting view so the decision-maker can't skip the counter-argument. Uses the
  Hammond-Keeney-Raiffa "Even Swaps" framework as the evaluation spine — options are compared
  on a shared set of objectives, and swaps are made explicit so the tradeoff becomes arithmetic,
  not intuition. Output is board-safe: one page, no filler, no false balance. The skill loads
  brand/company context through brand-brain so the recommendation is calibrated to the brand's
  actual constraints (budget posture, ICP, competitive position, growth stage) — not generic
  advice. Use whenever the user says "help me decide," "write a tradeoff memo," "decision memo,"
  "options doc," "we can't agree on," "pros and cons," "build vs buy," "should we do X or Y,"
  "document this decision," or hands over a decision that has no obvious single answer.
---

# Tradeoff Memo Writer

Most decisions stall because the options aren't written down in the same format, on the same axes. This skill fixes that: it takes a messy fork in the road and produces a structured one-page memo that a team can actually decide from — not a pro/con list that ends in "it depends," but a document with a recommendation and a dissent.

The spine is the **Hammond-Keeney-Raiffa "Even Swaps" framework** (from *Smart Choices*, 1998): identify the objectives that matter, score each option against each objective, and make any remaining tradeoffs explicit as arithmetic exchanges — "we'd accept X loss on cost to gain Y on speed." When a swap is unconscionable, you've found your constraint. That discipline turns implicit gut-feel into a defensible, reviewable record.

---

## Skills this calls

- **`brand-brain`** (required first) — loads the active brand's context: budget posture, ICP, competitive position, growth stage, known constraints. The recommendation is only credible if it's calibrated to the actual company, not a hypothetical one. Does not implement brand resolution or scanning.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no `brand.md` exists, ask the user for [decision context: company stage, budget posture, primary growth constraint, and any hard limits (legal, technical, political) on the options].
- **`go-no-go-gate-evaluator`** — optional; call if the user needs a binary pass/fail verdict per option against a defined gate criterion (e.g., "must hit ROAS > 2x before we scale") rather than a comparative memo.
- **`decision-log-brief-writer`** — optional; call after the memo is approved to write the structured decision log entry and Notion-ready row. Composes naturally: memo → approved → log.
- **`initiative-business-case-writer`** — optional; call if the chosen option requires a formal business case with CFO-defensible financials before proceeding.
- **`risk-log-builder`** — optional; call if any option's downside risk needs a full register (likelihood, impact, mitigation, owner) beyond the one-line risk note in the memo.
- **`pre-mortem-post-mortem-generator`** — optional; call to stress-test the recommended option before committing — assume it fails in 12 months, ask why.

---

## How a run works

```
Step 0  Load brand context   ──► call brand-brain; receive digest + constraints
Step 1  Clarify the decision ──► confirm decision statement, objectives, constraints
Step 2  Frame Even Swaps     ──► identify shared objectives + weight signals
Step 3  Write the memo       ──► structured one-pager (see template below)
Step 4  Self-review          ──► check against quality checklist; present
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). Receive the active brand's digest — growth stage, budget posture, ICP, competitive position, known constraints, banned framings. Use this to calibrate the recommendation: what's cheap for a Series B is prohibitive for a bootstrapped SMB; what's reversible for a platform team is irreversible for a two-person ops pod.

Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no `brand.md` exists, ask the user for [decision context: company stage, budget posture, primary growth constraint, and any hard limits on the options].

Do not write the memo before this step resolves.

### Step 1 — Clarify the decision

Before scoring anything, confirm three things — ask in a single batch if any are missing:

1. **Decision statement** (one sentence: "We are deciding whether to…"). If the user hasn't written one, draft it from context and confirm.
2. **Hard constraints** — anything that eliminates an option outright (legal, budget ceiling, timeline). Flag these before scoring; a violated constraint isn't a tradeoff, it's a disqualifier.
3. **Objectives that matter** — what does a good decision optimize for? Solicit or infer from context: typical axes include speed, cost, reversibility, customer impact, team bandwidth, strategic optionality, compliance. Cap at 5. Weight them loosely (must-have / nice-to-have / tie-breaker).

Never invent options. Work with what the user provides; add a "do nothing / status quo" option only when it's a genuine live choice.

### Step 2 — Apply Even Swaps

For each option × objective intersection:
- Score qualitatively (Strong / Neutral / Weak) or quantitatively if the user provides numbers.
- Identify **dominated options**: if one option is no better than another on any objective, flag it early and set it aside (state why).
- Identify **even swaps**: for the remaining close options, state the explicit exchange — e.g., "Option B costs $18K more than Option A annually; in exchange it saves ~3 hrs/week of ops time. At a $50/hr ops cost, the swap breaks even at year 2." Make every swap arithmetic where possible; flag `[verify]` on any number you haven't been given.
- This arithmetic is the memo's core intellectual contribution. Don't skip it.

---

## Memo template

Save to `./decisions/[slug]-tradeoff-memo.md` when the user asks or when the memo is confirmed.

```markdown
# Tradeoff Memo — [Decision Title]
**Date:** [today]  |  **Author:** [user or "drafted by AI"]  |  **Status:** Draft / Final

## Decision statement
[One sentence. Starts with "We are deciding…"]

## Context (2–3 sentences max)
[Why this decision is live now; what changed; what's at stake if it drifts.]

## Hard constraints
[Bullet list. Anything that disqualifies an option outright. If none: state "None identified."]

## Objectives (what a good decision optimizes for)
| Objective | Weight |
|---|---|
| [Speed / Cost / Reversibility / …] | Must-have / Nice-to-have / Tie-breaker |

## Options
### Option A — [Name]
**What it is:** [1–2 sentences]
**Scores:** [objective-by-objective, qualitative or quantitative]
**Risk / downside:** [one-liner; link to risk-log if expanded]
**Reversibility:** [High / Medium / Low + why]

### Option B — [Name]
[same structure]

### Option C — [Name, if present]
[same structure]

## Even Swaps
[For each close-option pair: state the explicit exchange arithmetic. Mark [verify] on unconfirmed numbers.]

## Recommendation
**Recommended option:** [Name + one-sentence rationale anchored to the weighted objectives]
**Conditions:** [What has to be true for this to remain the right call — e.g., "assuming ops headcount stays flat"]
**Next step:** [Single concrete action + owner + due date]

## Dissenting view
**The case for [Option X] instead:**
[2–4 sentences. Steelman the alternative. Name the specific objective or assumption the dissent pivots on. Do not dismiss it — a reader who weights that objective differently should understand why this is a live disagreement.]
```

---

## Even Swaps craft (what makes this hard to fake)

**Five objectives maximum.** More than five and the matrix becomes noise. If the user lists eight objectives, group and consolidate before scoring — then ask them to confirm the groupings.

**Dominance check before swaps.** Don't math your way through a dominated option. If Option C is weaker than Option A on every objective, say so in one sentence and remove it. The memo is about the live dilemma.

**Reversibility is always an objective.** Growth decisions almost always underweight reversibility. Surface it explicitly, especially for tool choices, team hires, and pricing changes. The Even Swaps framework calls this "preserving future decision freedom."

**One recommendation, not a range.** A memo that says "Option A or B depending on priorities" is not a memo; it's a spreadsheet. The skill commits to one pick and puts the alternative in the Dissent. The decision-maker can override — that's their job — but they should have to.

**Dissent is not cosmetic.** The dissenting view must name the specific objective or assumption the recommendation deprioritizes. "Some people might prefer Option B" is not a dissent. "If speed-to-market is the primary constraint and not cost, Option B is the right call — it ships 6 weeks earlier at a $12K premium" is a dissent.

**`[verify]` every unconfirmed number.** If the user gives you costs, use them. If you're inferring costs from context, mark `[verify]`. Never invent benchmark numbers to make the arithmetic tidy.

---

## Principles

- **Brand-brain first.** No recommendation before the brand's constraints are loaded. A recommendation uncalibrated to the company's actual budget posture, ICP, and stage is decoration.
- **Even Swaps, not gut feel.** Every close-option comparison gets explicit arithmetic. "It's complicated" is not a tradeoff memo entry.
- **One recommendation, with a real dissent.** Fence-sitting is not neutral — it's a failure to do the job. The dissent makes the memo honest, not the recommendation hedged.
- **Reversibility is always on the objectives list** unless there's an explicit reason to exclude it.
- **Hard constraints disqualify; they don't score.** Don't waste the reader's time scoring an option that violates a hard constraint. Flag it, remove it, move on.
- **Truth over tidy.** Real numbers or `[verify]`. Never round a rough estimate to a suspiciously clean one.

---

## What not to do

- Don't write the memo before brand-brain returns. Context-free recommendations are the thing this skill exists to replace.
- Don't produce a pro/con list and call it a tradeoff memo. The Even Swaps arithmetic is the structural difference.
- Don't invent options the user hasn't surfaced — except "do nothing / status quo," and only when it's a genuine live choice.
- Don't score a dominated option. Flag dominance, remove it, explain why in one sentence.
- Don't hedge the recommendation into a range. One pick; the dissent carries the alternative.
- Don't write a dissent that doesn't name the specific objective or assumption it pivots on. Generic "on the other hand" language is not a dissent.
- Don't mark a number `[verify]` and then use it as if it were confirmed in the same sentence.
- Don't exceed one page for the memo body (excluding the Even Swaps working). The point is a document the team can read before a 30-minute meeting.

---

## Quality checklist (self-review before presenting)

- brand-brain called and brand context loaded (or fallback path followed) before any recommendation written?
- Decision statement is one sentence starting "We are deciding…"?
- Hard constraints listed and any disqualified options removed before scoring?
- Objectives capped at 5, loosely weighted (must-have / nice-to-have / tie-breaker)?
- Dominated options identified and removed with a one-line explanation?
- Even Swaps section includes explicit arithmetic for every close-option pair; all unconfirmed numbers marked `[verify]`?
- Single recommendation with a condition statement and a next step (owner + date)?
- Dissenting view names the specific objective or assumption it pivots on — not generic hedging?
- Memo body fits one page (excluding working)?
- Voice and banned-words from brand-brain honored throughout?
