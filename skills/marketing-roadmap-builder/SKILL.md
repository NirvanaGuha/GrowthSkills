---
name: marketing-roadmap-builder
description: >
  Turns a raw initiative list, OKR set, or strategy doc into a structured marketing roadmap:
  a prioritized Now/Next/Later table, a quarter-by-quarter milestone schedule, and a clean
  slide outline a junior marketer can drop into a deck or Notion page. Uses a house
  horizon-planning model (Now/Next/Later placement × effort/impact scoring) to rank initiatives before
  placing them, so the roadmap reflects strategic priority — not whoever shouted loudest in
  the last planning meeting. Reads the active brand's positioning, ICP, and offer via
  brand-brain so every initiative is anchored to real business context and not generic marketing
  busywork. Calls okr-suite to validate that roadmap milestones map to measurable goals,
  channel-strategy-selector to sanity-check channel mix, and growth-diagnostic-deep-dive when
  the input is a vague "what should we do next quarter" (it diagnoses first, then roadmaps).
  Saves the roadmap to ./roadmap/[slug]-roadmap.md. Use when the user says "build a roadmap,"
  "Q-planning," "marketing plan for next quarter," "roadmap for the board," "prioritize our
  initiatives," "what should we focus on next," or hands over a backlog/initiative list and
  asks for a plan.
---

# Marketing Roadmap Builder

Raw initiative list in, decision-grade roadmap out. This skill does not just sort a backlog into a table — it uses our horizon-planning model (a house working model) to force explicit trade-offs, anchor every initiative to a measurable goal, and produce a roadmap that survives the first challenge from a skeptical CFO or a busy ops team.

Output: a prioritized Now/Next/Later table, a quarter-by-quarter milestone schedule with owners + success metrics, and a five-section slide outline ready for the deck or Notion.

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads active brand's positioning, ICP, offer, proof, and voice so roadmap themes are grounded in actual strategy, not generic marketing.
- **`okr-suite`** (required) — validates that each roadmap initiative maps to at least one measurable OKR; flags initiatives with no measurable goal as unanchored.
- **`channel-strategy-selector`** (when the input includes channel-level initiatives) — scores the proposed channel mix against ICP + budget + stage; flags under-resourced bets.
- **`growth-diagnostic-deep-dive`** (when input is vague: "what should we focus on?" / no initiative list yet) — diagnoses the growth bottleneck first, then returns a prioritized initiative list to feed this skill.
- **`positioning-messaging-architect`** (optional) — call when a roadmap phase is anchored to a repositioning or new messaging launch; get the messaging architecture before slotting campaign initiatives.
- **`campaign-brief-builder`** (optional, downstream) — hand off an approved roadmap initiative to build the campaign brief; keeps planning and execution distinct.

---

## How a run works

```
Step 0  Load the brand              ──► call brand-brain
Step 1  Diagnose if needed          ──► call growth-diagnostic-deep-dive when no initiative list
Step 2  Collect + structure inputs  ──► intake form if missing
Step 3  Score every initiative      ──► horizon-planning model (house)
Step 4  Build the roadmap           ──► Now/Next/Later table + Q-schedule + slide outline
Step 5  Validate OKR coverage       ──► call okr-suite; flag unanchored initiatives
Step 6  Save + present              ──► ./roadmap/[slug]-roadmap.md
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill before touching any input. Obey the returned positioning, ICP, and banned words as hard context. Use the brand slug as `[slug]` in the output path. Fallback if brand-brain absent: read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if missing, ask for brand context (positioning sentence, ICP, offer, 3 voice adjectives) before proceeding.

### Step 1 — Diagnose if needed

If the user's input is "what should we focus on?" or a strategy brief with no concrete initiative list, invoke `growth-diagnostic-deep-dive` first. Pass the brand context. It returns a prioritized bottleneck + initiative seeds. Feed that output into Step 2 — do not try to synthesize initiatives yourself from a vague brief.

### Step 2 — Intake (ask only what's missing)

Required to build a useful roadmap. Ask in one batch, never piecemeal:

| Field | Why it matters |
|---|---|
| Initiative list (raw bullets OK) | The raw material |
| Planning window (2Q / 4Q / annual) | Sets the horizon lengths |
| Owners / team capacity | What's actually executble |
| Budget envelope (rough OK) | Filters what's realistic |
| North-star goal for the period | Anchors prioritization |
| Hard constraints (launch dates, dependencies) | Gates Now items |

If the input already has some of these, don't re-ask — pull from context.

---

## Horizon-planning model (the engine — our house working model)

Every initiative is scored before it is placed. Score on two axes:

**Impact** (1–5): how directly does this move the north-star goal for the period?
- 5 = primary driver of the goal (e.g. the conversion funnel fix when goal is activation)
- 3 = supporting / multiplier (e.g. SEO content when goal is pipeline)
- 1 = hygiene / brand-health / nice-to-have

**Effort** (1–5): total resource cost — time × people × budget × dependencies.
- 1 = one person, one week, no dependencies
- 5 = cross-functional, multi-week, external dependencies or budget approval

**Priority score** = Impact ÷ Effort. Ties broken by: (a) strategic lock-in risk (high = bump up), (b) dependency on this for a later initiative (blocker = bump up), (c) pure reversibility (easy to undo = can park later).

**Horizon placement rules:**

| Horizon | Criteria |
|---|---|
| **Now** (current quarter) | Score ≥ 2.5, hard constraint, or it gates a Next initiative |
| **Next** (following 1–2 quarters) | Score 1.5–2.4, or requires a Now item to complete first |
| **Later** (3+ quarters / backlog) | Score < 1.5, or effort > available capacity even if high impact; revisit at next planning cycle |

Flag any "zombie initiative" — low score, no clear owner, no measurable goal — as `[PARKING LOT]`. Do not quietly park it; name it and explain why.

---

## Roadmap output structure

### 1. Horizon table (the centrepiece)

```
## [Brand] Marketing Roadmap — [Period]
North-star goal: [goal]
Brand context: [positioning line, ICP, via brand-brain]

| Initiative | Horizon | Owner | Impact | Effort | Score | Success metric | OKR link |
|---|---|---|---|---|---|---|---|
```

Every row gets a success metric and an OKR link. If no OKR maps to the initiative, flag it `[UNANCHORED — add a goal or deprioritize]`.

### 2. Quarter-by-quarter milestone schedule

```
Q[n] — Theme: [e.g. "Activate the Mid-Market Pipeline"]
  Milestone 1: [deliverable] — Owner — due [month]
  Milestone 2: …
  Gate: [what must be true before Q[n+1] begins]
  Leading indicator: [what to watch in-quarter]
```

Assign a single theme per quarter. Themes must be outcome-oriented ("Convert trial users") not activity-oriented ("Run email campaigns").

### 3. Capacity sanity check (one paragraph)

Compare total estimated effort across Now initiatives to stated team capacity. If overloaded (>80% capacity in Now), call it out explicitly and recommend which initiative to move to Next — don't let the user discover the overload mid-quarter. Mark it `[CAPACITY WARNING]`.

### 4. Dependency map (compact)

List any initiative that blocks or is blocked by another. Format: `[A] must precede [B]`. Surface hidden sequencing. If none, say "No critical dependencies identified."

### 5. Slide outline (for deck or Notion)

```
Slide 1 — Goals & north-star metric for the period
Slide 2 — Horizon table (Now / Next / Later)
Slide 3 — Q[n] milestones + owners
Slide 4 — Q[n+1] preview + gate conditions
Slide 5 — Parking lot + revisit trigger
```

Keep slide copy as bullets; do not write full sentences. Headlines should be the decision, not the topic ("Focus on activation this quarter, not acquisition" not "Roadmap priorities").

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No roadmap before the active brand's positioning and ICP are loaded. Generic initiatives not grounded in the brand's actual bottleneck are noise.
- **Score before you slot.** Never place initiatives on a horizon by gut or politics. The Impact ÷ Effort score is visible; if it's overridden, note why explicitly.
- **One success metric per initiative, always.** An initiative with no metric is a wish, not a plan.
- **Flag, don't hide.** Overloaded quarters, unanchored initiatives, and zombie items get called out by name — not quietly buried in Later.
- **Themes, not to-do lists.** Each quarter gets a one-line outcome theme so the team knows what winning looks like, not just what to do.
- **Roadmaps are wrong by definition; build in a gate.** Every quarter ends with a stated gate condition (the thing that must be true before the next horizon starts). If the gate fails, the plan changes — document the trigger.

## What Not to Do

- Don't produce a roadmap before brand-brain returns the active brand's context.
- Don't place initiatives on Now/Next/Later without scoring them first — the user can see the scores.
- Don't accept an initiative with no owner and no metric; ask or mark it `[UNANCHORED]`.
- Don't write quarter themes as activity labels ("Run SEO content") — label outcomes ("Establish organic authority in [category]").
- Don't call okr-suite after presenting — call it before finalizing the roadmap so unanchored items are flagged before the user sees the output.
- Don't confuse "roadmap" with "project plan" — milestones are outcomes, not task lists. Task-level work belongs in campaign-brief-builder downstream.
- Don't skip the capacity check. An overloaded Now quarter is the single most common reason roadmaps fail on contact with reality.

## Quality Checklist (self-review before presenting)

- `brand-brain` called; positioning + ICP loaded; brand slug used in output path?
- Every initiative scored on Impact and Effort; no horizon placement without a score?
- Horizon table has: owner, success metric, OKR link for every row?
- Unanchored initiatives flagged (not silently parked in Later)?
- Capacity sanity check run; overloads named and a move-to-Next recommended?
- Quarter themes are outcome-oriented (not activity labels)?
- Dependency map present (even if empty)?
- Slide outline uses decision-first headlines?
- File saved to `./roadmap/[slug]-roadmap.md`?
- `okr-suite` called and validated before final output?
