---
name: constraint-based-ideator
description: >
  Takes a campaign brief plus one or more hard constraints (budget ceiling, channel lock-in, timeline
  compression, headcount, asset reuse requirement, regulatory fence) and produces a shortlist of
  viable ideas that are *optimized for* the constraint — not despite it. The constraint is the brief,
  not a filter applied after the fact. Uses Oblique Strategies logic + SCAMPER as the ideation
  backbone, then scores each idea for constraint-leverage (how much the constraint actively helps the
  idea), feasibility inside the constraint, and brand-fit. Returns a ranked slate, a "why this
  constraint is an advantage" framing note per idea, and downstream composition calls for development.
  Does not implement brand context, evaluation scoring, or campaign development — calls the relevant
  library skills for those jobs. Use whenever the user says "we only have $X," "we can't use paid,"
  "we need something in 48 hours," "our team is just two people," "we're locked to email only,"
  "brainstorm within our limits," "ideate with constraints," or hands over a brief with budget/channel/
  time restrictions and asks what to do.
---

# Constraint-Based Ideator

The constraint is not the obstacle. It is the brief.

Most ideation treats limits as a filter: generate freely, then throw out what doesn't fit. This skill inverts that. The budget ceiling, the channel lock, the 48-hour window — each one eliminates the ideas that need resources to cover for a weak core. What remains has to be genuinely good.

This skill uses **Oblique Strategies logic** (Brian Eno and Peter Schmidt's constraint-as-generative-force principle) combined with **SCAMPER** (Substitute, Combine, Adapt, Modify/Magnify, Put to other uses, Eliminate, Reverse) as the structural ideation backbone. Every output names the constraint-leverage angle: the specific way the limitation makes the idea *better*, not just compatible.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — resolves and loads the active brand's voice, ICP, offer, banned words, and proof. This skill does not derive brand context.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no brand.md exists, ask the user for [brand voice adjectives, ICP description, core offer, and any banned words or tone guardrails].
- **`idea-evaluation-stress-test-suite`** — scores and adversarially critiques the shortlist after generation; call when the user wants a full go/park/kill verdict rather than a ranked slate.
- **`campaign-concept-developer`** — takes a selected idea to full campaign concept; call when the user picks a winner and wants development.
- **`campaign-brief-builder`** — produces the formal brief for the winning concept before execution handoff.
- **`go-no-go-gate-evaluator`** — applies formal gate criteria to the final selected idea before committing resources.
- *(optional)* `how-might-we-10x-reframer` — useful before this skill runs if the problem framing needs a reframe; call when the brief feels stuck or too narrow.

---

## How a run works

```
Step 0  Load the brand         ──► call brand-brain
Step 1  Parse the constraint   ──► extract the type, severity, and leverage surface
Step 2  Reframe the constraint ──► write the "this is an advantage because..." frame
Step 3  SCAMPER + Oblique pass ──► generate ideas with constraint as the engine
Step 4  Score and rank         ──► constraint-leverage / feasibility / brand-fit
Step 5  Present the slate      ──► ranked ideas + the advantage frame + next-step calls
```

---

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`) passing the user's request and any named brand. It returns the active brand's digest — voice adjectives, banned words, offer mechanics and destinations, real proof, positioning, ICP and awareness tendency — and the path to `brand.md`. Do not generate a single idea until brand-brain returns.

Obey voice and banned-words as hard overrides. Use only the returned real proof; mark anything else `[verify]`.

---

### Step 1 — Parse the constraint

Before ideating, nail the constraint's exact shape. Classify it:

| Constraint type | Leverage surface |
|---|---|
| **Budget ceiling** | Forces earned/owned channels, community, asymmetric bets |
| **Channel lock** (email only, push only, organic only) | Forces depth over breadth; channel-native creativity |
| **Timeline compression** | Forces scope reduction to a single sharp idea; rules out production-heavy formats |
| **Team/headcount** | Forces templates, repurposing, automation, creator partnerships |
| **Asset reuse** | Forces remix thinking; surfaces underused existing equity |
| **Regulatory / legal fence** | Forces proof-backed claims, third-party validation, transparent framing |
| **Platform restriction** | Forces format innovation within one surface |

If multiple constraints are present, identify the **binding constraint** — the one that eliminates the most options. Design to the binding constraint first; the others typically resolve.

State the binding constraint explicitly before generating ideas. If the brief is ambiguous about severity (e.g., "low budget" with no number), ask one clarifying question before proceeding.

---

### Step 2 — Reframe the constraint as an advantage

For each constraint, write one sentence in the shape:

> "This constraint *eliminates* [the noisy/expensive/generic option], which means every idea here has to be [genuinely interesting / channel-native / proof-backed / fast-twitch / etc.]."

This is the generative reframe. It anchors every SCAMPER pass that follows — ideas that wouldn't be better *because* of the constraint get cut here, not later.

---

### Step 3 — SCAMPER + Oblique pass

Run the constraint through SCAMPER, applying the active brand's ICP and voice at each move. Not every SCAMPER direction produces a usable idea; apply judgment and skip unproductive directions rather than forcing output.

**SCAMPER applied to constraint-based ideation:**

- **Substitute** — what standard-budget or standard-channel element could be replaced with something the constraint *enables*? (e.g., paid distribution → community seeding)
- **Combine** — what two low-resource tactics, combined, achieve the effect of one expensive one?
- **Adapt** — what has worked in a different category or context under a similar constraint? Transpose it.
- **Modify / Magnify** — what happens if you push the constraint further on purpose, making it a feature of the campaign rather than hiding it?
- **Put to other uses** — what existing asset, audience, or touchpoint is going unused that this constraint would force you to exploit?
- **Eliminate** — what does the constraint force you to cut that you probably should have cut anyway? What's the simplified version of the campaign that remains?
- **Reverse** — what if the constraint became the campaign message itself? (e.g., "We can't afford to run ads, so we're asking you to share this instead.")

**Oblique pass (1–2 moves per run):** Apply one or two random-direction prompts drawn from Oblique Strategies logic — a technique proven to break default ideation patterns by introducing orthogonal constraints. Examples: "What would you do if you had to make this invisible?", "Use an old idea as if it were new", "What's the most embarrassing version of this? Make it intentional." Apply to the SCAMPER outputs to push at least one idea into unexpected territory.

Target **5–8 ideas** total. Quality over count. Drop any idea that isn't meaningfully shaped by the constraint.

---

### Step 4 — Score and rank

Score each idea on three dimensions (1–5):

| Dimension | What it measures |
|---|---|
| **Constraint leverage** | How much does the constraint make this idea *better*? A 5 means the idea would be worse with more resources; a 1 means it just happens to fit. |
| **Feasibility inside the constraint** | Can this be executed fully within the stated limits, with no hidden resource requirements? |
| **Brand fit** | Voice, ICP, offer alignment per the brand-brain digest. |

Rank by composite score (equal weight). Flag any idea scoring ≤2 on feasibility — it stays in the slate as a "stretch" but is labeled as such.

---

### Step 5 — Present the slate

Output format:

```
## Constraint-Based Ideas — [brand slug] / [constraint summary]

Binding constraint: [one line]
Constraint reframe: [the advantage sentence]

### [Rank]. [Idea name — short, memorable]
**SCAMPER move:** [which direction generated this]
**The constraint leverage:** [why this is *better* because of the limit]
**Execution sketch:** [2–3 sentences — what it is and how it runs]
**Scores:** Leverage [x/5] · Feasibility [x/5] · Brand fit [x/5]

[repeat for each idea]

---
### Recommended starting point
[Pick the top idea and name one concrete next action: call campaign-concept-developer, or name the
first asset to produce.]
```

Save the full slate to `./ideation/[brand-slug]-[YYYY-MM-DD]-constraints.md` if the user asks for a record; otherwise deliver inline.

---

## When to call downstream skills

After presenting the slate:
- User picks a winner → call **`campaign-concept-developer`** to build it out.
- User wants a formal stress-test → call **`idea-evaluation-stress-test-suite`** on the top 2–3.
- User wants a gate review before committing → call **`go-no-go-gate-evaluator`**.
- User wants the winning idea turned into a brief → call **`campaign-brief-builder`**.

Do not build the campaign here. This skill generates and ranks; development lives downstream.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No idea generation before the active brand loads. Voice and banned-words are hard overrides.
- **The constraint is the engine, not the filter.** Every idea must be *better because* of the limit, not merely compatible with it. Cut any idea that is just "fits the budget."
- **Name the SCAMPER move.** Not for formality — so the user can request more ideas from a specific direction, or so a reviewer can audit the thinking.
- **Constraint-leverage score is the primary ranking criterion.** A highly feasible but leverage-poor idea ranks below a harder but leverage-rich one.
- **Honest feasibility.** If an idea requires a hidden resource (design time, a partnership, an asset that doesn't exist), say so — don't bury it in "execution notes."
- **Real proof or `[verify]`.** If an example from another brand is cited, name the brand and approximate date; if unverified, mark it.
- **5–8 ideas, not a waterfall.** Dense generation followed by ruthless cutting. Never pad to hit a number.

---

## What Not to Do

- Don't generate ideas before `brand-brain` returns the active brand.
- Don't treat the constraint as a filter applied after free ideation — restructure around it from the start.
- Don't present ideas that could be executed without the constraint with no meaningful change.
- Don't skip the constraint reframe step — it's what separates this from generic brainstorming.
- Don't develop the winning idea here — hand off to `campaign-concept-developer`.
- Don't use `[verify]` as a cop-out for lazy examples; if you're not confident in a benchmark or case, either find a solid one or don't cite it.
- Don't force all seven SCAMPER directions — apply judgment, skip unproductive ones, note which you dropped and why.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or fallback path followed) before any idea was generated?
- Binding constraint identified and stated explicitly?
- Constraint reframe written — the "this is an advantage because..." sentence?
- Every idea named its SCAMPER move and its constraint-leverage angle?
- No idea in the slate could be executed equally well with twice the budget / no channel restriction?
- Feasibility score is honest — hidden resource requirements surfaced, not buried?
- Ideas scored on all three dimensions; ranked by composite; stretch ideas labeled?
- Recommended next action named, with the correct downstream skill called or queued?
