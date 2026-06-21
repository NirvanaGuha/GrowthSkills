---
name: feature-adoption-campaign-planner
description: >
  Takes a low-adoption feature and a user segment and produces a complete multi-touch
  campaign plan spanning in-app tooltips, email sequences, push notifications, and
  contextual upgrade prompts — each touchpoint timed to the user's actual behavior, not
  a calendar. Built on the Hook Model (Nir Eyal) and the AARRR activation loop, it
  diagnoses WHY adoption is low first (awareness gap, friction, wrong-segment, or weak
  habit loop) before prescribing channel mix and copy. Calls brand-brain for voice and
  proof, then composes in-app-microcopy-writer-auditor, push-notification-copy-generator,
  subject-line-preview-text-optimizer, and cta-variant-generator rather than re-writing
  those skills. Saves a campaign brief and channel matrix to ./feature-adoption/[slug].md.
  Trigger phrases: "feature adoption," "nobody uses," "low adoption," "drive usage,"
  "get users to try," "in-app campaign," "activation campaign," "feature launch to
  existing users," "feature rollout campaign," or any request for a multi-touch plan
  targeting an underused capability.
---

# Feature Adoption Campaign Planner

A feature with no adoption is a cost center masquerading as a product update. This skill diagnoses the adoption gap first, then builds the right multi-touch campaign — in-app nudges, email, push, and upgrade prompts — timed to user behavior rather than calendar days, so the message lands when the user is in the workflow where the feature matters.

Framework: **Hook Model** (trigger → action → variable reward → investment) layered onto the **AARRR activation loop** to distinguish awareness gaps from friction gaps from habit-loop failures. The diagnosis determines the campaign architecture; the campaign architecture drives which sibling skills are composed.

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads voice, banned words, ICP, proof, positioning, and offer for the active brand. Does not reimplement brand resolution or scanning.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand voice adjectives, banned words, the feature's target ICP, and any real proof points (metric or social proof) before proceeding.
- **`aha-moment-activation-metric-definer`** — defines the measurable aha moment and activation event for the feature if not already known.
- **`in-app-microcopy-writer-auditor`** — writes and audits tooltip, empty-state, coach-mark, and modal copy; do not rewrite in-app copy inline.
- **`push-notification-copy-generator`** — writes push notification copy for behavior-triggered nudges; do not rewrite push copy inline.
- **`subject-line-preview-text-optimizer`** — optimizes email subject lines and preview text for each email in the sequence; do not rewrite subjects inline.
- **`cta-variant-generator`** — generates CTA variants for upgrade prompts and email CTAs; do not rewrite CTAs inline.
- **`usage-triggered-message-sequencer`** — if usage-trigger logic is complex (multiple events, branching), delegate the trigger map to this skill.
- **`lifecycle-email-push-copy-reviewer`** — reviews the full sequence for brand voice, flow, CTA strength, and character-limit compliance before finalizing.

---

## How a run works

```
Step 0  Load brand              ──► call brand-brain (always first)
Step 1  Diagnose the gap        ──► 4-quadrant diagnosis + Hook Model audit
Step 2  Define the aha moment   ──► compose aha-moment-activation-metric-definer if undefined
Step 3  Map the campaign         ──► channel matrix + trigger events + sequencing
Step 4  Compose copy skills     ──► in-app, push, email subjects, CTAs via siblings
Step 5  Review                  ──► lifecycle-email-push-copy-reviewer pass
Step 6  Save artifact            ──► ./feature-adoption/[slug].md
```

---

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. It returns the active brand's digest — voice adjectives, banned words, offer mechanics, real proof, ICP, and positioning. Do not write a single word of campaign copy until it returns.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand voice adjectives, banned words, the feature's target ICP, and any real proof points (metric or social proof) before proceeding.

---

### Step 1 — Diagnose the adoption gap (gate before campaign)

Map the feature to one or more of these four failure modes before prescribing a campaign. A wrong diagnosis wastes a sprint.

| Failure mode | Symptom | Primary lever |
|---|---|---|
| **Awareness gap** | Users with the need have never seen the feature | Triggered in-app discovery (tooltip, spotlight, empty-state) |
| **Friction gap** | Users start but don't complete setup | Onboarding modal, coach-mark, reduce steps |
| **Habit gap** | Users try once, then stop | Variable-reward loop, scheduled push nudge at natural trigger moment |
| **Segment mismatch** | Feature is shown to users who don't need it | Audience tightening — remove noise before adding messages |

Ask the user (or infer from data they share): what is the % who have ever used the feature vs. the % who use it regularly? Knowing which bucket (never tried vs. tried once) changes the campaign architecture entirely.

---

### Step 2 — Define the aha moment

If the feature's activation event (the specific in-product action correlated with retention) is not known, call **`aha-moment-activation-metric-definer`** before building the campaign. The trigger events and success metrics for every touchpoint anchor to this event.

---

### Step 3 — Build the campaign architecture

#### 3a. Channel matrix

For each failure mode diagnosed, assign the right channel. Channels are not equal weight — in-app is always the highest-leverage channel for adoption because it intercepts the user in the live workflow.

| Channel | Best for | Timing |
|---|---|---|
| **In-app tooltip / coach-mark** | Awareness + friction | Contextual: user visits the parent workflow |
| **In-app spotlight / modal** | Major feature launch to existing users | Session start or post-task success |
| **Email (behavior-triggered)** | Habit gap, winback of users who tried + stopped | 24–72h after trigger event or inactivity window |
| **Push notification** | Real-time nudge in active-use windows | At natural trigger moment (not time-of-day guess) |
| **Upgrade prompt** | Paywalled feature adoption | Contextual limit-hit or value-preview placement |

Avoid calendar-based email blasts for feature adoption. Trigger on behavior, not day.

#### 3b. Trigger event map

For each campaign touchpoint, specify:
- **Entry trigger:** the product event that fires the message (e.g., user completes Task X but has never used Feature Y)
- **Exit trigger:** the event that suppresses it (e.g., user has adopted the feature — defined by the aha moment)
- **Suppression:** never fire the same nudge to a user who has already completed the adoption event

#### 3c. Sequence cadence

A 5-touchpoint adoption sequence (adapt to fit the failure mode):

```
T+0  In-app: tooltip or spotlight at the moment of natural trigger
T+1d Email #1: "Here's what you're missing" — awareness + social proof
T+3d Push #1: behavior-triggered nudge (fires only on live workflow moment)
T+7d Email #2: friction-buster — "Here's how to set it up in 2 steps" + tutorial link
T+14d Email #3: value-reinforcement (success story / metric proof) OR sunset (stop messaging adopters)
```

If the user supplies a different window (e.g., a 30-day trial), adjust proportionally.

---

### Step 4 — Compose copy via sibling skills

Do not write in-app copy, push copy, or email subjects inline. Delegate:

- **Tooltip / coach-mark / modal copy** → call `in-app-microcopy-writer-auditor`, passing the feature name, the aha moment, voice from brand-brain, and the failure mode (awareness/friction/habit).
- **Push notification copy** → call `push-notification-copy-generator`, passing the trigger event, ICP, and voice.
- **Email subject lines + preview text** → call `subject-line-preview-text-optimizer` per email in the sequence.
- **CTAs (email buttons, upgrade prompts)** → call `cta-variant-generator`, passing placement, awareness stage, and offer.
- **If trigger logic is complex** (branching, multiple events, A/B branches at the trigger level) → call `usage-triggered-message-sequencer` to map the logic before copy is written.

---

### Step 5 — Review

After all copy is assembled, call **`lifecycle-email-push-copy-reviewer`** with the full sequence. It reviews brand voice, flow, CTA strength, urgency/clarity, and character-limit compliance. Address any flags before saving the artifact.

---

### Step 6 — Save the artifact

Save the completed campaign plan to `./feature-adoption/[feature-slug]-campaign.md` relative to the user's CWD. The file contains:
- Diagnosis table (failure mode + evidence)
- Aha moment definition + activation metric
- Channel matrix with trigger events + suppression conditions
- Sequence cadence with touchpoint specs
- Links/references to the copy assets produced by sibling skills
- Success metrics and a 30-day measurement plan

---

## The Hook Model applied to adoption campaigns

Nir Eyal's four phases map directly to campaign elements:

| Hook phase | What it means for adoption | Campaign element |
|---|---|---|
| **Trigger** | External cue (message) OR internal cue (existing habit) fires the behavior | In-app tooltip at natural workflow moment; email at inactivity window |
| **Action** | The simplest behavior in anticipation of reward (minimize steps) | Reduce setup friction; deep-link directly into the feature, not the dashboard |
| **Variable reward** | Outcome is satisfying but not entirely predictable | Show dynamic proof: "You've saved X hrs" / "Ranked in top Y%" |
| **Investment** | User puts data/time in, raising future value | Progress indicator; saved configuration that gets better over time |

A campaign that only addresses Trigger (sends a message) but ignores Action (friction) and Reward (value confirmation) will plateau at first-use, not sustained adoption.

---

## Principles (Non-Negotiable)

- **Diagnose before prescribing.** Never recommend a channel mix without first naming the failure mode. Segment mismatch is not fixed by more messages.
- **Behavioral triggers over calendar dates.** A message that fires when the user is in the relevant workflow converts; one that fires Tuesday at 10 AM adds noise.
- **Aha moment is the anchor.** Every touchpoint should reduce the friction between the user's current state and the defined aha-moment event. If the aha moment is undefined, define it first.
- **Exit triggers are not optional.** Every campaign must suppress itself when adoption succeeds. Messaging an already-adopted user erodes trust.
- **Compose, don't duplicate.** In-app copy, push copy, and subject lines belong to the sibling skills. This skill plans and orchestrates; it does not rewrite what siblings own.
- **Brand-brain first.** No copy, no channel recommendation, no proof claim before brand-brain returns the active brand's context.
- **Real proof or `[verify]`.** No invented metrics, no unconfirmed social proof. Mark anything unconfirmed `[verify]`.

---

## What Not to Do

- Don't run a calendar-based blast to your full user base to "announce" a feature to existing users. Reach the users who need it at the moment they need it.
- Don't write in-app copy, push copy, or email subjects inline — those belong to sibling skills.
- Don't skip the diagnosis gate. A "feature adoption campaign" for a segment-mismatch problem is wasted copy.
- Don't conflate feature announcement (marketing to prospects) with feature adoption (activation of existing users). The audience, voice, and funnel stage differ.
- Don't fire adoption nudges in the same sequence as onboarding. Separate trigger logic avoids double-messaging new users.
- Don't omit exit triggers. A user who has adopted the feature must exit the sequence.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or fallback executed) before any copy or channel recommendation?
- Failure mode diagnosed (awareness / friction / habit / segment mismatch) with supporting evidence?
- Aha moment defined and activation event named?
- Channel matrix includes entry trigger, exit trigger, and suppression condition for every touchpoint?
- In-app copy delegated to `in-app-microcopy-writer-auditor`? Push copy to `push-notification-copy-generator`? Subjects to `subject-line-preview-text-optimizer`? CTAs to `cta-variant-generator`?
- `lifecycle-email-push-copy-reviewer` review pass completed and flags addressed?
- Artifact saved to `./feature-adoption/[feature-slug]-campaign.md`?
- No invented proof; every unconfirmed metric marked `[verify]`?
- Success metrics and 30-day measurement plan included?
