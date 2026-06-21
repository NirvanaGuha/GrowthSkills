---
name: idea-evaluation-stress-test-suite
description: >
  Takes a list of raw ideas or a developed concept and produces a scored ranking by feasibility,
  novelty, and strategic fit; an adversarial critique that finds the kill shots before you spend
  a dollar; and a triaged backlog with Keep / Park / Kill decisions and next actions. Three modes:
  Rank (quick scoring of a batch), Stress-Test (deep adversarial attack on one idea), and Full
  (score + stress-test + backlog triage in one pass). Uses the ICE / RICE scoring family combined
  with the Six Thinking Hats framework for the adversarial pass — real, named, senior-operator
  methodology, not generic "pros and cons." Brand context from `brand-brain` ensures every idea
  is evaluated against the actual brand constraints, not hypothetical ones. Composes
  `go-no-go-gate-evaluator` for hard gate criteria, `constraint-based-ideator` to surface
  constraint-optimized alternatives before a Kill verdict, and `pre-mortem-post-mortem-generator`
  for the pre-mortem on ideas that survive. Use when the user says "evaluate these ideas,"
  "stress test this," "which idea should we run with," "score my backlog," "kill the weak ideas,"
  "pressure test this concept," "rank these campaigns," "is this worth pursuing," or submits
  a list of ideas and needs a go/park/kill triage.
---

# Idea Evaluation & Stress-Test Suite

A scoreboard and an attack dog in one. Point it at a raw idea list or a developed concept; it returns a scored ranking, an adversarial critique that finds the holes before execution does, and a triage decision for every idea in the batch. Nothing survives on enthusiasm alone.

Scoring mode is quick (a batch → ranked table in minutes). Stress-test mode is slow and deliberate (one idea → the full adversarial treatment). Full mode runs both plus a backlog triage and a pre-mortem for survivors.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's voice, ICP, offer, positioning, proof, and constraints. Every evaluation is anchored to the real brand, not a generic abstraction. Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no `brand.md` exists, ask the user for [brand name, core ICP, primary growth lever, known resource constraints (time/budget/headcount)].
- **`go-no-go-gate-evaluator`** (optional, Gate step) — when the user has explicit go/no-go criteria (launch deadline, minimum confidence level, budget ceiling), delegate the gate check here rather than re-implementing it. Call it after scoring.
- **`constraint-based-ideator`** (optional, Kill alternatives step) — when an idea scores Kill but the underlying job-to-be-done is sound, call this to generate constraint-respecting alternatives before closing the idea out.
- **`pre-mortem-post-mortem-generator`** (optional, Survivor step) — for ideas that earn a Keep verdict, call this to run the pre-mortem and surface the failure modes early, before the execution plan is locked.
- **`prioritization-framework-suite`** — if the user already has a prioritization framework in flight (RICE, ICE, Impact/Effort), defer to that skill for the scoring mechanics; don't double-implement.
- **`validity-threat-checker`** — for ideas that are framed as experiments (A/B tests, landing page experiments), call this to flag validity threats before the Keep verdict is final.

---

## How a run works

```
Step 0  Load the brand       ──► brand-brain (always first)
Step 1  Detect mode          ──► Rank | Stress-Test | Full
Step 2  Score (all modes)    ──► ICE + Strategic-Fit matrix
Step 3  Stress-Test pass     ──► Six Hats adversarial critique
Step 4  Triage decisions     ──► Keep / Park / Kill + next actions
Step 5  Compose if needed    ──► go-no-go-gate / pre-mortem / alternatives
```

---

## Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns the active brand's digest — ICP, positioning, offer mechanics, real proof, constraints, voice. Do not score or critique a single idea until it returns. Every score and every Kill verdict must be anchored to the real brand's constraints and strategic fit, not generic assumptions.

Fallback if `brand-brain` is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no `brand.md` exists, ask the user for [brand name, core ICP, primary growth lever, known resource constraints (time/budget/headcount)].

---

## Step 1 — Detect the mode

| Trigger | Mode |
|---|---|
| Batch of 3+ ideas with no "pick one" framing | **Rank** — score + triage table, no deep attack |
| Single idea, OR "stress test," "pressure test," "poke holes" | **Stress-Test** — deep adversarial treatment only |
| Single idea with "full evaluation," OR batch where one idea is clearly the candidate | **Full** — score + stress-test + backlog triage + pre-mortem offer |
| Ambiguous | Default to **Rank**; offer Stress-Test at the end on the top scorer |

---

## Step 2 — Scoring (ICE + Strategic-Fit)

Score each idea on four dimensions, each 1–10:

| Dimension | What it measures |
|---|---|
| **Impact** (I) | Magnitude of the win if it works — revenue, retention, reach, or the north-star metric from `brand-brain`. |
| **Confidence** (C) | Weight of evidence that it will actually work — past data, analogues, validated assumptions vs. pure hypothesis. Mark weak evidence `[verify]`. |
| **Ease** (E) | Inverse of effort — team capacity, tech lift, dependencies, time-to-first-result. |
| **Strategic Fit** (F) | Alignment with current brand positioning, ICP, active growth lever, and known constraints from `brand-brain`. |

**ICE-F score** = (I × C × E × F)^0.25 (geometric mean — a zero on any dimension can't be rescued by highs elsewhere).

Present as a ranked table:

```
| Rank | Idea | I | C | E | F | ICE-F | Verdict |
```

Flag any idea where C ≤ 3 with ⚠ (confidence gap — needs evidence before committing resources).

---

## Step 3 — Stress-Test (Six Hats adversarial pass)

Run for every idea flagged **Keep** in Rank mode, and for all ideas in Stress-Test / Full mode. This is the adversarial layer — the point is to find real kill shots before the team does.

**De Bono's Six Thinking Hats applied as an evaluation protocol** (not a brainstorm):

| Hat | Attack vector for ideas |
|---|---|
| **White (facts)** | What evidence actually supports this? What data would change the verdict? Name the assumption that, if wrong, makes the whole idea fail. |
| **Black (caution)** | Worst realistic outcome. Where has a similar idea failed and why? What does the opponent with maximum resources do in response? |
| **Yellow (optimism)** | Where is the upside genuinely larger than the score captured? What tailwind is the team underweighting? |
| **Red (emotion/politics)** | Which stakeholder will block this and why? Where does gut instinct diverge from the score — and is the gut right? |
| **Green (creativity)** | What's the minimum viable version that tests the core assumption in two weeks? What constraint, if lifted, makes this a 10× idea? |
| **Blue (process)** | What's the execution risk — sequencing, ownership, measurement? Is there a better idea that achieves the same outcome with less coordination cost? |

For each hat, write 2–3 crisp, pointed sentences. No padding. If a hat produces a genuine kill shot, flag it explicitly: **⛔ Kill shot: [the specific finding]**.

---

## Step 4 — Triage decisions

Every idea gets one verdict and a next action:

| Verdict | Criteria | Next action |
|---|---|---|
| **Keep** | ICE-F ≥ 6 AND no kill shot from the stress-test | Assign an owner + target date; offer pre-mortem via `pre-mortem-post-mortem-generator` |
| **Park** | ICE-F 4–5.9 OR one kill shot that is fixable with more evidence | State the specific condition that would upgrade it to Keep; set a review date |
| **Kill** | ICE-F < 4 OR a fatal kill shot with no fixable path | State the kill reason clearly; offer `constraint-based-ideator` if the JTBD is sound |

Output format:

```
## Triage decisions

| Idea | Verdict | ICE-F | Deciding factor | Next action |
```

Followed by a brief rationale (2–3 sentences) for each Keep and each Kill. Park decisions get the specific upgrade condition.

---

## Artifact persistence

Save the full evaluation to `./idea-evals/[slug]-eval.md` when the batch has ≥ 3 ideas or when the user asks for it. The slug is derived from the campaign or project name. Quick inline evaluations (1–2 ideas, Rank mode) stay inline unless asked.

---

## Principles

- **Brand-brain first.** No score is valid without the brand's real constraints, ICP, and growth lever loaded. A generic evaluation is a waste of everyone's time.
- **Geometric mean, not arithmetic.** A zero on Confidence can't be rescued by a 10 on Impact. Weak evidence stays weak regardless of how exciting the upside is.
- **Attack to find kill shots, not to be clever.** The Six Hats pass exists to find the one thing that would make the idea fail, not to generate impressive-sounding critique.
- **Kill is a service.** A clean Kill verdict with a clear reason saves more time than a vague Park. If the JTBD is sound but the idea is broken, say so and offer alternatives.
- **Compose, don't rebuild.** Gate criteria live in `go-no-go-gate-evaluator`, pre-mortem lives in `pre-mortem-post-mortem-generator`, constraint alternatives live in `constraint-based-ideator`. Call them; don't re-implement them.
- **Mark weak evidence.** Any score built on assumption rather than data gets `[verify]`. Never present a confident score on a shaky evidence base.

---

## What not to do

- Don't start scoring before `brand-brain` returns. Every score that ignores real brand constraints is fiction.
- Don't use arithmetic mean for ICE-F — it masks fatal weaknesses. Use geometric mean.
- Don't generate padded Six Hats commentary — each hat should say something that could actually change the verdict.
- Don't deliver a Kill verdict without a reason. "Doesn't fit" is not a reason.
- Don't Park everything to avoid conflict. Park means "fixable with evidence"; it's not a polite Kill.
- Don't run a stress-test on a Kill-level idea — score it, triage it, offer an alternative, move on.
- Don't invent proof points or competitive analogues. Real analogues or `[verify]`.

---

## Quality checklist

- `brand-brain` called and active brand loaded (or fallback executed) before any scoring?
- ICE-F uses geometric mean; any C ≤ 3 idea flagged with ⚠?
- Strategic Fit dimension explicitly anchored to brand's ICP and growth lever?
- Six Hats pass produced ≥ 1 genuine finding per hat (not filler); kill shots flagged ⛔?
- Every Keep verdict has an owner/date next action?
- Every Kill verdict states the deciding factor; `constraint-based-ideator` offered if JTBD is sound?
- Park verdicts specify the exact condition to upgrade, with a review date?
- `go-no-go-gate-evaluator` called if explicit gate criteria were provided?
- Artifact saved to `./idea-evals/` for batches ≥ 3 or on request?
- No invented proof — all weak evidence marked `[verify]`?
