---
name: customer-onboarding-time-to-value-program-designer
description: >
  Designs a complete post-sale onboarding program anchored to a defined Time-to-Value (TTV) metric
  — from customer profile and product milestones to CSM touchpoint schedule, success milestone map,
  kickoff agenda, and a TTV tracking spec. Calls brand-brain for voice and positioning, composes
  onboarding-flow-builder for the in-product activation track, and welcome-onboarding-email-sequence-builder
  for the email drip — then layers on the human-touch layer (CSM cadence, kickoff, escalation triggers,
  QBR format) that neither sibling covers. Structured around DARE (Define → Activate → Reach
  value → Expand), our working four-phase model for onboarding. Produces a deployable onboarding playbook: kickoff agenda, milestone scorecard,
  CSM contact-schedule, and TTV tracking spec saved to ./onboarding/<slug>-program.md.
  Use when the user says "design our onboarding," "reduce time to value," "new customer program,"
  "CSM playbook," "post-sale journey," "onboarding milestones," "kickoff agenda," "TTV tracking,"
  or hands over a customer segment and asks how to get them to first value fast.
---

# Customer Onboarding & Time-to-Value Program Designer

Get a new customer to their first undeniable win as fast as the product allows — and build the
CSM motion, milestone map, and measurement system that make it repeatable. This skill designs the
full post-sale program around a single north-star: the moment the customer experiences the core
value that justified the purchase. Everything else — kickoff, drip emails, CSM touches, escalation
triggers, QBRs — exists to accelerate or protect that moment.

It composes siblings for the in-product and email tracks; its unique contribution is the
human-touch layer and the TTV tracking architecture.

---

## Skills this calls

- **`brand-brain`** (required) — loads voice, ICP, proof, and offer mechanics so the program
  language, success metric names, and milestone labels are on-brand. Fallback if brand-brain is
  absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md`
  directly; if none exists, ask the user for brand name, ICP (role, company type, key pain),
  product's primary value delivered, and voice adjectives before proceeding.
- **`onboarding-flow-builder`** — generates the in-product checklist, welcome modal, product tour
  script, and empty-state copy for the activation track; this skill layers the human cadence on top.
- **`welcome-onboarding-email-sequence-builder`** — writes the email drip (Day 0 → Day 30+);
  this skill sets the milestone logic and trigger conditions it should follow.
- **`aha-moment-activation-metric-definer`** — defines the activation event if the team hasn't
  yet nailed it; call before designing milestones when the aha-moment is ambiguous.
- **`lifecycle-journey-mapper`** — consult when the onboarding program needs to connect
  forward into expansion and renewal stages.
- **`lifecycle-email-push-copy-reviewer`** — reviews the email drip output for voice, CTA
  strength, and character-limit compliance before finalizing.
- **`cta-variant-generator`** — used to sharpen milestone CTAs and kickoff confirmation buttons.
- **`proof-vault`** — supplies real customer proof and outcome stats to use in milestone
  confirmation messages and success summaries.

---

## How a run works

```
Step 0  Load brand context      ──► call brand-brain; get voice, ICP, proof
Step 1  Define TTV              ──► anchor the whole program to one measurable moment
Step 2  Map milestones          ──► 3–5 milestone sequence from signed to TTV
Step 3  Design the CSM cadence  ──► touchpoint schedule, kickoff agenda, escalation logic
Step 4  Compose the email track ──► hand milestone triggers to welcome-onboarding-email-sequence-builder
Step 5  Build TTV tracking spec ──► events, dashboards, health scores
Step 6  Assemble the playbook   ──► save to ./onboarding/<slug>-program.md
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns voice, banned words,
ICP, offer mechanics, real proof, and the brand slug. Do not produce any program artifacts until
it returns.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
brand's `brand.md` directly; if none exists, ask the user for brand name, ICP description (role,
company type, primary pain), the product's core delivered value, and 3 voice adjectives before
proceeding.

### Step 1 — Define TTV (the program's north star)

TTV = the earliest moment a customer experiences the core value they bought. Before designing
anything else, nail this definition:

- **TTV event:** one measurable in-product action or outcome (e.g., "first campaign sent,"
  "first report shared," "first checkout recovered").
- **TTV window:** target elapsed days from contract close to TTV event (industry baseline for
  B2B SaaS: 7 days for self-serve, 14–30 for SMB-CSM, 30–90 for enterprise [verify for your
  category]).
- **TTV proxy metric:** leading indicator visible before the TTV event (e.g., integration
  connected, first audience imported, first template saved).

If the user hasn't defined these, use the DARE framework (Step 2) to surface them from the
product milestone map.

---

## DARE: our working four-phase model

Every program runs on four phases (DARE is a house mnemonic we use here, not an external standard). Name them using the brand's own language where possible.

| Phase | Job | Done when… |
|---|---|---|
| **D — Define** | Customer + CSM align on goals, success criteria, and timeline | Kickoff complete; mutual success plan signed |
| **A — Activate** | Customer completes the minimum viable setup to reach TTV | TTV proxy metric fires |
| **R — Reach Value** | Customer experiences the TTV event and sees a measurable outcome | TTV event fires; outcome data visible |
| **E — Expand** | Customer expands usage, seats, or spend based on proven ROI | First expansion signal detected |

This skill fully designs D, A, and R; it hands off E to `lifecycle-journey-mapper` and
`upsell-cross-sell-campaign-builder`.

---

## Step 2 — Milestone Map (3–5 milestones D → R)

Map the customer's literal action sequence from signed to TTV. Each milestone needs:

| Field | What to specify |
|---|---|
| **Name** | Short on-brand label (e.g., "Integration Live," "First Campaign Sent") |
| **Owner** | Customer / CSM / Shared |
| **Day target** | Elapsed days from close; build in slack before TTV window |
| **Success signal** | The one observable fact that proves this milestone is done |
| **Blocker risk** | The most common reason this stalls (flag as `[verify]` if unknown) |
| **Celebration trigger** | The in-app or email message that fires on completion |

Standard skeleton (adapt to product complexity):

```
M0  Kickoff Complete     Day 0   Customer + CSM    Agenda followed, MSP drafted
M1  Technical Setup      Day 3   Shared            Integration / pixel / key config live
M2  First Use            Day 7   Customer          TTV proxy event fires
M3  TTV Event            Day 14  Customer          Core value delivered; outcome visible
M4  Habit Formation      Day 30  Customer          TTV event repeated ≥3x; health score green
```

### Minimum Viable Onboarding (MVO) principle

Every task in the setup checklist must answer: "Does removing this delay TTV?" If not, it
belongs in an advanced track, not Day 0–14. Protect the critical path; defer nice-to-haves.

---

## Step 3 — CSM Cadence & Kickoff Agenda

### CSM touchpoint schedule

| Touch | Timing | Format | Goal |
|---|---|---|---|
| Pre-kickoff prep | T−2 days | Email/async | Send agenda; confirm technical contacts |
| Kickoff | T+0 | 45-min video call | Align on goals, timeline, MSP; set M1 owner |
| M1 check-in | M1 due date | 15-min video or async | Unblock setup; confirm M1 signed off |
| TTV check-in | M3 due date | 30-min video | Review TTV outcome data; celebrate; document ROI |
| 30-day review | Day 30 | 45-min video | Health score review; identify expansion signal |
| QBR | Day 90 | 60-min exec call | ROI case; renewal/expansion conversation |

Escalation triggers (fire CSM outreach immediately regardless of schedule):
- No login within 72h of kickoff
- M1 not complete by Day 5
- TTV proxy not fired by Day 10
- NPS score ≤ 6 at any point
- Support ticket volume spike (≥2 unresolved tickets open simultaneously)

### Kickoff agenda template (45 min)

```
0:00–0:05  Introductions + recording consent
0:05–0:15  Customer shares: goals, success criteria, team/owner map
0:15–0:25  CSM walks: milestone map + Day-14 TTV target
0:25–0:35  Technical setup: confirm owners, timeline, blockers
0:35–0:43  Co-create Mutual Success Plan (MSP) — live in shared doc
0:43–0:45  Next step: M1 owner, due date, check-in booked
```

Mutual Success Plan (MSP) must capture: primary goal → TTV definition → milestone owners +
dates → escalation contacts → 90-day expansion hypothesis. Save as a live doc in the customer's
shared workspace.

---

## Step 4 — Email Track (compose, don't rebuild)

Call `welcome-onboarding-email-sequence-builder` with:
- Milestone names and day targets from the map
- Trigger events (milestone completion signals)
- TTV event definition
- Brand voice from brand-brain
- Escalation-path copy needs (re-engagement branch for stalled milestones)

This skill defines WHEN and WHY; the sibling writes WHAT. Review the output with
`lifecycle-email-push-copy-reviewer` before finalizing.

---

## Step 5 — TTV Tracking Spec

Specify exactly what to measure, where, and at what threshold.

### Events to instrument

| Event | Trigger | Property |
|---|---|---|
| `onboarding_kickoff_complete` | MSP signed | `customer_id`, `csm_id`, `target_ttv_days` |
| `m1_complete` | M1 success signal | `days_elapsed`, `blocker_flag` |
| `ttv_proxy_fired` | TTV proxy action | `days_elapsed` |
| `ttv_event` | Core value delivered | `days_elapsed`, `outcome_value` |
| `onboarding_stalled` | No login ≥72h post-kickoff | `days_since_kickoff`, `last_milestone` |

### Dashboard metrics (weekly CSM review)

- **Median TTV** (days from close to TTV event) — broken by segment, CSM, and cohort month
- **TTV funnel** — % reaching each milestone on-time
- **Stall rate** — % of accounts triggering escalation at each milestone
- **Onboarding health score** — composite: milestones on-time (40%) + login frequency (30%) +
  support ticket volume (30%)
- **Day-30 retention by TTV cohort** — fastest-to-value vs. slowest-to-value retention delta
  (expected: faster TTV → higher Day-30 retention [verify with your data])

### Health score thresholds

| Score | Status | Action |
|---|---|---|
| 80–100 | Green | Routine cadence |
| 60–79 | Amber | CSM proactive outreach within 24h |
| < 60 | Red | CSM + management escalation; pause marketing |

---

## Step 6 — Playbook output

Assemble everything into `./onboarding/<brand-slug>-program.md` with sections:

1. TTV definition (event, window, proxy metric)
2. DARE phase map
3. Milestone map table
4. CSM touchpoint schedule + escalation triggers
5. Kickoff agenda template
6. Email track brief (inputs for `welcome-onboarding-email-sequence-builder`)
7. TTV tracking spec (events + dashboard metrics + health score)
8. Open questions / `[verify]` items for the team

---

## Principles (Non-Negotiable)

- **TTV first.** Every element of the program exists to accelerate or protect the TTV event.
  If a task doesn't serve that, defer it.
- **Minimum viable onboarding.** The critical path to TTV must be as short as the product allows.
  Nice-to-haves live in an advanced track.
- **Milestone owners are explicit.** Every milestone has one human owner (not "team"). Shared
  ownership without a named lead is no ownership.
- **Human cadence and product track are coordinated.** CSM touchpoints and email triggers fire
  off the same milestone signals — never on calendar alone.
- **Truth only.** All benchmark numbers (TTV windows, retention lift) are either real for this
  brand or marked `[verify]`. Never fabricate outcomes.
- **Brand-brain first.** No program language until brand-brain returns voice and ICP.

## What Not to Do

- Do not rebuild the in-product checklist or email sequence — call the sibling skills.
- Do not set a TTV window without confirming the product's actual fastest viable setup time.
- Do not make the kickoff agenda longer than 45 minutes; respecting the customer's time is
  part of the program experience.
- Do not include every possible metric — track only the five that tell a CSM what to do next.
- Do not treat onboarding as complete at TTV; hand off explicitly to `lifecycle-journey-mapper`
  for the expansion stage.
- Do not invent CSM ratios (e.g., "1 CSM per 50 customers") — cite or mark `[verify]`.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and returned voice, ICP, and proof before any artifact was produced?
- TTV event, proxy metric, and target window all defined and specific (not vague)?
- Each milestone has: name, owner, day target, success signal, blocker risk, celebration trigger?
- MVO principle applied — no non-critical tasks on the Day-0–14 critical path?
- CSM touchpoint schedule covers: pre-kickoff, kickoff, M1, TTV, Day-30, QBR?
- Escalation triggers are specific (observable events, not "if things feel slow")?
- Email track handed off to `welcome-onboarding-email-sequence-builder` with milestone triggers?
- TTV tracking spec includes event list, dashboard metrics, and health score thresholds?
- All benchmark numbers either sourced from the brand or marked `[verify]`?
- Playbook saved to `./onboarding/<brand-slug>-program.md`?
