---
name: usage-triggered-message-sequencer
description: >
  Product events + lifecycle stage → trigger-based message sequence with event conditions,
  delays, and channel routing. Give it a set of product events (or a plain-English description
  of user behavior) plus the lifecycle stage you want to move users through, and it outputs a
  complete trigger-based sequence: event conditions and entry/exit logic, delay windows,
  channel routing (email, push, in-app, SMS), message copy scaffolds, suppression rules, and a
  sequence diagram. Anchored to the Hook Model (trigger → action → variable reward → investment)
  for the engagement layer and to AARRR for lifecycle placement — so every sequence has a
  behavioral reason to exist, not just a "day 3 drip" timer. Calls brand-brain for voice and
  proof so copy scaffolds are already on-brand. Integrates with lifecycle-journey-mapper for
  stage definitions, push-notification-copy-generator and full-email-push-asset-builder for
  finished channel copy, welcome-onboarding-email-sequence-builder for onboarding sequences,
  and abandon-flow-writer for cart/intent abandonment. Use when the user says "set up
  trigger-based messaging," "build a behavior sequence," "when a user does X send Y," "activation
  drip," "usage-based campaign," "in-app triggered messages," "product-led nurture," or hands
  you an event list and asks what to send.
---

# Usage-Triggered Message Sequencer

Product behavior is the most honest signal you have. This skill turns event streams into
purposeful, timed, channel-routed sequences — not arbitrary drip timers. Every trigger has a
behavioral hypothesis; every message has a single job.

Scope: sequence design and copy scaffolds. Finished production copy for a channel → delegate
to the relevant copy skill. Activation milestone and aha-moment definition → delegate to
`aha-moment-activation-metric-definer` first if those are not already settled.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — voice, banned words, ICP, offer, proof. Do not write
  any copy scaffold before this returns.
- **`lifecycle-journey-mapper`** — stage definitions, entry/exit criteria, and milestone events
  if the caller hasn't mapped the lifecycle yet. Call when stage definitions are absent or vague.
- **`aha-moment-activation-metric-definer`** — confirms the activation milestone the sequence
  must drive toward. Call when the aha moment is undefined or contested.
- **`push-notification-copy-generator`** — finished push copy for each sequence node.
- **`full-email-push-asset-builder-copy-responsive-html`** — finished email copy + HTML for
  each email node.
- **`welcome-onboarding-email-sequence-builder`** — delegate full onboarding email sequences
  rather than rebuilding them here.
- **`abandon-flow-writer`** — delegate cart or intent-abandonment sequences.
- **`in-app-microcopy-writer-auditor`** — tooltip/banner/empty-state copy for in-app nodes.
- **`esp-map-platform-builder`** — platform-specific delivery spec (Braze, Klaviyo, Intercom,
  Customer.io) once the sequence logic is finalized.
- **`a-b-multivariate-test-designer`** — test design for any node you want to experiment on.
- **`experiment-results-analyzer`** — analyze results after the sequence runs.

---

## How a run works

```
Step 0  Load brand context    ──► call brand-brain
Step 1  Lock the lifecycle     ──► confirm stage + activation milestone (call lifecycle-journey-mapper / aha-moment-activation-metric-definer if needed)
Step 2  Map the events         ──► build the event inventory with entry/exit logic
Step 3  Design the sequence    ──► Hook Model framing → node map → channel routing → delays
Step 4  Write copy scaffolds   ──► on-brand, single-job per node; delegate finished copy to channel skills
Step 5  Add suppression rules  ──► goal-achieved exits, frequency caps, re-entry gates
Step 6  Output the spec        ──► sequence diagram + delivery table + test suggestions
```

---

## Step 0 — Load brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns voice adjectives, banned
words, offer mechanics, proof, ICP, and positioning. Do not write copy scaffolds before it
returns. Obey voice + banned-words as hard overrides; use only real proof (else `[verify]`).

**Fallback if brand-brain is absent:** read `~/.brandbrain/brands/.active` and that brand's
`brand.md`. If none exists, ask the user to install `brand-brain` or supply a 4-field
mini-setup (product · ICP · offer · 3 voice adjectives + banned words) before proceeding.

---

## Step 1 — Lock the lifecycle stage

Confirm which AARRR stage the sequence operates in:

| Stage | Job of the sequence |
|---|---|
| Activation | Drive users to the aha moment before trial expires |
| Engagement | Deepen habit formation; move light users to power-user behavior |
| Retention | Interrupt disengagement before it becomes churn |
| Revenue | Trigger upgrade, upsell, or renewal at the moment of peak value |
| Referral | Surface share prompts at the moment of highest satisfaction |

If stage definitions or the activation milestone are undefined, call `lifecycle-journey-mapper`
and/or `aha-moment-activation-metric-definer` first. Never design a sequence against a vague
or assumed stage.

---

## Step 2 — Map the events

Build an event inventory from the user's inputs. For each signal, define:

| Field | What to capture |
|---|---|
| Event name | Exact identifier (`user_installed`, `report_exported`, `payment_failed`) |
| Entry condition | The logical predicate that fires the trigger (event + optional property filters) |
| Negative signals | Events that prove the user already got value (suppress/exit immediately) |
| Recency window | How fresh must the event be to qualify (first-time, Nth-time, within 7d) |

Group events into **trigger tiers:**
- **High-intent** — a specific value action that predicts retention (export, invite, connect data source)
- **Inactivity** — absence of a high-intent event within a threshold window
- **Risk** — cancellation-intent, payment failure, repeated friction events
- **Milestone** — Nth session, 30-day anniversary, plan limit hit

If the user hasn't mapped their events, ask three diagnostic questions:
1. What action, when first completed, reliably predicts a user will still be active in 30 days?
2. What does a disengaged but not-yet-churned user look like in your event log?
3. What is the last event before most users cancel?

---

## Step 3 — Design the sequence (Hook Model framing)

Anchor every sequence to the **Hook Model** (Nir Eyal): external trigger → action → variable
reward → investment. This prevents the common failure mode of "sends that inform but don't
change behavior."

For each sequence node:

**Trigger layer**
- External trigger: the message itself (what the user sees in push/email/in-app)
- Internal trigger: the emotional state the message targets (frustration, FOMO, curiosity,
  pride in progress)

**Action layer**
- The single behavior the message asks for. One CTA. No optionality blur.

**Variable reward layer**
- What the user gets that is proportional to their effort but not entirely predictable
  (new insight, social proof, progress visualization, an unlocked feature)

**Investment layer**
- What the action deposits that makes the product stickier (data, configuration, a saved item,
  a connection)

If a proposed node has no clear investment layer, flag it. Information-only sequences have no
Hook and typically underperform.

**Channel routing logic** — assign each node a primary channel:

| Signal type | Primary channel | Rationale |
|---|---|---|
| Time-sensitive / real-time | Push notification | Immediacy; short attention window |
| High-complexity / decision-heavy | Email | Space to explain; asynchronous |
| In-session / contextual | In-app banner/tooltip | Zero context-switch |
| High-value risk event | Email + push (same node, staggered 30 min) | Redundancy warranted |

**Delay logic** — default heuristics (adjust per product cadence):

| Stage | First touch | Follow-up gap | Max sequence length |
|---|---|---|---|
| Activation | Within 15 min of trigger | +24h, +72h | 5–7 nodes / 14 days |
| Engagement | Same-session (in-app) or +1h | +3d, +7d | 3–5 nodes |
| Retention | At inactivity threshold | +3d, +7d | 3–4 nodes |
| Revenue | At event (limit hit, trial day 7) | +24h, +48h | 2–3 nodes |

---

## Step 4 — Copy scaffolds

Write a scaffold for each node: subject/push title · preview/subtitle · single body
paragraph · CTA label · suppression note. Use the brand's voice and real proof. Every scaffold
follows the Hook Model framing from Step 3: the subject line is the external trigger; the body
builds toward the action; the CTA is the action.

Mark any unconfirmed proof or metric `[verify]`.

**Delegate finished copy** to the relevant channel skill:
- Push nodes → `push-notification-copy-generator`
- Email nodes → `full-email-push-asset-builder-copy-responsive-html`
- In-app nodes → `in-app-microcopy-writer-auditor`

Do not write production-quality HTML or full-length email bodies here — that is the channel
skill's job. Scaffolds are sufficient for sequence design; production copy comes from the
downstream call.

---

## Step 5 — Suppression rules (non-negotiable)

Every sequence must specify:

- **Goal-achieved exit:** the event that proves the user completed the sequence's objective.
  When fired, exit immediately — do not send remaining nodes.
- **Frequency cap:** max messages per user per day across all sequences (default: 2/day,
  1/day for retention stage).
- **Re-entry gate:** how long before a user can re-enter the same sequence (default: 30 days
  for the same trigger event).
- **Global suppression:** paying customers excluded from activation sequences; churned users
  excluded from engagement sequences; recent purchasers excluded from upsell sequences for
  N days.

Missing suppression rules are the most common reason trigger sequences annoy instead of help.
Flag any sequence the user proposes without a clear goal-achieved exit.

---

## Step 6 — Output

Produce:

1. **Sequence diagram** — ASCII or Mermaid flowchart: trigger → nodes → exits. Label each
   node with channel, delay, and Hook stage (T/A/R/I).
2. **Delivery table** — one row per node:

```
| Node | Trigger / Delay | Channel | Hook Stage | Subject / Title | CTA | Exit if |
```

3. **Copy scaffolds** — one block per node (see Step 4 format).
4. **Suppression spec** — goal-achieved exit + frequency cap + re-entry gate + global rules.
5. **Test suggestions** — for the highest-leverage node, propose one A/B test using
   `a-b-multivariate-test-designer` (name the variable: subject line angle, delay window,
   channel, CTA commitment level).

Save to `./plans/[brand-slug]-[sequence-name]-sequence.md` when the user asks to save or
when the output exceeds what is comfortable inline.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No copy scaffold before the brand context loads. Voice + banned-words
  override everything here.
- **Behavior over time.** Every trigger is a behavioral signal, not a calendar tick. If you
  can't name the user behavior that fires the trigger, it is not a trigger — it is a guess.
- **One job per node.** Each message asks for exactly one action. Split-job messages split
  attention and degrade conversion.
- **Hook or cut.** If a proposed node has no investment layer, flag it before scaffolding it.
  Information-only sequences waste send capacity and train users to ignore messages.
- **Suppression is not optional.** A sequence without a goal-achieved exit is a broken sequence.
- **Delegate production copy.** Scaffolds here, finished copy in the channel skill. Don't
  rebuild what those skills do better.
- **Stage before sequence.** Never design message logic against an undefined or assumed
  lifecycle stage. Lock it first.

---

## What Not to Do

- Don't write copy scaffolds before `brand-brain` returns the active brand.
- Don't define "trigger" as a day number ("day 3"). Day-number drips are time-based, not
  usage-triggered. Name the behavior.
- Don't assign multiple CTAs to one node. Optionality is friction.
- Don't omit the goal-achieved exit rule. Sending node 4 to a user who already converted is
  the most visible sign of a broken sequence.
- Don't write production email HTML here — call `full-email-push-asset-builder-copy-responsive-html`.
- Don't design sequences without knowing the lifecycle stage. "Activation" and "retention"
  sequences for the same trigger event are completely different sequences.
- Don't reimplement aha-moment or lifecycle-stage logic — call the owning skills.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded before any copy scaffold?
- Lifecycle stage confirmed; activation milestone defined or delegated?
- Each trigger is a named behavior event, not a time-only delay?
- Every node has: channel assignment · delay · Hook stage label · single CTA?
- Goal-achieved exit defined for the sequence?
- Frequency cap + re-entry gate + global suppression rules specified?
- Finished copy delegated to channel skills (not rebuilt here)?
- Sequence diagram and delivery table both present?
- At least one A/B test suggestion included for the highest-leverage node?
- Any unconfirmed metric or proof marked `[verify]`?
