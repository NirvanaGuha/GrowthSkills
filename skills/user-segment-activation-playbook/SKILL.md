---
name: user-segment-activation-playbook
description: >
  Takes a user segment definition (by acquisition source, use-case, plan tier, industry, or
  behavioral trait) plus the product's feature set and turns both into a complete, segment-specific
  activation playbook: the job-to-be-done lens for that segment, the single Aha Moment to target,
  an ordered sequence of first-session actions (with in-product triggers), segment-tailored messaging
  angles for every channel, and concrete success milestones the team can instrument. Calls brand-brain
  for voice and ICP, aha-moment-activation-metric-definer for instrumented success metrics,
  icp-persona-builder when the segment lacks a persona, onboarding-flow-builder for the full
  onboarding spec if needed, cta-variant-generator for first-action CTAs, in-app-microcopy-writer-auditor
  for contextual prompts, usage-triggered-message-sequencer for trigger-based follow-up, and
  lifecycle-email-push-copy-reviewer to review the produced sequences before final output.
  Use when the user says "activation playbook," "segment onboarding," "first-session flow for [segment],"
  "how do we activate [cohort]," "what's the Aha Moment for [persona]," "build an activation map,"
  or hands over a CRM segment definition and asks what to do with it.
---

# User Segment Activation Playbook

One playbook for all segments is the fastest path to mediocre activation. This skill builds a
**segment-specific playbook** — grounded in the segment's real job-to-be-done, anchored to a single
instrumented Aha Moment, sequenced through first actions the segment will actually take, and written
in the brand's real voice. Every output is composable: the copy goes to channel skills, the trigger
spec goes to engineering, the milestones go to your analytics stack.

The framework is **Jobs-to-Be-Done (JTBD) → Aha Moment → First-Action Ladder → Messaging Map →
Milestone Dashboard**. Each layer is derived from the prior one; change the segment definition and
the whole playbook regenerates consistently.

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads the active brand's voice, ICP, offer, banned
  words, and real proof. This skill does not implement brand resolution or scanning.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
  brand's `brand.md` directly; if none exists, ask the user for brand voice adjectives, ICP description,
  and primary product offer before proceeding.
- **`aha-moment-activation-metric-definer`** — defines and instruments the Aha Moment for the segment;
  do not hand-wave the Aha Moment without calling this skill.
- **`icp-persona-builder`** — call when the segment lacks a documented persona; passes scanned context
  so it doesn't re-interview what's already known.
- **`onboarding-flow-builder`** — call when the user wants the full onboarding spec (checklist,
  modal, tour script); this skill produces the activation map that feeds it.
- **`cta-variant-generator`** — generates first-action CTAs for each step in the First-Action Ladder.
- **`in-app-microcopy-writer-auditor`** — writes contextual empty-state, tooltip, and progress copy
  for each ladder step; compose rather than re-draft inline.
- **`usage-triggered-message-sequencer`** — turns the trigger spec in the playbook into the actual
  triggered message sequence.
- **`lifecycle-email-push-copy-reviewer`** — reviews any email/push copy this playbook produces
  before finalizing; do not self-approve in the same pass.

---

## How a run works

```
Step 0  Load brand context    ──► call brand-brain
Step 1  Resolve the segment   ──► parse definition; call icp-persona-builder if no persona
Step 2  Define the Aha Moment ──► call aha-moment-activation-metric-definer
Step 3  Build the playbook    ──► JTBD lens → First-Action Ladder → Messaging Map → Milestones
Step 4  Compose copy + spec   ──► call cta-variant-generator, in-app-microcopy-writer-auditor
Step 5  Review                ──► call lifecycle-email-push-copy-reviewer on email/push output
Step 6  Output + save         ──► inline playbook; offer usage-triggered-message-sequencer handoff
```

---

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns the active brand's
digest: voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning,
ICP + awareness tendency. Obey the returned voice and banned-words as hard overrides. Use only
confirmed proof — mark anything else `[verify]`.

**Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
brand's `brand.md` directly; if none exists, ask the user for brand voice adjectives, ICP description,
and primary product offer before proceeding.**

---

### Step 1 — Resolve the segment

Accept any segment definition the user provides: a CRM filter (source = "organic-blog", plan = "free"),
a behavioral cohort (users who completed step 1 but not step 2), a use-case tag (e-commerce store
owners), or a named persona. Extract or infer:

- **Primary job-to-be-done** — what outcome does this segment hired the product for? Use their
  language, not the product's feature names.
- **Awareness at entry** — how much do they already know about the product? (Maps to Schwartz
  awareness ladder: unaware → most-aware.) This governs message ceiling for every channel.
- **Friction hypotheses** — where similar segments historically stall (from brand proof, JTBD
  interviews, or `[verify]` if unknown).

If the segment has no existing persona, call `icp-persona-builder` before continuing.

---

### Step 2 — Define the Aha Moment

**Call `aha-moment-activation-metric-definer`** with the resolved segment + product feature set. It
returns:

- The single Aha Moment event (the in-product action most predictive of retention for this segment).
- The instrumented metric definition (event name, properties, threshold, window).
- Proxy leading indicators for users not yet at Aha.

**Do not proceed to Step 3 without an instrumented Aha Moment definition.** Playbooks built around
vague "value moments" produce dashboards you can't act on.

---

## The JTBD → Aha → Ladder → Map → Milestones framework

### Layer 1: JTBD Lens

State the segment's functional, emotional, and social job in one sentence each. Example structure:

> **Functional:** [Segment] hires [product] to [specific outcome] so they can [downstream benefit].
> **Emotional:** They want to feel [adjective] rather than [adjective they feel now].
> **Social:** They want [peers/boss/customers] to see them as [identity shift].

This lens governs every downstream message. If a messaging angle doesn't serve one of these jobs,
cut it.

---

### Layer 2: First-Action Ladder

A sequenced list of ≤7 actions from first login to Aha Moment. Each rung has:

| # | Action | Trigger condition | Success signal | Stall signal | Next nudge |
|---|---|---|---|---|---|
| 1 | [verb + object] | [session start / time elapsed / prior action] | [event] | [timeout / 0-progress] | [in-app prompt / email / push] |

Rules:
- Each action must be completable in ≤10 minutes (respect early-session attention budget).
- Actions are ordered by dependency, not by what you want the user to do first.
- Every action has a stall signal and a recovery nudge — activation is a loop, not a funnel.
- Call `cta-variant-generator` for the CTA at each rung. Call `in-app-microcopy-writer-auditor` for
  empty-state and tooltip copy at each rung. Do not write these inline; compose the sibling outputs.

---

### Layer 3: Messaging Map

For each channel, one segment-specific angle per stage. Do not reuse the same message across
channels — the medium changes what lands.

| Stage | Channel | Message angle | Key proof point | Awareness ceiling | CTA |
|---|---|---|---|---|---|
| Pre-Aha (< 24h) | In-app tooltip | [JTBD-specific] | [real proof or `[verify]`] | [stage] | [from cta-variant-generator] |
| Pre-Aha (< 24h) | Email | | | | |
| Stalled (no step 2 in 48h) | Push | | | | |
| Post-Aha | Email | | | | |
| Day 7 | In-app | | | | |

Voice, banned words, and proof constraints come from `brand-brain`. Mark any unconfirmed numbers
`[verify]`. Keep awareness ceiling discipline: don't ask for a credit card from someone who hasn't
hit Aha yet.

---

### Layer 4: Success Milestones

Three tiers of milestones, all instrumented:

**Leading indicators (Days 1–3):** proxy signals that predict whether the user will hit Aha.
Examples: completed ≥2 ladder steps; invited a teammate; connected an integration. Define event +
threshold + window for each.

**Aha Moment:** the event defined in Step 2 — the single binary milestone this playbook is built
around. Every upstream action points here.

**Retention proof (Days 7, 14, 30):** downstream signals that validate the Aha Moment was durable.
Examples: returned ≥3 times in 7 days; created ≥N objects; upgraded plan. Each needs an event name
and threshold — not a vague description.

Format as a table the team can drop into a GA4 Audience, Mixpanel Funnels, or a SQL query:

| Milestone | Event name | Property filter | Threshold | Window | Owner |
|---|---|---|---|---|---|

---

## Principles (Non-Negotiable)

- **JTBD first, features second.** Every message angle must trace back to the segment's job.
  Feature names are not benefits; translate them.
- **One Aha Moment per segment.** Multiple "value moments" dilutes the playbook into a feature tour.
  If the segment genuinely has two distinct use-cases, split it into two segments and two playbooks.
- **Ladder, not funnel.** Each rung has a stall condition and a recovery path. An unrecovered stall
  is a churn event in waiting.
- **Awareness ceiling discipline.** Match the commitment ask to how much trust you've earned so far.
  Never ask for upgrade commitment before Aha is confirmed.
- **Instrument before you ship.** Every milestone in Layer 4 must have an event name before the
  playbook is considered complete. `[verify]` on instrumentation is a blocker, not a footnote.
- **Brand-brain first.** No copy or CTAs before `brand-brain` returns. Voice and banned-words are
  overrides, not suggestions.
- **Compose don't duplicate.** CTAs come from `cta-variant-generator`, in-app copy from
  `in-app-microcopy-writer-auditor`, triggered sequences from `usage-triggered-message-sequencer`,
  reviews from `lifecycle-email-push-copy-reviewer`. Never re-implement those here.

---

## What not to do

- Don't write a single generic playbook and call it segmented by changing one word.
- Don't define the Aha Moment by committee consensus ("users love when they see value") — it must
  be an instrumented event with a measurable threshold.
- Don't skip the stall signals — a ladder without recovery paths is a funnel doc, not an activation
  playbook.
- Don't invent proof points, conversion benchmarks, or competitor comparisons — use real data from
  `brand-brain` and `proof-vault`, or mark `[verify]`.
- Don't produce final email/push copy without running it through `lifecycle-email-push-copy-reviewer`.
- Don't merge two distinct JTBDs into one segment — split them.
- Don't use banned words from the brand digest; don't exceed awareness ceilings on CTAs.

---

## What gets saved

Save the full playbook to `./activation-playbooks/[segment-slug]-activation-playbook.md`. The file
should be self-contained (brand slug, segment definition, JTBD lens, Aha Moment event spec, Full
Ladder table, Messaging Map, Milestone table). Never overwrite `brand.md`.

Offer a handoff to `usage-triggered-message-sequencer` after output: "Ready to turn the trigger spec
into the full triggered sequence? Run `usage-triggered-message-sequencer` with this playbook as input."

---

## Quality checklist

- `brand-brain` called before any copy or CTAs; voice + banned-words honored throughout?
- Segment has a JTBD lens (functional, emotional, social) in the segment's own language?
- `aha-moment-activation-metric-definer` called; Aha Moment is a named event with threshold + window?
- First-Action Ladder has ≤7 rungs, each with trigger condition, success signal, stall signal, and
  next nudge?
- `cta-variant-generator` called for each ladder CTA; `in-app-microcopy-writer-auditor` called for
  in-product copy?
- Messaging Map covers ≥3 stages, ≥2 channels, awareness ceiling noted per row?
- Milestone table has event names + thresholds for all three tiers (leading, Aha, retention)?
- All proof confirmed real or marked `[verify]`; no invented benchmarks?
- `lifecycle-email-push-copy-reviewer` called before finalizing any email/push copy?
- Playbook saved to `./activation-playbooks/[segment-slug]-activation-playbook.md`?
