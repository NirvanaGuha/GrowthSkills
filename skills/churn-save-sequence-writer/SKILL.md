---
name: churn-save-sequence-writer
description: >
  Takes a cancellation-intent trigger (downgrade click, cancel-survey submission, inactivity
  signal, billing failure, or a manual segment) plus the brand's available retention offers and
  writes a 2–3 step churn-save sequence: a save attempt at the cancel moment, one or two
  follow-up messages, and an escalation routing decision. Every message is on-brand voice, matched
  to the cancellation reason, and anchored to a specific offer — never a generic "please don't go."
  Calls brand-brain for voice/ICP/offers, cta-variant-generator for the save CTA, and
  usage-triggered-message-sequencer for trigger conditions and delay logic. Output saves to
  ./retention/churn-save/ ready for your ESP, in-app tool, or CRM workflow. Use when the user says
  "write a churn save," "cancellation save email," "win back before they leave," "at-risk sequence,"
  "save flow," "retention email when they cancel," "cancel page offer," or hands over a reason
  code and asks for the save message.
---

# Churn-Save Sequence Writer

A cancellation click is a conversation, not a door slamming. This skill writes the 2–3 messages
that happen inside that window — a tailored save attempt the moment intent is detected, an
optional follow-up if they leave anyway, and an escalation path for high-value accounts. Every
word is matched to the declared cancellation reason, the user's lifecycle stage, and the brand's
real retention offers. No generic "we'd hate to see you go."

This skill writes the copy and sequence logic. It does not build the automation workflow
(that is `usage-triggered-message-sequencer`) or design the cancel-survey form. If the save offer
is weak or the cancellation reason signals a product gap, this skill names it instead of hiding it
in a discount.

---

## Skills this calls

- **`brand-brain`** (required, first) — voice, banned words, ICP, offer/pricing, real proof,
  positioning. Never write a word before it returns.
- **`cta-variant-generator`** — generates the save-moment CTA (button label + friction-reducer
  microcopy) for each message in the sequence.
- **`usage-triggered-message-sequencer`** — (call when the user wants the full trigger spec, not
  just copy) returns the event conditions, delays, and channel routing to wire the sequence into
  an ESP or CRM workflow.
- *(optional)* `offer-pricing-brain` — if retention offers or discount tiers are undocumented.
- *(optional)* `proof-vault` — real social proof and outcome statements to anchor save messages.
- *(optional)* `objection-library-builder` — objection/reframe pairs when reason codes map to
  known objections.

---

## How a run works

```
Step 0  Load brand + offers  ──► call brand-brain; extract retention offers + proof
Step 1  Classify the trigger  ──► reason code → save archetype → offer tier
Step 2  Map the sequence      ──► 2–3 steps, channels, delays, escalation routing
Step 3  Write each message    ──► subject/push title, body, CTA per step
Step 4  CTA variants          ──► call cta-variant-generator for save-moment CTA
Step 5  Self-review + output  ──► check offer integrity, reason match, voice
```

---

## Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). Extract from the returned
digest:

- **Voice + banned words** — hard overrides on every line of copy.
- **ICP + customer lifecycle signals** — what "high value" looks like (tenure, plan, seat count,
  revenue tier) to determine escalation routing.
- **Retention offers** — pause option, downgrade path, discount, extended trial, a success
  manager call, or a feature unlock. Use only real offers; mark anything unconfirmed `[verify]`.
- **Real proof** — outcome stats, customer results; `[verify]` anything unconfirmed.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + that brand's
`brand.md`. If none exists, ask the user: product and plan structure, ICP + churn risk profile,
available retention offers (discount / pause / downgrade / human call), and 3 voice adjectives
+ banned words. Prefer the call.

---

## Step 1 — Classify the trigger and map to a save archetype

The framework driving every message is **Reason → Archetype → Offer Tier**. A generic save
message loses; a reason-matched one has something real to say.

### Reason codes → archetypes

| Reason code | Save archetype | Lever |
|---|---|---|
| Too expensive / pricing | Value reframe → downgrade or discount | Show ROI before the discount |
| Not using it / unclear value | Activation rescue | Surface the one feature that creates the aha moment for their use case |
| Missing a feature | Product-gap acknowledgment | Roadmap note or alternative workaround; never overpromise |
| Switching to a competitor | Competitive reframe | One specific advantage; call `objection-library-builder` |
| Business circumstances (budget, shutdown, hiring freeze) | Pause / defer | Offer pause or future re-entry without penalty |
| Service / support issue | Recovery + human touch | Apology + escalation to CSM or founder; no discount until trust is restored |
| Unspecified / no survey | Curiosity open + retention offer | Ask one question; lead with the highest-value offer |

If the trigger is a billing failure (not voluntary cancellation), do not use save-sequence copy —
write a dunning recovery message (payment recovery, not churn-save).

### Offer tier assignment

Match offer to reason and account value:

- **Tier 1 (high-value account):** personal outreach (CSM/founder call) + strongest offer.
- **Tier 2 (mid-tier):** targeted discount or plan adjustment + a specific ROI statement.
- **Tier 3 (SMB / low-tenure):** self-serve save (pause/downgrade/resource) + no human loop
  unless they respond.

Ask the user (or infer from `brand-brain`) which tier the triggering segment falls into.

---

## Step 2 — Map the sequence structure

Default structure. Adjust channel mix to what the brand has wired:

```
Message 1 — Save Moment  (in-app modal or email, at trigger, or ≤10 min after)
  ↓ if no action within [X hrs/days]
Message 2 — Follow-Up    (email, 1–3 days post-trigger; often the human touch or richer proof)
  ↓ if still no action
Message 3 — Escalation or Exit  (email/in-app, 5–7 days post-trigger)
  → Tier 1: route to CSM; Tier 2/3: final offer + graceful goodbye
```

Two-message version (simpler products or short save windows): collapse Messages 2 and 3.

For each step, specify:
- **Channel** (in-app modal, email, SMS if available)
- **Trigger condition** (event name + property, or time delay from cancellation intent)
- **Offer surfaced** (from Tier assignment above)
- **Escalation flag** (yes/no → human hand-off)

If the user wants the full trigger spec for their ESP/CRM, call
`usage-triggered-message-sequencer` and pass the step map.

---

## Step 3 — Write each message

For every step, produce:

**Subject line / push title** — (call `subject-line-preview-text-optimizer` if the user wants
variants; otherwise write one recommended subject + 1 alternate)

**Body copy** — follow the **CARER structure** for save messages:

```
C — Confirm the decision / acknowledge (show you heard them; no guilt-trip)
A — Address the stated reason directly (one sentence; reason-matched)
R — Reframe or reveal (specific value they may not know; real proof or outcome)
E — Extend the offer (pause / discount / call / resource — exactly one, clearly stated)
R — Route clearly (one CTA; what happens next, no surprises)
```

Length norms:
- Message 1 (save-moment modal): 40–80 words body; CTA is the focal point.
- Message 2 (follow-up email): 80–150 words; add proof or social validation.
- Message 3 (escalation/exit): 60–120 words; if exit, end gracefully (leave the door open;
  no passive-aggression).

Voice: obey the brand's adjectives and banned words throughout. Empathy without begging.
Specificity without data dumps.

---

## Step 4 — Save-moment CTA

**Call `cta-variant-generator`** (Skill tool) for the primary save-moment CTA (Message 1 button
label + friction-reducer microcopy). Pass: the copy, the offer, the placement (cancel modal or
email), and the brand context. Use the returned primary CTA + 1 alternate; do not generate CTAs
from scratch here.

For Messages 2 and 3, write CTA copy inline (these are softer, lower-commitment actions —
"Book a 15-min call," "See what's new," "Resume my plan" — that do not need a full variant
battery).

---

## Output format

Save to `./retention/churn-save/[brand-slug]-[reason-code].md`. Present inline as well.

```markdown
## Churn-Save Sequence — [Brand] · [Reason code]
Trigger: [event / segment description]
Account tier: [Tier 1 / 2 / 3]
Offer: [exact offer and conditions]
Brand: [slug, via brand-brain]

---
### Message 1 — Save Moment
Channel: [in-app modal / email]
Trigger: [event + delay]
Subject / Modal headline: …
Body: …
CTA: [primary] | Alt: [alternate]
Microcopy: …

---
### Message 2 — Follow-Up
Channel: [email]
Delay: [X days after trigger with no action]
Subject: … | Alt: …
Body: …
CTA: …

---
### Message 3 — [Escalation / Exit]
Channel: [email]
Delay: [X days after Message 2 with no action]
Escalation flag: [yes → route to CSM / no → self-serve exit]
Subject: … | Alt: …
Body: …
CTA: …

---
### Sequence notes
[Reason-match rationale, offer rationale, any offer integrity flags, [verify] items]
```

---

## Principles

- **Brand-brain first.** No copy before the brand digest returns. Voice and banned words are
  non-negotiable overrides.
- **Reason-matched always.** A save message that does not address the stated reason is worse than
  silence — it signals you weren't listening.
- **One offer per sequence, escalating.** Do not stack a discount on a pause on a call in one
  message. Reveal the offer in sequence; save the strongest for the escalation.
- **Real proof or nothing.** No made-up stats, no generic social-proof platitudes; `[verify]`
  anything unconfirmed.
- **Empathy, not begging.** Acknowledge the decision; make one strong case; route clearly.
  Guilt-tripping erodes brand equity and accelerates departure.
- **Leave the door open.** The exit message (if they leave) is a brand impression, not a loss.
  A graceful goodbye creates re-activation equity.
- **Name product gaps.** If the reason code is "missing feature" and there is no workaround,
  say so — do not overpromise the roadmap or discount around a real gap.

---

## What not to do

- Don't write save copy before `brand-brain` returns.
- Don't use the same message for every cancellation reason — reason-matching is the whole job.
- Don't stack multiple offers in one message — you lose escalation leverage.
- Don't apply churn-save copy to billing-failure triggers — write dunning copy instead.
- Don't promise roadmap features or delivery timelines that are not confirmed.
- Don't guilt-trip ("You'll lose everything!"), manufacture urgency, or use dark-pattern copy.
- Don't use emojis, exclamation marks, or informal sign-offs unless the brand explicitly allows
  them.
- Don't output to brand.md or the skill folder — saves go to `./retention/churn-save/`.

---

## Quality checklist

- `brand-brain` called; voice + banned words honored; only real offers and real proof used
  (unconfirmed items marked `[verify]`)?
- Reason code classified to an archetype; account tier assigned; offer tier matched?
- Sequence is 2–3 steps with explicit channels, delays, and escalation routing per step?
- Each message follows CARER structure within length norms?
- `cta-variant-generator` called for the save-moment CTA; returned primary + alternate used?
- One offer per message; strongest offer held for escalation step?
- No guilt-trip language; graceful exit message if the user leaves; door left open?
- Output saved to `./retention/churn-save/[brand-slug]-[reason-code].md`?
