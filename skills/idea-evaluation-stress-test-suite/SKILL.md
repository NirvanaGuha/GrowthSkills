---
name: idea-evaluation-stress-test-suite
description: >
  Takes a list of raw ideas or a developed concept and produces a scored ranking by feasibility,
  novelty, and strategic fit; an adversarial critique that finds the kill shots before you spend
  a dollar; and a triaged backlog with Keep / Park / Kill decisions and next actions. Three modes:
  Rank (quick scoring of a batch), Stress-Test (deep adversarial attack on one idea), and Full
  (score + stress-test + backlog triage in one pass). Scores on a house ICE-F model — a deliberate
  four-dimension extension of Sean Ellis's ICE (canonical ICE is Impact × Confidence × Ease, no
  Strategic Fit), aggregated with a geometric mean so one weak dimension drags the score down — and
  borrows the six lenses of De Bono's Six Thinking Hats as a structured critique pass (De Bono's own
  method is parallel, non-adversarial; this skill repurposes the lenses for evaluation). For standard
  RICE/ICE/WSJF, defer to `prioritization-framework-suite`. Brand context from `brand-brain` ensures every idea
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
Step 2  Score (all modes)    ──► house ICE-F model (geometric mean, 0–10)
Step 3  Stress-Test pass     ──► Six Hats lenses as a critique checklist
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

## Step 2 — Scoring (the house ICE-F model)

**Attribution, stated plainly.** Canonical ICE (Sean Ellis) is **Impact × Confidence × Ease** — three dimensions, no Strategic Fit, usually multiplied or averaged. ICE-F is *this skill's own four-dimension extension*: it adds a Strategic-Fit dimension and aggregates with a geometric mean. It is a house model, not standard ICE. If the user wants textbook RICE/ICE/WSJF, hand off to `prioritization-framework-suite` and don't re-implement it here.

Score each idea on four dimensions, each **0–10** (0 is a real, enterable score — see why below):

| Dim | What it measures | 0 means | 5 means | 10 means |
|---|---|---|---|---|
| **Impact** (I) | Magnitude of the win if it works — revenue, retention, reach, or the north-star metric from `brand-brain`. | No measurable move on any metric that matters. | Moves a secondary metric, or the north-star metric modestly. | Materially moves the north-star metric this cycle. |
| **Confidence** (C) | Weight of evidence it will actually work — past data, analogues, validated assumptions vs. pure hypothesis. Mark weak evidence `[verify]`. | Pure hope; zero evidence and a known counter-signal. | A plausible analogue or one weak signal. | Direct prior data on this brand/audience. |
| **Ease** (E) | Inverse of effort — team capacity, tech lift, dependencies, time-to-first-result. | Blocked: a hard dependency or capacity gap makes it un-shippable now. | Doable in a normal sprint with known lift. | Trivial; ships this week with no new dependencies. |
| **Strategic Fit** (F) | Alignment with current brand positioning, ICP, active growth lever, and known constraints from `brand-brain`. | Contradicts positioning or serves a non-ICP segment. | Adjacent — neither on-strategy nor off. | Directly advances the active growth lever for the core ICP. |

**ICE-F = (I × C × E × F)^¼** — the geometric mean of the four scores.

**Why a 0 floor and a geometric mean, together.** They are the same decision. The geometric mean only earns its keep if a true 0 is reachable: score any dimension 0 and the product is 0, so **ICE-F = 0 — the kill-property literally fires.** That is the point. A blocked idea (E=0), a positioning contradiction (F=0), or a pure-hope bet with a counter-signal (C=0) collapses to 0 no matter how high the other three are. On any non-zero scores, the geometric mean still penalizes imbalance harder than an arithmetic mean would: I=9, C=2, E=8, F=8 averages to 6.75 but scores ICE-F ≈ **5.8** — the low Confidence drags it down. Use 0 only for a genuine dealbreaker; if a dimension is merely poor, score it 1–2, not 0.

Present as a ranked table:

```
| Rank | Idea | I | C | E | F | ICE-F | Verdict |
```

Flag any idea where C ≤ 3 with ⚠ (confidence gap — needs evidence before committing resources). Any dimension scored **0** is auto-routed to triage as a Kill candidate; name the dealbreaker on its row.

---

## Step 3 — Six Hats applied as a structured critique pass

Run for every idea flagged **Keep** in Rank mode, and for all ideas in Stress-Test / Full mode. The point is to find real kill shots before the team does.

**Borrowed lenses, not De Bono's format.** De Bono's Six Thinking Hats is *parallel thinking*: a whole room wears one hat at a time, in sequence, precisely to avoid adversarial debate. This skill does not run that facilitation — it lifts the six lenses and points each one at a single idea as a critique checklist, run by one evaluator. So the six categories are De Bono's; the one-person, idea-attacking application is this skill's adaptation. Keep the hats true to their meaning: White is facts, Black is caution/risk, Yellow is upside, Red is feeling/politics, Green is alternatives, Blue is process — none of them is an insult contest.

| Hat | Critical question |
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
- **ICE-F is a house model — say so.** It is a four-dimension geometric-mean extension of ICE, not canonical ICE (which is Impact × Confidence × Ease). Never present it as standard ICE; for textbook RICE/ICE/WSJF, defer to `prioritization-framework-suite`.
- **Geometric mean, and a real 0 floor.** Scores run 0–10. A genuine dealbreaker scores 0 and collapses ICE-F to 0 — that is the kill-property working as designed. Short of a dealbreaker, the geometric mean still penalizes a single low score harder than an average would, so weak evidence stays weak regardless of how exciting the upside is.
- **The Six Hats lenses are De Bono's; the application isn't.** His method is parallel thinking, not an attack. This skill borrows the six lenses as a one-person critique checklist; keep each hat true to its meaning and use it to find the one thing that would make the idea fail, not to generate impressive-sounding critique.
- **Kill is a service.** A clean Kill verdict with a clear reason saves more time than a vague Park. If the JTBD is sound but the idea is broken, say so and offer alternatives.
- **Compose, don't rebuild.** Gate criteria live in `go-no-go-gate-evaluator`, pre-mortem lives in `pre-mortem-post-mortem-generator`, constraint alternatives live in `constraint-based-ideator`. Call them; don't re-implement them.
- **Mark weak evidence.** Any score built on assumption rather than data gets `[verify]`. Never present a confident score on a shaky evidence base.

---

## What not to do

- Don't start scoring before `brand-brain` returns. Every score that ignores real brand constraints is fiction.
- Don't use arithmetic mean for ICE-F — it masks fatal weaknesses. Use the geometric mean.
- Don't pass ICE-F off as canonical ICE. It's a house four-dimension extension; canonical ICE is Impact × Confidence × Ease with no Strategic Fit.
- Don't reserve 0 as "just a low score." 0 is the dealbreaker floor that collapses ICE-F to 0 — if an idea is merely weak on a dimension, score it 1–2.
- Don't pretend you're running De Bono's parallel method. You're applying his six lenses as a critique checklist; don't twist a hat away from its meaning (Red stays feeling/politics, not "the harshest take").
- Don't generate padded Six Hats commentary — each hat should say something that could actually change the verdict.
- Don't deliver a Kill verdict without a reason. "Doesn't fit" is not a reason.
- Don't Park everything to avoid conflict. Park means "fixable with evidence"; it's not a polite Kill.
- Don't run a stress-test on a Kill-level idea — score it, triage it, offer an alternative, move on.
- Don't invent proof points or competitive analogues. Real analogues or `[verify]`.

---

## Quality checklist

- `brand-brain` called and active brand loaded (or fallback executed) before any scoring?
- ICE-F named as a house model (not canonical ICE), with the geometric mean shown and any C ≤ 3 idea flagged with ⚠?
- Dimensions scored 0–10; any 0 reserved for a genuine dealbreaker and auto-routed to Kill?
- Strategic Fit dimension explicitly anchored to brand's ICP and growth lever?
- Six Hats applied as a critique checklist (lenses true to De Bono's meaning, not renamed), with ≥ 1 genuine finding per hat (not filler) and kill shots flagged ⛔?
- Every Keep verdict has an owner/date next action?
- Every Kill verdict states the deciding factor; `constraint-based-ideator` offered if JTBD is sound?
- Park verdicts specify the exact condition to upgrade, with a review date?
- `go-no-go-gate-evaluator` called if explicit gate criteria were provided?
- Artifact saved to `./idea-evals/` for batches ≥ 3 or on request?
- No invented proof — all weak evidence marked `[verify]`?
