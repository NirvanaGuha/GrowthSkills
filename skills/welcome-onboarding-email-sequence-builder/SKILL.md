---
name: welcome-onboarding-email-sequence-builder
description: >
  Builds complete welcome and onboarding email sequences from scratch — subject lines, preview text,
  and full body copy for every email in the series. Takes a product overview and activation goals,
  applies the SaaS Onboarding Ladder framework (Welcome → Activation → Feature Depth → Social Proof
  → Conversion Gate), and outputs a publish-ready sequence that moves a new subscriber or trial user
  from signup to first meaningful outcome. Calls brand-brain for voice and ICP, subject-line-preview-text-optimizer
  for every subject line, cta-variant-generator for primary CTAs, and proof-vault for social proof
  inserts. Two modes: Quick (3-email core) for simple products and blogs; Full Sequence (5-7 emails)
  for SaaS trials, freemium products, and course enrollments. Saves sequences to ./sequences/.
  Use when the user says "write my welcome emails," "build an onboarding sequence," "new user drip,"
  "welcome email series," "onboarding emails for my product," "what should I send after signup,"
  "first-week email drip," or hands over a product and asks what to send new users or subscribers.
---

# Welcome & Onboarding Email Sequence Builder

Turn a product brief and an activation goal into a complete, on-voice, publish-ready email sequence. Every email earns its place — this skill does not pad sequences to hit a number. It ships the minimum cohesive arc that moves a new user from signup to first meaningful outcome, then stops.

---

## Skills this calls

- **`brand-brain`** (required, always first) — active brand's voice, banned words, ICP, offer/pricing + destination URLs, real proof, positioning. Do not write a single word before it returns.
- **`subject-line-preview-text-optimizer`** — called per-email to produce subject line + preview text pairs; do not hand-write subject lines here.
- **`cta-variant-generator`** — called for each email's primary CTA button; do not hand-write CTAs here.
- **`proof-vault`** *(optional)* — pulls real customer quotes, case study stats, or G2-style proof for the social proof email; synthesize inline if unavailable.
- **`de-slop-humanize-pass`** *(optional)* — run the drafted sequence through it if the copy feels formulaic after the first pass.

---

## How a run works

```
Step 0  Load the brand       ──► call brand-brain; wait for digest
Step 1  Gather inputs        ──► product, activation goal, mode, ESP/trigger info
Step 2  Map the arc          ──► SaaS Onboarding Ladder → assign emails + timing
Step 3  Draft each email     ──► for each: call subject-line skill, draft body, call CTA skill
Step 4  Self-review          ──► Quality Checklist pass before presenting
Step 5  Present + persist    ──► offer to save to ./sequences/<slug>-onboarding.md
```

### Step 0 — Load the brand (non-negotiable first step)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns the active brand's digest — voice, banned words, offer mechanics + destination URLs, real proof, ICP + awareness tendency, positioning — and the path to `brand.md`. If no brand exists, `brand-brain` bootstraps one first. **Do not draft any email copy until it returns.**

Obey voice and banned-words as hard overrides. Use only confirmed proof points (mark anything else `[verify]`). Anchor every CTA destination to the brand's real URLs from the digest.

Fallback if `brand-brain` is absent: read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if none, ask the user for the brand name, ICP, 3 voice adjectives, banned words, and the primary product URL. Always prefer the call.

### Step 1 — Gather inputs

Collect before drafting. Ask in a single block if not provided:

| Input | Why it matters |
|---|---|
| Product name + one-line description | Names the offer; anchors every email |
| The activation milestone | The aha moment — what the user must do to "get it" (e.g., first campaign sent, first widget installed). Sequence is reverse-engineered from this. |
| Sequence goal | Trial conversion / freemium upgrade / subscriber nurture / course completion |
| Trigger event | What fires Email 1 (signup, trial start, download, purchase) |
| ESP / platform | Klaviyo, ActiveCampaign, Braze, etc. — affects token syntax and delay format |
| Segments or personas | If the brand has multiple (from brand-brain's ICP); else one universal sequence |
| Mode | Quick (3 emails) or Full Sequence (5-7 emails); default Full if goal is trial conversion |

If activation milestone is unclear, ask one question: "What is the single action that means a new user has experienced the product's core value?" Do not proceed without an answer — it is the sequence's spine.

---

## The SaaS Onboarding Ladder (the framework)

Every sequence follows this arc, trimmed to the chosen mode. Each rung has a single job; combining jobs into one email dilutes both.

```
Rung 1 — Welcome & Promise         Send: T+0 (immediate)
Rung 2 — Activation Nudge          Send: T+1 day
Rung 3 — Feature Depth             Send: T+3 days
Rung 4 — Social Proof Bridge       Send: T+5 days        [Full Sequence only]
Rung 5 — Objection Handler         Send: T+7 days        [Full Sequence only]
Rung 6 — Conversion Gate           Send: T+10 days       [Full Sequence only, if trial]
Rung 7 — Final Nudge / Sunset      Send: T+13 days       [Full Sequence only, if trial]
```

**Quick mode uses Rungs 1, 2, 6.** Best for newsletters, lead magnets, and simple subscriber nurture where a conversion gate is not the goal — swap Rung 6 for a soft "here's what's next" close.

### Per-rung brief (what to write at each rung)

**Rung 1 — Welcome & Promise**
Job: Confirm the decision, deliver the promised value (download/login link/access), set expectations for what's coming. No feature dump.
Structure: warm open → confirm what they signed up for → deliver the goods (button/link) → one sentence previewing Rung 2.
Tone: excited but earned. If the brand's ICP is skeptical/technical, skip the exclamation marks and lead with specifics.

**Rung 2 — Activation Nudge**
Job: Drive the single activation milestone. One clear ask. If the user has already hit the milestone (use a conditional branch if the ESP supports it), swap to a "what's next" path.
Structure: name the milestone → explain why it matters (in ICP language, not feature-speak) → remove one common friction point → CTA to the exact page/step.
Conditional: `IF activated → branch to Rung 3 early; IF not → send this`.

**Rung 3 — Feature Depth**
Job: Surface one underused feature that gives power users disproportionate value. Not a feature tour — one thing, done well.
Structure: open with the outcome the feature creates → show the quickest path to it (2-3 steps max) → link to docs or a demo. Keep short.

**Rung 4 — Social Proof Bridge** *(Full only)*
Job: Reduce doubt at the mid-sequence drop-off point. One real customer story or data point — not a testimonials wall.
Structure: transition that acknowledges the user hasn't yet committed → one specific proof point (from `proof-vault`; mark `[verify]` if pulled inline) → restate the core value prop in 1 sentence → soft CTA.
Do not invent quotes. If no real proof is available, use an outcome stat (mark `[verify]`) or skip to Rung 5.

**Rung 5 — Objection Handler** *(Full only)*
Job: Name the most common objection to conversion at this stage and defuse it with fact or reframe.
Source objections from: brand-brain's `objections.md` companion file if present; else ask the user for the top 1-2 objections heard from trial users or in churn interviews.
Structure: name the objection honestly (builds trust) → answer it specifically → connect back to the activation milestone they've (hopefully) hit by now.

**Rung 6 — Conversion Gate**
Job: Make the ask. Trial ending / upgrade window / enrollment close. Hard deadline or value-based urgency (not fake countdown copy).
Structure: state what changes (trial ends, price goes up, cohort closes) → remind them of the value they've experienced → CTA to upgrade/enroll. Keep short — this email's job is the button, not the essay.
Urgency rule: only use deadline copy if the deadline is real. If there is no hard deadline, use value-based urgency ("everything you've built in the trial comes with you") instead.

**Rung 7 — Final Nudge / Sunset** *(Full, trial-mode only)*
Job: One last send, explicitly low-pressure. Give them an easy out if it's not right; make a genuine case if it is.
Structure: acknowledge the sequence is ending → one-sentence restatement of core value → CTA (upgrade or start fresh) → optional: "if this isn't for you, no hard feelings — here's how to [unsubscribe/downgrade/pause]."
Do not guilt-trip. Do not send if Rung 6 converted.

---

## Per-email format

Each email in the output uses this structure:

```
### Email [N] — [Rung Name]
Delay: T+[X] days
Trigger: [trigger event or conditional branch condition]
Audience: [universal / segment name]

**Subject line + preview text:** [call subject-line-preview-text-optimizer; show the recommended pair + 1 alt]

**Body:**
[Full copy — from salutation through sign-off. No placeholders except brand tokens ({{first_name}}, etc.) in the ESP's syntax. Inline links in [anchor text](URL) format. Approximate word count.]

**Primary CTA:** [call cta-variant-generator; show recommended button label + 1 alt]
**CTA destination:** [URL from brand-brain digest]

**Conditional branch (if applicable):**
- If [event]: → [action or email path]
- Else: → continue sequence
```

---

## Sequence timing defaults (adjust to ESP + brand context)

| ESP type | T+0 | T+1 | T+3 | T+5 | T+7 | T+10 | T+13 |
|---|---|---|---|---|---|---|---|
| Immediate trigger | Immediate | 1 day | 3 days | 5 days | 7 days | 10 days | 13 days |
| Batch send | Same day | Next morning | Day 3 | Day 5 | Day 7 | Day 10 | Day 13 |

If the brand's trial or free tier is shorter than 14 days (e.g., 7-day trial), compress proportionally: Rung 6 at T+5, Rung 7 at T+6. State the adjusted timing explicitly.

---

## Principles

- **Brand-brain first.** No copy before the brand digest returns. Voice + banned words override every instinct.
- **One job per email.** An email that tries to activate AND upsell AND share proof fails at all three.
- **Reverse-engineer from the aha moment.** Every email exists to move the user toward or past the activation milestone — not to show off features.
- **Honest urgency only.** Fake countdowns, invented scarcity, and vague "limited time" claims are banned. If there is no real deadline, write value-urgency copy instead.
- **Real proof or `[verify]`.** Do not invent customer quotes, case study stats, or G2 ratings. Pull from `proof-vault`; if unavailable, mark clearly.
- **Delegate, don't reimplement.** Subject lines → `subject-line-preview-text-optimizer`. CTAs → `cta-variant-generator`. Proof → `proof-vault`. This skill's job is the sequence arc and body copy.
- **Minimum viable sequence.** A tight 3-email Quick sequence beats a bloated 7-email sequence with filler. Earn each email or cut it.

---

## What not to do

- Don't hand-write subject lines or CTAs — call the sibling skills.
- Don't invent customer proof or stats — pull from proof-vault or mark `[verify]`.
- Don't write a feature-tour dump disguised as a welcome email.
- Don't use fake urgency (countdown timers, "only X spots left" when untrue).
- Don't send a Rung 7 to a user who already converted via Rung 6; note the conditional.
- Don't pad to 7 emails if the product and goal only need 3; Quick mode is a first-class option.
- Don't violate brand banned words even if they "work" for email copy in general.
- Don't use emojis in subject lines or body copy unless the brand explicitly allows them.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called and digest loaded before any copy was drafted?
- `subject-line-preview-text-optimizer` called for each email; recommended pair shown + 1 alt?
- `cta-variant-generator` called for each email's primary CTA; recommended label shown + 1 alt?
- Every email has exactly one job; no email is doing two?
- Activation milestone named and every email maps back to it?
- Urgency copy is real (deadline exists) or replaced with value-urgency language?
- All proof points sourced from proof-vault or marked `[verify]`; no invented quotes or stats?
- Timing delays stated; compressed if trial is shorter than 14 days?
- Conditional branch logic noted for Rung 2 and Rung 7?
- Voice and banned-words honored throughout; no exceptions?
- Output offered for save to `./sequences/<slug>-onboarding.md`?
