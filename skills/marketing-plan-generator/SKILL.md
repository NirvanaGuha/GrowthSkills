---
name: marketing-plan-generator
description: >
  OKRs + channel list + budget → full marketing plan with per-channel initiatives,
  quarterly themes, and milestone schedule. Converts ambiguous goals into a written,
  actionable plan a team can execute and a CFO can interrogate. Uses the SOSTAC
  planning framework (Situation → Objectives → Strategy → Tactics → Actions → Control)
  as its structural backbone so every section earns its place, nothing floats free,
  and the output is auditable end-to-end. Calls brand-brain for voice/ICP/offer/proof,
  channel-strategy-selector when a channel list needs to be derived, okr-suite when
  OKRs need drafting or decomposing, and growth-model-builder when acquisition/retention
  loops are absent. Saves the plan to a project-relative path. Use when the user says
  "write a marketing plan," "build our plan for Q[N]," "give me the annual plan,"
  "turn these OKRs into a plan," "I have a budget and channels, now what," or hands
  over goals and asks how to get there. Output is the plan itself — structured,
  channel-specific, milestone-backed — not advice about how to write one.
---

# Marketing Plan Generator

Turn OKRs, channels, and a budget into a complete, executable marketing plan. Not a deck of nice ideas — a structured document that specifies what is happening each quarter, on which channel, at what cost, measured by which KPI, with explicit accountability.

Every plan is grounded in the active brand's real voice, real ICP, and real offer (via `brand-brain`) and structured through SOSTAC — the six-section model that forces internal consistency. If you can't answer "Control," your objectives weren't measurable. If you can't answer "Strategy," your tactics are tactics in search of a war.

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads active brand: voice, ICP + awareness tendency, offer mechanics + destinations, real proof, positioning, banned words.
- **`channel-strategy-selector`** (when no channel list is provided) — derives a ranked channel shortlist from ICP + budget + stage.
- **`okr-suite`** (when OKRs are missing or vague) — drafts or decomposes quarterly OKRs before building tactics around them.
- **`growth-model-builder`** (when the brand has no documented acquisition/retention loop) — surfaces the lever map the plan should move.
- **`positioning-messaging-architect`** (optional) — called when the plan's strategic theme needs a messaging spine it cannot inherit from `brand-brain` alone.
- **`campaign-brief-builder`** (optional, per initiative) — expands any initiative flagged as a major campaign into a full brief.

---

## How a run works

```
Step 0  Load brand context       ──► call brand-brain (always)
Step 1  Assess inputs             ──► OKRs? channels? budget? time horizon?
Step 2  Fill gaps                 ──► call okr-suite / channel-strategy-selector / growth-model-builder as needed
Step 3  Build the SOSTAC plan     ──► six sections, in order (see below)
Step 4  Self-review               ──► internal consistency check, brand check
Step 5  Save + present            ──► write to ./plans/[slug]-marketing-plan-[YYYY-Q].md; print summary
```

### Step 0 — Load brand context (always first)

**Invoke the `brand-brain` skill** before writing a single line of plan. It returns the active brand slug, voice adjectives, banned words, ICP + awareness tendency, offer mechanics + destination URLs, real proof, and positioning. Everything in the plan — the strategic theme, the channel mix, the copy angles on each initiative, the proof used in campaigns — must be consistent with this digest.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If neither exists, ask the user for brand slug + a 4-question mini-setup (what it is · ICP + awareness · offer + destination · 3 voice adjectives + banned words). Always prefer the call.

### Step 1 — Assess inputs

Collect or confirm:

| Input | Required | Default if absent |
|---|---|---|
| OKRs or goals | Yes | Call `okr-suite` to draft |
| Channel list | Yes | Call `channel-strategy-selector` |
| Total budget | Yes — ask if missing | Cannot proceed without it |
| Planning horizon | Yes | Ask: full year, half, quarter? |
| Quarter the plan starts | Yes | Infer from today's date |
| Current baseline metrics | Strongly preferred | Mark downstream targets `[estimate]` |

Do not begin Section S (Situation) until all required inputs are confirmed or filled by a sibling skill.

### Step 2 — Fill gaps (delegate, don't synthesize)

If OKRs are missing or too vague to pin tactics to → call `okr-suite`, pass the goals + brand digest, fold the returned OKRs in.

If no channel list is provided → call `channel-strategy-selector`, pass ICP + budget + stage. Use the returned ranked list.

If the brand has no documented growth loop → call `growth-model-builder`. The plan's strategy section must move named levers; if those levers aren't mapped, the strategy section is guesswork.

---

## The SOSTAC Plan (the framework)

SOSTAC was developed by PR Smith and is the gold-standard six-section marketing planning model. Each section is a gate — you cannot coherently fill a later section if an earlier one is underspecified. The model enforces internal consistency: if the Control metrics don't match the Objectives, one of them is wrong.

### S — Situation (where we are)

1. **Brand snapshot** (from `brand-brain`): positioning line, ICP, primary offer, primary acquisition channel today, revenue/growth stage.
2. **Current baselines** (user-supplied or flagged `[estimate]`): traffic, leads, MQL, pipeline, CAC, LTV, top organic terms, email list size, conversion rates at key funnel stages.
3. **Growth model summary** (from `growth-model-builder` if called, else inferred): acquisition loop, retention loop, named levers.
4. **SWOT table** — four quadrants, each with 3–5 bullets drawn from the actual inputs, not generic filler. Threats and weaknesses must name a real competitor or a real measured gap.

### O — Objectives (where we need to get to)

Map directly from the supplied or `okr-suite`-generated OKRs. For each objective:

```
Objective:     [state the O]
Key Results:   KR1 · KR2 · KR3  (each SMART: metric + number + deadline)
Baseline:      [current number or [estimate]]
Why it matters: [one sentence connecting to revenue or retention]
```

Do not write objectives without KRs. Do not accept KRs without numbers; push back or flag `[estimate]` with a note.

### St — Strategy (how we get there: the theme + the bets)

The plan's strategic layer. Three components:

1. **Quarterly themes** — one overarching theme per quarter (e.g., "Q3: Retention & Expansion — protect the base while ICP shifts to mid-market"). Themes are directional, not tactical. Every initiative in that quarter must connect back to its theme or it should be cut or moved.

2. **Channel-budget allocation table**

```
| Channel | Role (Acquire / Nurture / Retain / Activate) | Budget % | Budget $ | Primary KPI |
```

Allocate against the OKR mix — if 60% of OKRs are acquisition, roughly 60% of budget should face acquisition channels. Flag any mismatch.

3. **The 3–5 strategic bets** — the highest-leverage initiatives the plan is organized around. Each bet names: the hypothesis ("if we do X, then Y will move because Z"), the channel it lives in, the quarter it starts, and the owner role. These are not tactics — they are the choices that distinguish this plan from a generic one.

### T — Tactics (how each channel executes)

One block per channel in the allocation table. Each block contains:

```
### [Channel Name]
Goal: [channel-level KPI + target]
Q[N] Theme connection: [one line]
Initiatives:
  - [Initiative name] — [brief description, expected output, metric]
  - ...
Key assets needed: [list — landing page, email sequence, ad creative, etc.]
Dependencies: [other channels, skills, or team members this relies on]
Budget: $X
```

For channels that have a relevant sibling skill, name it: e.g., "Use `blog-post-drafting-engine` for SEO content, `push-notification-copy-generator` for push cadence, `cold-outreach-sequence-architect` for SDR sequences."

Keep initiative descriptions sharp: a new hire reading this should know exactly what to build and why, without a follow-up conversation.

### A — Actions (the milestone schedule)

A concrete, sequenced schedule showing the critical path.

```
## Milestone Schedule

| Month | Milestone | Owner | Deliverable | Depends on |
|---|---|---|---|---|
| Month 1 | Brand foundation locked | Brand | brand.md updated, messaging doc | brand-brain refresh |
| Month 1 | Q1 campaigns live | Growth | Ads + landing pages + email live | Creative QA, UTMs |
| Month 2 | First OKR check-in | Lead | KR progress vs target | GA4 / CRM pull |
| ...     | ...                  | ...   | ...                   | ...         |
```

Rules:
- At least one milestone per month for each active quarter.
- Every major strategic bet must have a milestone marking its launch.
- "Owner" is a role, not a name (the plan outlasts any one person).
- The critical path — the sequence of milestones that, if delayed, delay everything — must be identifiable by reading the table.

### C — Control (how we know it's working)

The measurement + review cadence that closes the loop.

1. **Measurement stack**: list the 3–5 primary KPIs across all OKRs, their data sources, and who pulls them. Mark any KPI that requires a new instrumentation step (GA4 event, CRM field, UTM taxonomy) as `[setup required]`.

2. **Review cadence table**:

```
| Cadence | Forum | KPIs reviewed | Decision trigger |
|---|---|---|---|
| Weekly | Growth standup | CAC, pipeline velocity | Pause tactic if 2-week trend is negative |
| Monthly | Full OKR review | All KRs vs target | Re-allocate budget if any KR is >20% behind |
| Quarterly | Exec review | OKR scores, theme verdict | Carry, pivot, or kill each strategic bet |
```

3. **Leading vs lagging indicator map** — for each OKR, identify the leading indicator (the metric that tells you in week 2 whether you're on track for the quarter-end KR). If you can only measure success at the end, you've already lost.

4. **Budget reallocation rule** — one explicit sentence: what threshold triggers a reallocation, who approves it, and what the fallback channel is.

---

## Save behavior

Save the completed plan to `./plans/[brand-slug]-marketing-plan-[YYYY-Q].md` (e.g., `./plans/pushengage-marketing-plan-2026-Q3.md`). Create the `./plans/` directory if it does not exist. Print the path after saving.

If the user is mid-run on an existing plan (same slug + quarter), diff the changes and ask before overwriting.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No plan section written before `brand-brain` returns the active digest.
- **SOSTAC in order.** Do not write Tactics before Objectives are confirmed. Do not write Control before you know what the Objectives are measuring. Each section is a gate.
- **Real numbers or `[estimate]`/`[verify]`.** Never invent a baseline, a benchmark, or a budget split. If the user hasn't provided it, flag it.
- **Strategies are bets, not categories.** "Content marketing" is not a strategy. "Publish 8 high-intent comparison pages targeting bottom-of-funnel eCommerce queries to grow organic MQL 30% by Q4" is.
- **Quarterly themes must filter.** Every initiative that cannot be connected to its quarter's theme is a candidate for the cut list, not an afterthought.
- **Compose, don't rebuild.** If a channel needs a campaign brief, call `campaign-brief-builder`. If OKRs are missing, call `okr-suite`. This skill assembles the plan; siblings do their specialized work.
- **The plan must be executable by someone other than its author.** If a milestone requires context that isn't in the document, it is underspecified.

---

## What Not to Do

- Do not begin writing before `brand-brain` returns the active brand.
- Do not write Tactics without Objectives. Do not write Control metrics that don't map to a stated Objective.
- Do not use "increase brand awareness" as an objective. Push back; demand a measurable KR.
- Do not allocate budget in percentages only — always convert to dollars once the total is confirmed.
- Do not list more than 5 strategic bets. A plan that bets on everything is a plan that bets on nothing.
- Do not use voice, claims, or proof not sanctioned by `brand-brain`. Banned words are banned in the plan, too.
- Do not generate the milestone schedule before the Tactics section is stable — the schedule is a consequence of decisions already made, not a starting point.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded before any plan section written?
- Situation: baselines real or flagged `[estimate]`; SWOT is specific (real competitor names, real measured gaps)?
- Objectives: every O has ≥2 SMART KRs with numbers; no KR is purely qualitative?
- Strategy: quarterly themes cover the full horizon; channel-budget table adds to 100%; ≥3 strategic bets with named hypotheses?
- Tactics: every channel has a goal, theme connection, named initiatives, asset list, dependencies, and budget?
- Actions: milestone table covers every month in the plan; every strategic bet has a launch milestone; critical path is identifiable?
- Control: leading indicator named for every OKR; review cadence defined; reallocation rule explicit; any new instrumentation flagged `[setup required]`?
- Plan saved to `./plans/[slug]-marketing-plan-[YYYY-Q].md` and path confirmed?
