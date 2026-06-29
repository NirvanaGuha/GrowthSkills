---
name: quarterly-goal-decomposer
description: >
  Takes 1–3 quarterly goals and decomposes them into an execution-ready project plan: month-by-month
  milestones, weekly leading indicators, dependency sequencing, and a Notion-compatible task board.
  Uses an OKR-cascade approach (our working model, we call it the "OKR waterfall") to keep outcomes
  (the "O") and key results (the "KR") structurally separate from tasks, so the plan stays connected
  to the goal even as work shifts. Brand context loads
  via brand-brain so company naming, voice, and ICP-anchored priority logic are accurate from the
  start. Produces a single markdown plan plus an optional Notion-importable CSV. Use when the user says
  "break down my Q goals," "make a quarterly plan," "decompose my OKRs," "turn my Q into a project
  plan," "what should I be doing this quarter," "quarterly roadmap," or hands you goals and says
  "build me a plan."
---

# Quarterly Goal Decomposer

Give it 1–3 quarterly goals; get back a fully-wired execution plan — monthly milestones, weekly
leading indicators, a dependency-sequenced task board, and the honest risks that could derail it.
Connected to the goal at every layer so you're not just executing tasks, you're moving the needle.

This skill plans. It does not write content, build campaigns, or run analysis — it identifies which
skills to call downstream and wires the work into the right sequence. If the goals are vague, it
clarifies before it plans; a crisp quarter starts with a crisp goal.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's ICP, positioning, offer mechanics,
  and proof so milestone framing and priority logic are brand-accurate. Does not reimplement brand
  resolution. Fallback if brand-brain is absent or returns no brand: do a thin direct read of the
  already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no
  `brand.md` exists, ask the user for [brand name, ICP one-liner, primary growth metric this quarter].
- **`kpi-tree-builder`** *(optional)* — if the user hasn't defined KRs yet, call this to derive a
  measurable KR set from the goal before decomposing. Pass it the goal statement and brand context.
- **`marketing-roadmap-builder`** *(optional)* — when the output needs to land in a slide or
  now/next/later format rather than a task board, pipe the milestone list here.
- **`okr-suite`** *(optional)* — when the goals need a formal OKR structure before decomposing, call
  this first to draft and score the Objective + Key Results, then feed the output into Step 2 here.
- **`risk-log-builder`** *(optional)* — after the plan is drafted, call this to generate a full risk
  register. Synthesize inline when absent.
- **`go-no-go-gate-evaluator`** *(optional)* — useful at milestone boundaries; the user can run it
  against each monthly gate to get a proceed/pause/kill verdict.
- **`deadline-drift-detector`** *(optional)* — run mid-quarter to flag milestones at risk.
- **`weekly-ops-digest`** *(optional)* — the plan's weekly leading indicators become this skill's
  weekly input; connect them explicitly in the output.

---

## How a run works

```
Step 0  Load the brand         ──► call brand-brain
Step 1  Clarify the goals      ──► extract or sharpen Objective + Key Results
Step 2  Structure the quarter  ──► OKR cascade: months → milestones → leading indicators
Step 3  Build the task board   ──► projects, owners, dependencies, sequencing
Step 4  Surface the risks      ──► 3–5 honest threats, each with a mitigation
Step 5  Deliver the artifacts  ──► markdown plan + optional Notion CSV
```

---

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request and
any named brand. It returns the active brand digest: voice adjectives, banned words, ICP + awareness
tendency, offer mechanics, proof, positioning. Use the ICP and positioning to inform *which*
milestones matter (e.g., for a PLG brand, activation milestones rank above awareness milestones).

Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written
brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no `brand.md` exists,
ask the user for [brand name, ICP one-liner, primary growth metric this quarter].

### Step 1 — Clarify the goals

Before decomposing, every goal must have:
- **An Objective** — the qualitative direction ("Grow enterprise pipeline in North America")
- **Key Results** — 2–4 measurable outcomes that define "done" for the quarter (SMART: specific,
  measurable, time-bound). Numbers where available; `[verify]` where not.
- **The primary growth metric** — the one number this quarter lives or dies by.

If the user provides goals that lack KRs or have no numbers, ask for them in a single batch. Show what
you inferred from context and ask only for the gaps — don't re-ask what was already stated.

**If goals are ambiguous or overlapping**, surface the conflict before planning. A plan built on a
contradictory goal is worse than no plan.

### Step 2 — Structure the quarter (OKR cascade)

Apply our OKR-cascade structure (we call it the "OKR waterfall") — the standard OKR outcome-to-output
hierarchy from Doerr's *Measure What Matters*, wired into a month-by-month plan:

```
Objective
  └── Key Result 1
        └── Initiative A  →  Month 1 milestone  →  Month 2 milestone  →  Month 3 milestone
        └── Initiative B  →  ...
  └── Key Result 2
        └── ...
```

Rules:
- Every initiative traces back to exactly one KR. No orphan tasks.
- Every month gets a milestone that is *observable* (something you can confirm happened) not just
  a progress hedge ("make progress on X").
- Month 1 milestones are foundations: data/tooling/research that unblocks M2 and M3.
- Month 3 milestones are outcome-close: the KR number should be in reach.
- Sequence dependencies explicitly: flag when Initiative B cannot start until Initiative A ships.

**Weekly leading indicators** (one per KR, measured every week):
A leading indicator predicts whether the lagging KR will land. Pick the earliest-in-the-funnel metric
that causally precedes the KR. Example: if the KR is "100 SQLs," the leading indicator is not SQLs —
it's qualified demos booked per week (which becomes SQLs ~3 weeks later).

### Step 3 — Build the task board

For each initiative, produce a task list with:
- **Task name** — verb-first, specific, completable in ≤1 week
- **Owner** — role (not person, unless the user specifies)
- **Due** — week number within the quarter (W1–W13)
- **Dependency** — which task(s) must complete first (task ID or name)
- **Skill to call** — if a task maps to a library skill (e.g., "Write Q3 email drip" → call
  `lead-nurture-drip-builder`), name it. Reduce reinventing the wheel.

Organize the board by initiative, not by week. Week-ordering is for a Gantt; the board is for
ownership and sequencing.

```
### Task Board — [Objective]

| ID  | Task                          | Owner   | Due  | Depends on | Skill              |
|-----|-------------------------------|---------|------|------------|--------------------|
| T1  | Define ICP qualification rules | RevOps  | W1   | —          | icp-persona-builder|
| T2  | Build lead scoring model       | RevOps  | W2   | T1         | lead-scoring-routing-model-designer |
| ... |
```

### Step 4 — Surface the risks

3–5 named risks, each with:
- What could go wrong (specific, not generic "resourcing issues")
- Probability: High / Medium / Low
- Impact on quarter if it lands
- Mitigation: the one action that most reduces the risk, and by when

Call `risk-log-builder` if installed; synthesize inline otherwise.

### Step 5 — Deliver the artifacts

**Primary output:** a single markdown plan saved to `./plans/q[N]-[year]-[brand-slug].md`.

Structure:
```
# Q[N] [Year] Plan — [Brand] — [Objective headline]

## Goals & Key Results
## Monthly Milestones
## Weekly Leading Indicators
## Task Board
## Risk Register
## Downstream skills to call (with suggested timing)
```

**Optional Notion CSV** (on request): one row per task with columns: Name, Status, Owner, Due Week,
KR link, Dependency, Notes. Saves to `./plans/q[N]-[year]-[brand-slug]-notion.csv`. Importable via
Notion's CSV import into a Board or Table view.

**If calling downstream skills:** end the plan with a sequenced list of library skills to invoke
(with timing), so the plan generates its own execution chain.

---

## The OKR cascade — our working model

Our OKR cascade (a house model we nickname the "OKR waterfall," built on Doerr's outcome-to-output
hierarchy) maintains two structural rules that most plans break:

1. **Outcome-to-output hierarchy is strict.** KRs define *what* lands; Initiatives define *how*.
   If a task doesn't move a KR, cut it or park it. If a KR has no initiative under it, the quarter
   is aspirational, not planned.

2. **Leading indicators are not checkboxes.** A weekly leading indicator is a *rate* (demos/week,
   signups/day, posts/week) that you track every single week. If it trends below target by W4, the
   KR is already at risk. The plan must name these explicitly — they are the quarter's early-warning
   system, not a nice-to-have.

**Month sequencing logic (the 3-act structure):**
- M1 — Foundation: instruments, research, ICP confirmation, tool setup
- M2 — Execution: campaigns live, sequences running, content publishing
- M3 — Optimization + close: test winners rolled out, KR gap closed, learnings captured

Deviating from this order is legitimate — just flag *why* (e.g., a live event in M1 forces an M2
cadence forward).

---

## Principles

- **Brand-brain first.** No plan before the brand digest returns. ICP and positioning determine
  which milestones are worth putting on the board.
- **Every task traces to a KR.** Orphan tasks are either scope creep or a missing KR. Name which.
- **Leading indicators are rates, not events.** A rate can be tracked weekly; an event is a
  milestone. Don't conflate them.
- **Sequence explicitly.** A plan that doesn't name dependencies is a list, not a plan.
- **Downstream skills, not instructions.** Where a task maps to a library skill, name it — don't
  duplicate the skill's logic inline.
- **Truth discipline.** Numbers in KRs are the user's own targets. Don't invent benchmarks;
  use `[verify]` for anything unconfirmed.

## What Not to Do

- Don't produce a plan before the goals have measurable KRs — a goal without a KR is a wish.
- Don't decompose ambiguous or conflicting goals; surface the conflict first.
- Don't use week-ordered task lists as a substitute for dependency sequencing.
- Don't reimplement brand scanning/interviewing here — call `brand-brain`.
- Don't write "make progress on X" as a milestone — every milestone is observable or it isn't a
  milestone.
- Don't invent staffing or budget figures; use `[verify]` or ask the user.
- Don't duplicate sibling skill logic inline — name the skill to call.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or fallback executed) before any plan output?
- Every goal has an Objective + 2–4 measurable KRs; primary growth metric named?
- Ambiguous or conflicting goals surfaced *before* planning, not papered over?
- OKR cascade structure intact: every initiative traces to a KR, every KR has an initiative?
- Monthly milestones observable (not hedge phrases); M1 = foundations, M3 = outcome-close?
- One weekly leading indicator per KR — a rate, not an event?
- Task board has owner, due week, dependency, and a skill-to-call where relevant?
- 3–5 specific risks with probability, impact, and mitigation?
- Plan saved to `./plans/q[N]-[year]-[brand-slug].md`; Notion CSV offered?
- Downstream skill sequence listed at the end of the plan?
