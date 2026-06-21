---
name: onboarding-flow-builder
description: >
  Turns a product description, ICP, and activation milestones into a complete, ready-to-hand-off
  onboarding spec — sequenced setup checklist, welcome modal copy, product tour step scripts,
  empty-state copy for every key surface, and success/congratulations messages — grounded in the
  brand's real voice and the Jobs-to-be-Done activation logic for the target persona. It calls
  brand-brain for voice/ICP/proof context, aha-moment-activation-metric-definer to lock the
  activation milestone if not already defined, in-app-microcopy-writer-auditor for every UI string,
  and cta-variant-generator for every action button. Output is a single onboarding spec doc ready
  for engineering, design, and CS handoff. Use when the user says "design our onboarding,"
  "build an onboarding checklist," "write the product tour," "write empty states," "welcome modal
  copy," "first-run experience," "new user activation flow," "onboarding spec," or "time-to-value
  playbook for new users."
---

# Onboarding Flow Builder

The gap between signup and the aha moment is where most SaaS revenue dies. This skill closes it.
Give it the product, the ICP, and whatever activation milestones you have — and get back a complete,
engineering-ready onboarding spec: a sequenced setup checklist, welcome modal, product tour script,
empty-state copy for every hollow surface, and the micro-celebration messages that confirm the user
crossed each milestone. Every string is in the brand's voice; every milestone is anchored to a real
activation definition, not a proxy vanity metric.

Framework: **Jobs-to-be-Done activation sequencing** — each checklist step maps to a functional,
emotional, or social job the user hired the product to do, ordered so the smallest-effort step that
delivers the fastest proof of value comes first (the "first win" principle). Activation is defined
as the observable action that predicts 30-day retention — not feature breadth, not time on site.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — resolves the active brand's voice, ICP, proof points, and
  banned words. Do not write a single string before this returns.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
  brand's `brand.md` directly; if none exists, ask the user for brand name, ICP job title/pain,
  voice adjectives + banned words, and primary product destination URL before proceeding.

- **`aha-moment-activation-metric-definer`** (Step 1, if activation milestone not provided) — locks
  the single north-star activation event that predicts 30-day retention. Do not invent this; call the
  skill. If the user has already defined it, skip.

- **`icp-persona-builder`** (optional, Step 1) — when the ICP is thin (job title only, no pains or
  awareness stage), call this to flesh out the first-session mental model before writing checklist copy.

- **`in-app-microcopy-writer-auditor`** (Step 3) — generates and audits every UI string: tooltip,
  empty-state body, helper text, modal body copy, success messages. Compose rather than duplicate.

- **`cta-variant-generator`** (Step 3) — writes button labels for every checklist item CTA, modal
  primary action, and tour step forward/skip. Pass the placement + awareness context from Step 0.

- **`proof-vault`** (Step 3, optional) — pulls verified social proof for empty-state copy ("3,000
  teams already use this" type lines). Mark any unverified stat `[verify]`.

- **`welcome-onboarding-email-sequence-builder`** (Step 4, optional) — builds the parallel email track
  that runs alongside the in-product flow. Compose it as a companion, not a duplicate.

- **`usage-triggered-message-sequencer`** (Step 4, optional) — specifies the event-based push/in-app
  nudges that fire when a user stalls at a checklist step for N hours. Call rather than rebuild.

---

## How a run works

```
Step 0  Load brand         ──► brand-brain (voice, ICP, proof, banned words)
Step 1  Lock the aha       ──► aha-moment-activation-metric-definer (or use user-supplied)
Step 2  Map the checklist  ──► JTBD sequencing → milestone order → copy brief per step
Step 3  Write the surfaces ──► in-app-microcopy-writer-auditor + cta-variant-generator + proof-vault
Step 4  Companion channels ──► welcome-onboarding-email-sequence-builder + usage-triggered-message-sequencer
Step 5  Assemble the spec  ──► single onboarding-spec.md, self-review, present
```

---

## Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns: voice adjectives,
banned words, offer mechanics, real proof, positioning, and ICP + awareness tendency. If the brand
is new, `brand-brain` bootstraps it first. **Do not write copy until it returns.**

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
brand's `brand.md` directly; if none exists, ask the user for brand name, ICP job title/pain, voice
adjectives + banned words, and primary product destination URL before proceeding.

---

## Step 1 — Lock the activation milestone

The activation milestone is the single observable event that causally predicts 30-day retention
for the ICP. It is specific, measurable, and meaningful — not "completed onboarding" or "logged in
twice." Examples: "connected first data source," "sent first push notification," "imported product
catalog," "invited one teammate."

- If the user has supplied a clear activation milestone → confirm it in one line and proceed.
- If unclear or absent → invoke `aha-moment-activation-metric-definer`. Pass the product, ICP, and
  any retention data the user provides. Use what it returns as the flow's north star.

Everything downstream — checklist order, empty-state copy, success messages — is in service of
moving the user to this one milestone as fast as possible.

---

## Step 2 — Map the checklist (JTBD sequencing)

Build the checklist by answering three questions per candidate step:

1. **Which job does this step complete?** (Functional / emotional / social)
2. **How much effort does it require?** (Low = friction < 2 min; High = requires external asset or
   decision)
3. **What is the first-proof-of-value step?** — the lowest-effort step that produces visible output
   the user can point to. This goes first.

**Sequencing rules (non-negotiable):**
- First-proof step comes first, even if it is not logically "first" in the product flow.
- No step before the user has seen value at least once (no forced setup before the demo/payoff).
- Checklist length: 3–7 steps. More than 7 predicts abandonment; fewer than 3 signals a trivial
  product that doesn't need a checklist.
- Each step has: title (≤6 words), one-line description, CTA label (from `cta-variant-generator`),
  estimated time to complete, and the milestone it unlocks.
- Mark which step is the activation milestone with `[AHA]`.

**Produce a checklist brief** (step title, JTBD job, effort, time, milestone) before writing copy
for Step 3. If inputs are too thin to sequence confidently, ask two questions maximum:
(1) what is the very first thing a new user needs to do to see value? and (2) what do your best
3-month users have in common that trial users don't?

---

## Step 3 — Write the surfaces

For each surface, delegate to the appropriate sibling skill and assemble the result. Do not
re-implement their logic here.

### Welcome modal
The first thing the user sees after signup. Job: reduce cognitive load, restate the value promise,
and give one clear next action — not five.

Structure:
- **Headline:** the transformation the user just chose to make (not "Welcome to [Product]").
  ≤8 words, first-person or second-person, from the brand voice.
- **Body:** 1–2 sentences. What they'll be able to do in the next 5 minutes. Real, specific,
  achievable. Use proof from `proof-vault` if verified.
- **Primary CTA:** from `cta-variant-generator`. One action, value-led, low-commitment ceiling for
  awareness stage.
- **Secondary link:** "Take a quick tour" or "I'll explore on my own" — never "Skip" alone.

### Onboarding checklist copy (per step)
For each step in the map from Step 2:
- **Step title** (≤6 words)
- **Helper text** (1 sentence, explains the "why" not the "how")
- **CTA label** (from `cta-variant-generator`)
- **Completed state** (short past-tense confirmation, ≤5 words: "Connected. You're ready.")

Delegate body microcopy and tooltip text to `in-app-microcopy-writer-auditor`.

### Product tour steps (if requested or implied)
A product tour supplements the checklist — it shows; the checklist does. Structure each tour step:
- **Step number + surface** (e.g., "Step 2 — Dashboard")
- **Spotlight copy:** ≤2 sentences pointing to one UI element. What it does + why it matters now.
- **Forward CTA** (from `cta-variant-generator`): "Next →" is always wrong; use the action.
- **Skip path:** every tour must have a graceful exit. Never dead-end.

Cap at 5 steps. Tours longer than 5 steps are feature demos, not onboarding.

### Empty-state copy (per key surface)
Empty states are the highest-leverage copy in onboarding — the user is staring at a blank
canvas and deciding whether to continue or quit.

For each key empty surface (minimum: dashboard, main list/feed, first major feature surface):
- **Headline:** ≤8 words, action-oriented or value-priming (not "Nothing here yet").
- **Body:** 1–2 sentences. What this surface will look like when populated, or why it is worth
  filling. Use a verified proof point from `proof-vault` if available.
- **Primary CTA** (from `cta-variant-generator`): the single action that populates this surface.
- **Optional illustration alt-text:** one sentence for designers.

Delegate copy drafts to `in-app-microcopy-writer-auditor`; pass the empty-state context explicitly.

### Success / milestone messages
Write one message per checklist step completion + one for the activation milestone `[AHA]`.

- **Step completion:** inline, dismissible. ≤12 words. Past tense + immediate next context.
  Example: "First push sent. Your audience is live — see it in Reports."
- **AHA milestone message:** modal or prominent banner. 2–3 sentences. Name what they did,
  quantify the outcome if the brand has real proof (`[verify]` otherwise), and tee the next
  growth action (invite a teammate, explore the next feature, share a win).

---

## Step 4 — Companion channels (optional)

If the user wants a parallel email or push track:
- Invoke `welcome-onboarding-email-sequence-builder` for the email drip (D0 welcome → D1 first-win
  follow-up → D3 checklist-stall nudge → D7 activation summary).
- Invoke `usage-triggered-message-sequencer` for event-based nudges (stall at step N for >24h →
  in-app tooltip or push; completed AHA → upsell prompt after 48h).

Coordinate timing so email and in-product messages don't overlap on the same day with the same ask.

---

## Step 5 — Assemble and save the spec

Compile everything into a single file at `./onboarding/onboarding-spec.md` (project-relative CWD,
never inside the skill folder). Structure:

```
# Onboarding Spec — [Brand] — [date]
## Activation milestone (north star)
## Onboarding checklist (sequenced, JTBD-mapped)
## Welcome modal copy
## Product tour script (if built)
## Empty-state copy (per surface)
## Success + milestone messages
## Companion channel brief (email / push, if built)
## Open questions + [verify] items
```

Tell the user where the file was saved in one line.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No string is written before the brand is loaded. Voice + banned-words
  override every default.
- **One aha, one north star.** All copy, sequence, and success messages serve the single activation
  milestone. Never diffuse toward feature breadth.
- **First win, fast.** The lowest-effort step that produces visible value comes first in the
  checklist, always.
- **Compose, don't duplicate.** Microcopy → `in-app-microcopy-writer-auditor`. CTAs →
  `cta-variant-generator`. Proof → `proof-vault`. Email → `welcome-onboarding-email-sequence-builder`.
  Never re-implement a sibling's logic here.
- **Empty states are conversion copy.** Never write "Nothing here yet." Every empty state has a
  headline, a reason, and one action.
- **3–7 checklist steps.** Outside this range, push back before writing.
- **Verified proof or `[verify]`.** Never invent social proof numbers; mark unconfirmed stats.

---

## What Not to Do

- Don't write any copy before `brand-brain` returns the active brand.
- Don't invent the activation milestone — call `aha-moment-activation-metric-definer` or ask.
- Don't build a checklist of more than 7 steps without flagging the abandonment risk.
- Don't put feature education before the first-win step; that is a feature demo, not onboarding.
- Don't write "Welcome to [Product]!" as a modal headline — it is a wasted line.
- Don't duplicate email copy in-product or in-product copy in email on the same day.
- Don't overwrite `brand.md` or store mutable data inside the skill folder.
- Don't write empty states that apologize ("Nothing here yet," "Looks a bit lonely here") —
  reframe every blank canvas as an invitation.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called first; voice + banned-words honored throughout; only verified proof used?
- Activation milestone defined, specific, and measurable (predicts retention, not vanity)?
- Checklist: 3–7 steps, JTBD-sequenced, first-win step first, `[AHA]` step marked?
- Each checklist step has title (≤6 words), helper text, CTA (from `cta-variant-generator`),
  and completed-state confirmation?
- Welcome modal: transformation headline + specific body + one primary CTA + graceful exit?
- Every key surface has an empty state with action-oriented headline + body + CTA?
- Success messages: inline for each step + prominent modal/banner for AHA milestone?
- No "Welcome to [Product]!", "Nothing here yet," or fabricated proof?
- Spec saved to `./onboarding/onboarding-spec.md`; open `[verify]` items listed?
- Companion email/push track coordinated (no same-day overlap) if built?
