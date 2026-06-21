---
name: creative-ideation-framework-suite
description: >
  Takes a problem statement, campaign goal, or creative brief and returns structured, divergent ideas
  using a curated set of named ideation frameworks: SCAMPER, Six Thinking Hats, First-Principles
  Deconstruction, Crazy-8s, Brainwriting 6-3-5, and lateral provocations (PO technique). Does NOT
  generate brand context itself — it calls brand-brain first so every idea is grounded in real voice,
  real ICP, and real offer before generating. Produces 3–5 immediately usable concept directions per
  session plus a champion pick with rationale, or a full ideation board across all six frameworks on
  request. Naturally feeds campaign-concept-developer, headline-hook-generator, cta-variant-generator,
  and a-b-multivariate-test-designer downstream. Use when the user says "I'm stuck on ideas,"
  "brainstorm this campaign," "what angles could we take," "I need creative concepts," "help me think
  differently about this," "creative sprint," "generate ideas for [X]," or hands over a brief and asks
  for options.
---

# Creative Ideation Framework Suite

Structured divergence engine for growth and content marketers. Give it a problem statement or campaign
goal and it applies the right frameworks to escape default thinking — then filters back to the 3–5
strongest ideas grounded in the brand's voice, ICP, and real offer. No open-ended free-association; every
framework has a job and an output shape.

This skill generates divergent ideas and frames them for execution. It does not write the final copy,
build the brief, or measure performance — it hands off to the right downstream skill for that.

---

## Skills this calls

- **`brand-brain`** (required first) — resolves voice, banned words, ICP, offer mechanics, real proof,
  and positioning. No idea generation until this returns. Do not reimplement brand resolution here.
- **`campaign-concept-developer`** — hand off the champion idea(s) when the user wants a fully fleshed
  concept (big idea + hero message + execution angles).
- **`headline-hook-generator`** — when a direction needs headlines and hooks developed.
- **`cta-variant-generator`** — when a direction reaches a conversion moment.
- **`a-b-multivariate-test-designer`** — when two competing directions should be tested; structure the
  hypothesis here, hand the test brief there.
- **`content-brief-builder`** — when a direction becomes a piece of content.
- **`ad-copy-variant-generator`** — when a direction turns into paid creative.

Fallback if `brand-brain` is not installed: read `~/.brandbrain/brands/.active` and that brand's
`brand.md` directly; if absent, ask 4 bootstrap questions (what it is · ICP + awareness tendency ·
offer + destination · 3 voice adjectives + banned words), then proceed. Prefer the call.

---

## How a run works

```
Step 0  Load brand context  ──► call brand-brain skill; block until it returns
Step 1  Scope the problem   ──► clarify input type: campaign goal | creative brief | stuck problem
Step 2  Pick mode           ──► Quick (1–2 frameworks + 3–5 ideas) | Full board (all 6)
Step 3  Generate            ──► run frameworks; ICP-filter each output; mark unconfirmed [verify]
Step 4  Converge            ──► champion pick + rationale; offer handoff to downstream skills
```

Default to Quick. Trigger Full board on "full ideation," "brainstorm session," "six hats," or an
explicit count request (≥8 ideas, "all frameworks," "give me everything").

---

## The six frameworks — what each does and when to use it

### SCAMPER  (Substitute · Combine · Adapt · Modify · Put to other uses · Eliminate · Reverse)

Best for: taking an existing asset, channel, offer, or mechanic and forcing non-obvious variants.
Not for: generating a concept from scratch.

Run each lens against the input. Discard any lens that yields nothing in 60 seconds (don't force it).
Output: 3–5 short concept seeds, one line each, labelled by lens. Flag which ICP segment each lands best
for and why.

### Six Thinking Hats (de Bono)

Best for: unblocking a decision or campaign direction where the team keeps colliding — separates
optimism, data, caution, creativity, process, and feeling into distinct passes so no angle is crowded
out.

| Hat | Lens | Question to ask |
|-----|------|-----------------|
| White | Data / facts | What do we actually know about this audience or channel? |
| Red | Gut / emotion | What would this ICP feel, not just think? |
| Black | Caution | What could genuinely kill this idea? |
| Yellow | Optimism | What's the best-case scenario if this works perfectly? |
| Green | Creativity | What's the wildest version of this that could still work? |
| Blue | Process | What's the smartest sequence to test this? |

Output: one strong insight per hat. The Green hat usually produces the seedbed for further ideation;
feed its output into SCAMPER or Crazy-8s next.

### First-Principles Deconstruction

Best for: a category assumption that everyone takes for granted and nobody has questioned. The antidote
to "that's how it's always done in our space."

1. State the conventional campaign or content assumption explicitly.
2. List every sub-assumption baked inside it.
3. For each: "Is this actually true? What if it weren't?"
4. Rebuild from the atoms that survive the interrogation.

Output: 1–2 rebuilt directions that could only be generated by dismantling the assumption. Label the
assumption that was broken. Mark any unverified claims `[verify]`.

### Crazy-8s (time-boxed volume burst)

Best for: generating eight meaningfully different executions of the same concept fast, before the inner
critic kills ideas. Mimics the Google Ventures sprint format.

Constraint: 8 ideas, 8 minutes of generation (one per minute). Each must be distinct in execution
(different channel, format, hook, or mechanics — not different words for the same idea). Volume is the
point; judgment is suspended.

Output: the 8 raw ideas, one sentence each, labelled 1–8. Immediately follow with a fast filter: mark
each H (keep) / M (maybe) / L (drop) against brand fit + ICP resonance. Carry only H's into the
champion selection.

### Brainwriting 6-3-5 (asynchronous divergence in solo runs)

Best for: a solo marketer who needs the equivalent of a round-table — building on a seed idea in
multiple passes without anchoring to the first thought.

In a solo / AI-assisted run, simulate three "passes":
- **Pass 1 (you):** 3 ideas stemming directly from the prompt.
- **Pass 2 (challenge):** for each Pass 1 idea, generate a stronger or weirder variation.
- **Pass 3 (combination):** combine any two Pass 2 ideas into a single hybrid concept.

Output: 3 seed ideas, 3 variations, 1–2 hybrids. A good hybrid is the most common origin of a
genuinely new campaign concept.

### Lateral Provocations (Edward de Bono's PO technique)

Best for: forcing a break from logical constraints when every other framework circles back to the same
answer.

Generate a deliberate, illogical provocation — something that sounds wrong but creates a new mental
path. Format: "PO: [impossible or backwards statement]." Then mine it: "If that were somehow true,
what would we have to do / what new feature/message/channel would it suggest?"

Three provocations per run. Mine each for at least one non-obvious direction. You don't have to use the
provocation — you use the direction it opens.

---

## Converge: champion pick and handoff

After generating (Quick: 1–2 frameworks; Full: all 6), apply a three-lens filter to every surviving
idea:

1. **ICP resonance** — does this land for the actual person in the brand's `brand.md`, at their
   awareness stage? Not a hypothetical buyer.
2. **Brand permission** — voice, banned words, and offer mechanics honored? Unconfirmed proof is
   `[verify]`.
3. **Executability** — can a small team deliver this in the available channel/format/budget without
   dependencies that haven't been approved? If uncertain, note the constraint rather than dropping it.

Pick 1 champion and 1–2 strong alternates. For the champion: state the concept seed (1–2 sentences),
name the framework it came from, give the ICP hook, and flag the next logical downstream skill
(`campaign-concept-developer` to develop fully, `headline-hook-generator` for top-of-funnel hook, etc.).

Save a Full board output to `./ideation/[brand-slug]-[date].md` if asked or if the session generated
more than 20 ideas. Quick runs are inline.

---

## Principles

- **Brand-brain first. Always.** No idea generation before the brand context is loaded. Voice +
  banned words are hard overrides.
- **Frameworks are lenses, not assembly lines.** Run the lens that fits the problem; skip lenses that
  don't. Never force output to fill a template.
- **Diverge then converge — in that order.** Judgment during generation kills the best ideas. Suspend
  it explicitly until the filter pass.
- **Different execution, not different words.** Volume only counts if the ideas are genuinely distinct
  in angle, channel, or mechanic — not lexical variations.
- **ICP grounding over cleverness.** A brilliant idea that doesn't land for the specific person in
  `brand.md` is a wrong answer. Filter ruthlessly.
- **Truth discipline.** Unverified numbers, unconfirmed proof claims, or invented differentiation get
  `[verify]` — not omission, not fabrication.

---

## What not to do

- Don't generate ideas before `brand-brain` returns the active brand context.
- Don't run all six frameworks on every request — match mode to the scope; default to Quick.
- Don't produce near-identical ideas and label them as distinct angles. If you can't find 3
  genuinely different directions, say the brief is too narrow and ask for a wider aperture.
- Don't write final copy, a full brief, or a test spec here — hand off to the right downstream skill.
- Don't invent proof, stats, or competitor claims to make an idea sound stronger; mark uncertain
  claims `[verify]`.
- Don't use emojis or exclamation marks unless the brand's voice explicitly allows them.
- Don't bury the champion in a flat list of eight — it should be clearly labelled and easy to hand off.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or bootstrapped) before any idea generation?
- Mode matched to request: Quick (1–2 frameworks, 3–5 ideas) or Full board (all 6)?
- Each surviving idea is genuinely distinct in angle, channel, or mechanic — not a synonym?
- Every idea passes the ICP resonance filter against the real `brand.md` persona?
- Voice honored; banned words absent; unconfirmed proof marked `[verify]`?
- Champion clearly identified with framework origin, ICP hook, and downstream handoff named?
- Full board (>20 ideas) offered for save to `./ideation/[brand-slug]-[date].md`?
