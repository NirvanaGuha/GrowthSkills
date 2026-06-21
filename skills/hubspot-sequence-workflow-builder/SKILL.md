---
name: hubspot-sequence-workflow-builder
description: >
  Takes a sequence brief, automation goal, or raw campaign-idea list and returns HubSpot-ready
  sequence steps, workflow enrollment triggers, and a lead-prioritization scoring model — everything
  needed to deploy a complete outbound or lifecycle motion inside HubSpot without starting from a
  blank canvas. Two modes: Sequence mode builds a timed email/task cadence for 1:1 sales rep sends
  (HubSpot Sequences tool), including subject lines, body copy, send-day offsets, task reminders, and
  unenrollment logic. Workflow mode builds an automated multi-branch workflow with enrollment
  criteria, goal events, delays, If/Then branches, and action nodes (internal notifications, property
  updates, deal stage moves, list membership). Scoring mode outputs a Contact property-based lead
  scoring rubric (behavioral + demographic) mapped to lifecycle stages and routing rules. The skill
  does NOT provision HubSpot or write API calls; it produces the configuration spec a marketer or
  RevOps operator pastes directly into the platform. Calls brand-brain for voice and offer context,
  data-qa-measurement-gotcha-checker for enrollment-criteria hygiene, and cold-outreach-sequence-architect
  for strategic sequencing logic on complex multi-persona plays. Use when the user says "build me a
  HubSpot sequence," "set up a workflow," "enrollment trigger," "lead scoring in HubSpot," "automate
  my outreach in HubSpot," "HubSpot cadence," "workflow branch," or hands over a campaign brief and
  asks for HubSpot execution.
---

# HubSpot Sequence & Workflow Builder

Give it a goal, get a deployment-ready HubSpot configuration. Not a strategic overview — a literal step-by-step spec your team pastes into HubSpot Sequences or Workflows and activates.

This skill knows the difference between Sequences (rep-sent, 1:1, enrollment pauses on reply) and Workflows (automated, high-volume, enrollment continues unless explicitly stopped). It will never conflate them, and it will flag when you're using the wrong tool for the job.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads active brand voice, banned words, ICP, offer mechanics, and real proof. Every email and task script must be on-voice. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand voice adjectives, ICP job title/pain, primary offer, and any banned phrases before proceeding.
- **`data-qa-measurement-gotcha-checker`** — gate enrollment criteria before finalizing any workflow; catches unmapped properties, list-membership pitfalls, and re-enrollment loops that silently break automations.
- **`cold-outreach-sequence-architect`** — called on complex multi-persona or multi-product outbound plays; provides the strategic sequencing logic (persona mapping, channel mix, angle progression) that this skill then converts to HubSpot-specific configuration.
- **`lead-scoring-routing-model-designer`** — invoked in Scoring mode to produce the underlying scoring rubric; this skill translates that rubric into HubSpot Contact property configs and lifecycle-stage routing rules.
- *(optional)* `lifecycle-email-push-copy-reviewer` — when copy review is requested, passes completed sequence copy for a brand-voice and CTA audit before handing off.

---

## How a run works

```
Step 0  Load the brand            ──► brand-brain (always first)
Step 1  Identify the mode         ──► Sequence | Workflow | Scoring | Hybrid
Step 2  Clarify the brief         ──► confirm inputs; run data-qa gate on triggers/criteria
Step 3  Build the spec            ──► mode-specific output (see below)
Step 4  Self-review               ──► quality checklist; flag gaps
Step 5  Save artifact             ──► ./hubspot/[mode]-[slug].md
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's digest: voice adjectives, banned words, offer mechanics, destination URLs, real proof, ICP, and awareness tendency. Do not write a single subject line or email body before this returns.

Obey voice and banned words as hard overrides. Use only real proof from the brand brain; mark anything unconfirmed `[verify]`. Anchor message-match to the brand's live offer and ICP pain.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand voice adjectives, ICP job title/pain, primary offer, and any banned phrases before proceeding.

### Step 1 — Identify the mode

| Mode | Trigger | HubSpot tool |
|---|---|---|
| **Sequence** | outbound cadence, 1:1 rep sends, SDR cadence, sales follow-up | HubSpot Sequences |
| **Workflow** | automated nurture, lifecycle triggers, deal-stage moves, notifications | HubSpot Workflows |
| **Scoring** | lead scoring rubric, PQL scoring, lifecycle-stage thresholds | HubSpot Scoring Properties |
| **Hybrid** | campaign brief covering all three | combine modes in order |

If the user's input mixes Sequences and Workflows terminology without distinguishing, clarify in one line before proceeding. Conflating the two is the most common HubSpot configuration error.

### Step 2 — Clarify the brief

Confirm these inputs (batch the questions, ask only what's missing):
- Target persona and lifecycle stage at enrollment
- Primary goal / conversion event (meeting booked, demo, trial, closed-won)
- Sequence length or workflow depth preference
- Existing contact properties and lists available (critical for enrollment criteria)
- Re-enrollment rules (default: off for Sequences, configurable for Workflows)

Then run the enrollment criteria through `data-qa-measurement-gotcha-checker` before Step 3.

---

## Sequence mode — the Prospect Ladder framework

HubSpot Sequences are rep-activated, not automated. They pause on reply or meeting book. Build them around the **Prospect Ladder**: each touch moves the prospect one rung closer to a committed next step — never two rungs at once.

### Output spec

```
## Sequence: [name]
Persona: [job title / segment]
Goal event: [meeting booked / trial started / demo attended]
Unenrollment triggers: reply received | meeting booked | hard bounce | manual unenroll
Re-enrollment: [yes/no + condition]

### Steps
| # | Day offset | Type | Subject / Task | Body / Notes |
|---|---|---|---|---|
| 1 | Day 0 | Email | [subject line] | [full body, ~100–150 words] |
| 2 | Day 2 | Task | LinkedIn check / call | [talk-track note, 2–3 lines] |
| 3 | Day 5 | Email | [subject line] | [body] |
...

### Unenrollment logic
- On reply: pause immediately, assign to rep inbox
- On meeting booked: unenroll + trigger deal-creation workflow
- After step N with no response: move to nurture list [specify]

### Copy notes
[Voice compliance flags, banned-word check, message-match note]
```

**Ladder rungs:** curiosity (Step 1) → relevance/pain (Step 2–3) → social proof (Step 4) → breakup/low-friction ask (final). Never open with a demo ask on Step 1. Never use the same angle twice in a row.

**Subject line rules for HubSpot Sequences:** keep under 50 chars (mobile preview); avoid spam-trigger words; personalization tokens (`{{contact.firstname}}`, `{{contact.company}}`) are supported — use at least one per sequence.

**Task steps** are not optional filler. Every call task needs a 2–3 line talk track; every LinkedIn task needs a specific connection or engagement action. Vague tasks ("follow up") are not permitted.

---

## Workflow mode — the State-Machine spec

HubSpot Workflows are deterministic automations. Think of them as state machines: a contact enters a state, conditions branch it, actions mutate properties or trigger sends, exit conditions end the run. Build the spec as a state machine diagram in prose + table form.

### Output spec

```
## Workflow: [name]
Type: Contact-based | Deal-based | Company-based
Enrollment trigger: [property filter / list membership / form submit / event]
Re-enrollment: [on/off + rule]
Goal (exit condition): [property = value | event fired | deal stage reached]
Time zone / schedule: [send-window constraint if any]

### Branch map
[plain-English state-machine description: "Contact enters → If [condition A] → Branch A1 … else Branch A2 …"]

### Action nodes (tabular)
| Node | Type | Condition | Action | Delay before |
|---|---|---|---|---|
| 1 | Entry | — | Set lifecycle stage = MQL | — |
| 2 | If/Then | Lead score ≥ 50 | → Hot branch | — |
| 3 | Hot branch | — | Notify owner (internal email) | 0 min |
| 4 | Hot branch | — | Send email: [name] | 30 min |
| 5 | Cold branch | — | Add to nurture list | — |
...

### Property mutations
[List every Contact/Deal property this workflow reads or writes]

### Exit / suppression logic
[Goal event + suppression lists + manual unenrollment conditions]

### Data-quality gate result
[Output from data-qa-measurement-gotcha-checker: enrollment-criteria risks, re-enrollment loop flags, property-mapping issues]
```

**If/Then branch rules:** HubSpot evaluates If/Then branches at the moment the contact hits that node, not at enrollment. Flag any branch that relies on a property that may not be populated at that point — these silently route everyone to the "false" path.

**Delay discipline:** always specify a delay before the first action node (minimum 5 minutes) to allow property syncs to complete. Immediate-action workflows that fire on form submit routinely race the CRM sync.

---

## Scoring mode — the Threshold Stack

HubSpot scoring properties use additive positive/negative point rules against Contact or Company properties and behavioral events. The output of this mode is a deployable scoring rubric, not a strategy doc.

Invoke `lead-scoring-routing-model-designer` for the underlying scoring logic, then translate its output into this HubSpot-specific format.

### Output spec

```
## Lead Scoring Rubric: [brand / product]

### Demographic / Firmographic rules (HubSpot Score property)
| Attribute | Property | Operator | Value | Points |
|---|---|---|---|---|
| Target title | Job Title | contains | "VP", "Director", "Head of" | +15 |
| Target industry | Industry | is | [list] | +10 |
| Disqualifying title | Job Title | contains | "Student", "Intern" | −20 |
...

### Behavioral rules
| Action | Event | Points | Decay |
|---|---|---|---|
| Pricing page view | Page view: /pricing | +10 | none |
| Demo request | Form submit: Demo | +30 | none |
| Email unsubscribe | Email opt-out | −50 | none |
...

### Lifecycle-stage thresholds
| Score range | Lifecycle stage | Routing action |
|---|---|---|
| 0–24 | Subscriber | nurture workflow |
| 25–49 | Lead | MQL-watch workflow |
| 50–74 | MQL | notify SDR |
| 75+ | SQL | create deal + assign owner |

### Implementation notes
[HubSpot path: Settings → Properties → Score → create/edit; Workflow triggers reference "HubSpot Score ≥ N"]
```

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No subject line or email body before the brand context loads. Voice and banned words are hard overrides.
- **Sequences ≠ Workflows.** Never conflate. Flag the distinction if the user does. Using a Workflow where a Sequence is needed (or vice versa) is the root cause of most HubSpot automation failures.
- **Enrollment criteria must pass the data-quality gate.** A workflow that silently never enrolls anyone — because a property is unmapped or a list is empty — is worse than no workflow.
- **If/Then branches need populated properties.** Flag any branch condition that may evaluate on an empty field at the moment of evaluation.
- **Every email needs a single, specific next step.** Not "let me know if you're interested." A date, a link, a question. One.
- **Task steps are not filler.** Every task node needs a concrete action and a talk track or it will be skipped.
- **Real proof only.** Any stat or differentiator in sequence copy comes from brand-brain's proof set. Unconfirmed figures get `[verify]`.
- **Save the artifact.** Always offer to save to `./hubspot/[mode]-[slug].md` after a completed run.

---

## What Not to Do

- Don't build a Workflow when the user needs a Sequence (or vice versa) — ask.
- Don't write email copy before brand-brain returns. Don't reimplement brand scanning here.
- Don't skip the data-quality gate on enrollment criteria. Empty-list and unmapped-property bugs are invisible until you look for them.
- Don't generate scoring rubrics without calling `lead-scoring-routing-model-designer` for the logic layer — don't invent point values from scratch.
- Don't write task nodes that say "follow up" with no specifics.
- Don't use the same persuasion angle in back-to-back sequence steps.
- Don't promise HubSpot API calls, provisioning, or direct platform writes — this skill produces configuration specs, not integrations.

---

## Quality Checklist (self-review before presenting)

**All modes:**
- [ ] `brand-brain` called and active brand loaded before any copy written?
- [ ] Voice adjectives honored; banned words absent; proof sourced from brand.md?
- [ ] Mode identified correctly (Sequence vs. Workflow vs. Scoring) and stated at the top?
- [ ] `data-qa-measurement-gotcha-checker` run on enrollment criteria?

**Sequence mode:**
- [ ] Unenrollment triggers specified (reply, meeting, bounce)?
- [ ] No two consecutive steps use the same persuasion angle?
- [ ] Every task node has a concrete action + talk track?
- [ ] Subject lines under 50 chars; at least one personalization token per sequence?
- [ ] Ladder progression: curiosity → relevance → proof → breakup?

**Workflow mode:**
- [ ] Branch map written as a state machine (readable without seeing the platform)?
- [ ] Every If/Then node's condition flags any property that may be unpopulated at evaluation time?
- [ ] All property mutations listed?
- [ ] Delay before first action node specified (minimum 5 minutes)?
- [ ] Exit/goal condition and suppression lists defined?

**Scoring mode:**
- [ ] `lead-scoring-routing-model-designer` called for underlying logic?
- [ ] Demographic, firmographic, and behavioral rules all present?
- [ ] Lifecycle-stage thresholds mapped to routing actions?
- [ ] Implementation path noted (HubSpot Settings → Properties → Score)?

**All modes:**
- [ ] Artifact saved (or save offered) to `./hubspot/[mode]-[slug].md`?
