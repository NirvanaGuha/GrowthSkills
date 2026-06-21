---
name: how-might-we-10x-reframer
description: >
  Transforms a problem statement, brief, or existing campaign idea into two interlocking outputs:
  (1) a curated set of How Might We (HMW) question variants that reframe the problem at the right
  level of abstraction — wide enough to open options, narrow enough to stay actionable — and
  (2) a set of 10x concept stretches that deliberately remove the small-thinking defaults embedded
  in the original framing. The skill diagnoses constraint assumptions, runs the IDEO HMW ladder
  (too narrow → too broad → sweet spot), then applies moonshot reframing (remove the bottleneck,
  flip the model, change the unit of value) to surface ideas a team in "optimize mode" would never
  reach. Brand-contextualized: voice, ICP, offer, and competitive moats come from brand-brain so
  every reframe is anchored to a real market position rather than generic provocation. Output is a
  ready-to-facilitate ideation session or a standalone async brief. Use whenever the user says
  "how might we," "reframe this problem," "10x this idea," "think bigger," "ideation session,"
  "we're stuck on X," "what's the moonshot version," or hands over a problem and asks for
  creative unlocks. Does not write final copy or ads — hands off to campaign-concept-developer,
  headline-hook-generator, or cta-variant-generator for execution.
---

# How Might We & 10x Reframer

You can't 10x a solution designed around a 1x assumption. This skill does one thing before ideation begins: it breaks the frame. It surfaces the hidden constraints baked into the original problem statement, reframes them as design questions, then stretches each question to a scale that makes the incremental answer look small.

The output is a structured ideation brief — not brainstormed copy, not a campaign. The brief hands off cleanly to execution skills once the team agrees on which reframe to chase.

---

## Skills this calls

- **`brand-brain`** (required first) — loads voice, ICP, offer mechanics, positioning, and competitive moats. Reframes land differently when you know the brand's actual constraints vs. assumed ones.
- *(optional, compose when relevant)* `positioning-messaging-architect` — if reframes expose a positioning gap, hand off rather than synthesize positioning here.
- *(optional, compose when relevant)* `campaign-concept-developer` — takes a chosen 10x reframe to a full concept with hero message and execution angles.
- *(optional, compose when relevant)* `creative-ideation-framework-suite` — if the user wants a broader SCAMPER/Six Hats/Crazy-8s run after the reframe, chain it.
- *(optional, compose when relevant)* `headline-hook-generator` — for turning a 10x reframe into candidate headline concepts.
- *(optional, compose when relevant)* `cta-variant-generator` — for any reframe that resolves to a new conversion mechanic or offer.
- *(optional, compose when relevant)* `a-b-multivariate-test-designer` — if a reframe is testable today, scaffold the experiment.

---

## How a run works

```
Step 0  Load the brand        ──► call brand-brain; absorb positioning + ICP + moats
Step 1  Unpack the original   ──► extract the embedded assumption(s) driving the current frame
Step 2  HMW ladder            ──► narrow / sweet-spot / broad variants (IDEO three-level method)
Step 3  10x stretches         ──► moonshot reframes per constraint identified in Step 1
Step 4  Prioritize + hand off ──► recommend 1–2 frames to chase; flag execution-skill handoffs
```

---

## Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's digest: voice adjectives, banned words, offer mechanics + destination URLs, real proof, ICP + awareness tendency, positioning line. Do not produce any reframes before it returns.

Use the brand context to calibrate the reframes: a 10x stretch for a self-serve PLG product is different from one for an enterprise ABM play. ICP constraints and moats are inputs, not walls.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If neither exists, ask the user for: brand name, ICP (who + their job-to-be-done), current offer, and one key competitive constraint. Prefer the call.

---

## Step 1 — Unpack the original frame

Before writing any HMW questions, audit the input statement for embedded assumptions. These are the small-thinking defaults the reframe must remove.

**Common assumption types to surface:**

| Assumption type | Example hidden in "increase trial-to-paid conversion" |
|---|---|
| Channel lock-in | "via email drip" |
| Unit of value | "per seat / per user" |
| Direction of motion | "push them to upgrade" |
| Who acts | "the champion decides alone" |
| Time horizon | "within the 14-day trial" |
| What the bottleneck is | "they haven't seen the value" |

Name 2–4 assumptions explicitly. These become the axes for 10x reframing in Step 3.

---

## Step 2 — HMW ladder (IDEO three-level method)

The IDEO How Might We technique reframes a problem at three levels of abstraction. A well-calibrated HMW sits in the sweet spot: wide enough that multiple solutions exist, narrow enough that a team can act.

For each identified assumption, write one HMW at each level:

**Narrow (too specific — names a solution):** useful as a sanity-check, shows where current thinking is stuck.
**Sweet spot (the useful zone):** opens 5–10 different solution directions without prescribing any.
**Broad (too abstract):** reveals the larger goal; useful as a north-star but rarely immediately actionable.

Format:
```
Assumption: [name it]
  ↳ Narrow   HMW: …
  ↳ Sweet    HMW: …  ← recommended for ideation
  ↳ Broad    HMW: …
```

Aim for 2–3 assumptions → 6–9 HMW questions, of which 2–3 sweet-spot ones are flagged for use.

---

## Step 3 — 10x stretches (moonshot reframing)

Where HMW opens the design space, 10x reframing collapses the assumption that the current constraint is permanent. Use these four lenses on the assumptions from Step 1:

| Lens | Prompt | What it breaks |
|---|---|---|
| **Remove the bottleneck** | "What if the bottleneck didn't exist at all?" | Exposes whether the bottleneck is real or assumed |
| **Flip the model** | "What if the value flowed in the opposite direction?" | Reverses who pays, who acts, who benefits |
| **Change the unit** | "What if we delivered 10x the value to 1/10th the audience?" (or 1/10th the cost to 10x the audience) | Breaks pricing and scale assumptions simultaneously |
| **Borrow from a category** | "How does [analogous category with opposite problem] solve this?" | Lateral theft of a working mechanic |

Write 1 stretch per lens per key assumption. Not every stretch is viable — flag obviously blocked ones rather than suppressing them; the point is to make the default solution look like a local maximum.

**Output format per stretch:**
```
[Lens] on [Assumption]:  "What if … [stretch statement]"
Implication: [what this unlocks / what it would require]
Viability: [likely viable / requires structural change / deliberately impossible — for framing only]
```

---

## Step 4 — Prioritize and hand off

Recommend **1–2 reframes** most worth pursuing, using:
- **Novelty × feasibility signal:** strong reframes are surprising _and_ structurally possible with the brand's actual moats.
- **ICP unlock:** does it address an unmet job the ICP has that competitors haven't touched?
- **Moat alignment:** does it play to a differentiated capability, not a me-too feature?

End with explicit handoff suggestions:
- If a reframe resolves to a campaign concept → `campaign-concept-developer`
- If it resolves to a testable hypothesis → `a-b-multivariate-test-designer`
- If it exposes a positioning gap → `positioning-messaging-architect`
- If it unlocks a new headline/message angle → `headline-hook-generator`

Save the full reframe brief to `./plans/hmw-reframe-[slug]-[YYYYMMDD].md` if the session is named or the user wants to persist it. Never save to brand.md or the skill folder.

---

## Principles

- **Break the frame before filling it.** No reframe is useful if it preserves the assumption that created the problem.
- **Brand-brain first, always.** ICP constraints, competitive moats, and the offer mechanics are non-negotiable inputs — a 10x stretch that ignores them is creative fiction, not a growth lever.
- **HMW questions are not slogans.** They are design constraints written as invitations. They should feel slightly uncomfortable, not exciting.
- **Name what you're breaking.** Every 10x stretch should explicitly state which assumption it removes. If it doesn't, it's a synonym.
- **Viable and impossible both belong.** A deliberately impossible stretch makes the viable one look bolder. Don't filter before the team sees both.
- **Don't execute here.** This skill produces briefs, not copy. Hand off to execution skills cleanly.

---

## What not to do

- Don't produce HMW questions before `brand-brain` returns — generic reframes detached from real positioning are a waste of a session.
- Don't write 10 near-identical HMW variants; vary the assumption axis, not the vocabulary.
- Don't suppress "impossible" stretches — mark them and keep them; the best ideas often start there.
- Don't write campaign copy, ads, or headlines here — chain to the right execution skill.
- Don't reimplement brand scanning, ICP research, or competitive analysis — those live in their respective skills.
- Don't present a reframe without naming the assumption it removes; the team needs to know what they're committing to abandon.

---

## Quality checklist

- `brand-brain` called and active brand loaded (digest: voice, ICP, offer, moats) before any output?
- 2–4 embedded assumptions named explicitly from the original frame?
- HMW ladder complete: narrow / sweet-spot / broad for each key assumption; sweet-spot flagged?
- 10x stretches: one per lens (Remove / Flip / Change unit / Borrow) with implication + viability?
- Recommendation names 1–2 frames with rationale tied to ICP unlock and moat alignment?
- Execution handoffs specified (which downstream skill, which input)?
- Voice + banned-words from brand-brain honored throughout; no invented proof?
