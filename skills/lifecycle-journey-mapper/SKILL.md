---
name: lifecycle-journey-mapper
description: >
  Maps a brand's customer lifecycle end-to-end — from first awareness through deep retention and
  advocacy — into an annotated stage-by-stage diagram with trigger points, recommended channels,
  and on-brand message themes at each stage. It does NOT write full sequence copy (hand off to
  sibling skills for that); it produces the strategic blueprint the rest of the lifecycle stack
  executes against. Calls `brand-brain` to load the active brand's ICP, offer, and voice before
  producing anything, and calls `icp-persona-builder` when personas are absent. Use whenever the
  user says "map our lifecycle," "what emails should we send when," "build our customer journey,"
  "lifecycle stages," "what triggers do we need," "journey map," "where are we losing people,"
  "retention strategy map," or hands over product milestones and asks how to message them.
---

# Lifecycle Journey Mapper

Turns ICP knowledge and product milestones into a complete, annotated lifecycle map — stages, triggers, channel mix, and message themes — so any growth or lifecycle marketer can see the whole retention picture and know exactly which sibling skill to call next to execute each stage.

This skill maps and recommends. It does not write sequence copy, build automation flows, or produce HTML assets. When the map is done, it points to the right skill for each execution job.

---

## Skills this calls

- **`brand-brain`** (required) — resolves and loads the active brand's voice, ICP, offer mechanics, proof, and positioning. Never begin mapping before it returns.
- **`icp-persona-builder`** (called when no personas exist in brand context) — returns named personas with JTBD, awareness tendency, and language patterns that shape trigger language and message themes.
- *(optional, for execution handoff)* `welcome-onboarding-email-sequence-builder`, `lead-nurture-drip-builder`, `push-notification-copy-generator`, `subject-line-preview-text-optimizer`, `cta-variant-generator`, `proof-vault`. Noted in the map's Execution column so the user knows which skill to call at each stage.

---

## How a run works

```
Step 0  Load the brand         ──► call brand-brain (bootstraps on first use)
Step 1  Check personas         ──► call icp-persona-builder if none exist in brand context
Step 2  Gather product milestones ──► confirm or elicit (see inputs)
Step 3  Build the stage map    ──► eight-stage lifecycle model (see below)
Step 4  Resolve trigger priority ──► run the trigger-priority table; assign one active stage per user
Step 5  Annotate each stage    ──► triggers · channel recs · message themes · execution pointer
Step 6  Flag gaps & priorities ──► which stages are unmapped, underserved, or high-risk
Step 7  Present & offer to save
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns the active brand's digest: voice, banned words, offer mechanics + destination URLs, real proof, positioning line, ICP + awareness tendency. Do not write a single stage label before it returns.

Use the ICP's awareness tendency (problem-aware, solution-aware, etc.) to calibrate stage entry points. Use the offer mechanics to name real milestone triggers. Banned words override any copy suggestions in the map.

**Fallback if `brand-brain` is absent or returns no brand:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user for product, ICP + primary pain, activation milestone, retention signal, and voice adjectives + banned words before mapping.

### Step 1 — Check personas

If the brand digest includes named personas with JTBD, proceed. If not, **call `icp-persona-builder`** to produce them before building the map. A lifecycle map built for "everyone" becomes noise; personas anchor trigger language and message theme choices to real jobs-to-be-done.

### Step 2 — Gather product milestones

The map's trigger layer is only as good as its milestone inputs. Confirm or elicit:

1. **The activation moment** — the single action that correlates with a user becoming retained (e.g., first push sent, first campaign live, first 100 subscribers reached). If unknown, flag it: defining this is the job of `aha-moment-and-activation-metric-definer`.
2. **The expansion signal** — behavior or threshold that predicts upgrade/upsell (feature usage, seat count, limit approach, etc.).
3. **The churn signal** — leading indicator of disengagement (login gap, feature-drop-off, downgrade intent, cancellation-flow entry).
4. **Any hard business milestones** — trial expiry date, billing anniversary, plan-limit thresholds, contract renewal windows.

If the user cannot provide these yet, build the map with `[define]` placeholders at trigger fields and note that the map is a hypothesis to validate.

---

## Our model: the trigger-driven lifecycle graph (a working model we use here)

This is our house model — not an externally established framework. It treats the customer lifecycle as a directed graph of **named stages** connected by **behavioral triggers**. Each node (stage) has a goal, a primary channel mix, a message theme family, and a risk flag. Each edge (trigger) is an observable event or condition — never time-alone.

It is graph-shaped on purpose: it inherits the customer-as-driver stance from **Customer-Led Growth** (Georgiana Laudi & Claire Suellentrop, *Forget the Funnel*, 2023) — map the journey from the customer's own struggle → evaluation → growth, not the brand's pipeline stages — but keeps eight operational stages instead of CLG's three, because lifecycle marketers need to message at a finer grain than a strategy framework prescribes. Where CLG asks *what job is the customer hiring us for*, this model asks *which observable event moves them between stages, and what do we send when it fires*.

**Why not time-based drips?** Time-based sequences send the same message whether the user is stuck or thriving. Trigger-based stages send the right message based on what the user actually did (or stopped doing). This model is the strategic layer that makes trigger selection intentional rather than arbitrary.

### The eight canonical lifecycle stages

| Stage | User state | Brand's goal | Risk if absent |
|---|---|---|---|
| **1. Acquisition** | Awareness → first contact | Capture intent, establish relevance | Wasted spend; wrong leads enter funnel |
| **2. Welcome / First Mile** | Signed up, no activation yet | Orient, reduce overwhelm, build confidence | High early churn before value is felt |
| **3. Activation** | Approaching or reaching aha moment | Get the user to the activation milestone | Silent churn; user leaves before seeing value |
| **4. Habit Formation** | Post-activation, building routine | Deepen usage, add second/third feature touchpoint | Shallow engagement; vulnerable to churn on first friction |
| **5. Expansion** | Power user or approaching plan limits | Drive upgrade, seat add, cross-sell | Revenue left on table; plan-limit frustration unaddressed |
| **6. Retention / Loyalty** | Long-term engaged customer | Reinforce value, deepen relationship | Complacency; easy target for competitive displacement |
| **7. At-Risk / Win-Back** | Declining engagement or churn signal | Re-engage before or after lapse | Preventable churn; no second-chance sequence |
| **8. Advocacy** | High-satisfaction customer | Convert satisfaction into referrals and proof | Zero referral flywheel; proof assets ungathered |

Map only the stages that exist for this product. A pure SaaS trial product may collapse stages 1–3. A B2B enterprise product may have a distinct stage between 6 and 7 for renewal. Name them honestly.

---

## Trigger-priority resolution (the decision mechanic)

A real user fires more than one trigger at a time. Someone who just hit the activation milestone can *also* be approaching a plan limit *and* have a 6-day login gap. If two edges could fire on the same user in the same window, the graph is ambiguous and the wrong sequence sends. This table resolves it: when triggers collide, the highest-priority one wins and decides which stage the user is in **right now**. Lower-priority triggers queue; they do not send concurrently.

Priority is ordered by **reversibility cost** — fire first on the trigger whose window closes soonest and whose miss is hardest to undo (a churn you didn't catch beats a referral you sent late).

| Priority | Trigger class | Example observable event | Resolves to stage | Why it wins |
|---|---|---|---|---|
| **P1 — Save** | Churn / cancellation signal | Cancellation-flow entry, downgrade click, hard usage cliff | 7. At-Risk / Win-Back | A live churn is irreversible if missed; it preempts everything, including expansion |
| **P2 — Convert** | Hard business deadline | Trial expiry < 48h, contract renewal window open, payment failure | Acquisition/Expansion gate (per deadline) | Time-boxed and revenue-bearing; the window will not reopen |
| **P3 — Activate** | Activation milestone reached/missed | First push sent, or stalled N days short of it | 3. Activation | The single highest-leverage moment for long-term retention; short-lived |
| **P4 — Expand** | Expansion / limit signal | Plan-limit approach, seat-add behavior, power-usage threshold | 5. Expansion | Revenue upside, but the limit and the intent both persist — it can wait behind a save |
| **P5 — Deepen** | Habit / engagement signal | Second-feature adoption, streak, return visit | 4. Habit Formation / 6. Retention | Ongoing state, not an event; lowest urgency, always yields to the above |
| **P6 — Advocate** | Satisfaction signal | High NPS, milestone celebrated, review left | 8. Advocacy | Valuable but fully deferrable; never preempts a save, convert, or activate |

**Tie-break rules within the table:**
- **Negative beats positive.** A churn or payment-failure signal (P1/P2) always overrides any positive signal firing in the same window — mirror of the lead-scoring sibling's disqualifier cap.
- **Sooner-closing window wins ties within a priority band.** Two P2 deadlines? The nearer expiry fires first.
- **One active stage per user at a time.** A user occupies exactly one stage; queued lower-priority triggers re-evaluate only after the active stage's exit trigger fires or its send completes.
- **A time-only trigger never outranks an event.** If the only thing that "fired" is a clock (e.g., "day 7 of onboarding"), it loses to any real behavioral event and is flagged `[time-only — validate]`.

Use this table to set each stage's **entry trigger** in the block below: when you name an entry trigger, confirm it isn't silently outranked by a higher-priority trigger the same user could fire. If it is, note the precedence in the Segment note so the executing sibling skill suppresses the lower-priority send.

---

## Building the annotated map

For each applicable stage, produce a block using this structure:

```
### Stage N — [Stage Name]
Goal: [one sentence — what success looks like for the brand at this stage]
Entry trigger: [the observable event that moves a user INTO this stage]
Priority band: [P1–P6 from the trigger-priority table — what this entry trigger yields to]
Exit trigger: [the event or condition that moves them OUT (to next stage OR to At-Risk)]
Segment note: [any persona-specific splits + any higher-priority trigger that suppresses this send]

Channel mix:
  Primary: [channel(s) — email / push / in-app / SMS / sales touch / community]
  Secondary: [channel(s)]
  Avoid: [channels wrong for this stage + why]

Message theme family:
  - [Theme 1: what the message is about + the emotional register]
  - [Theme 2]
  - [Theme 3]
  (Banned words from brand-brain apply to all theme copy suggestions.)

Proof to deploy: [specific proof type from brand-brain: stat / case study / review / testimonial — or [verify] if unconfirmed]

Execution: call [skill name] to build this stage's copy/sequence/flow
```

Keep each block concise. The map is a decision layer, not a copywriting layer.

---

## Gap analysis (always included)

After the full map, produce a **Gap & Priority Table**:

| Stage | Status | Gap / Risk | Priority |
|---|---|---|---|
| [Stage name] | Mapped / Unmapped / Hypothesis | [What's missing or uncertain] | High / Medium / Low |

Priority rules:
- **High** = gap exists in a stage where churn or revenue loss is already measurable or likely.
- **Medium** = gap exists but the user is in an earlier stage of building this out.
- **Low** = stage is low-risk given the product's retention model.

Flag any stage where a `[define]` placeholder sits in the trigger field — those are measurement gaps, not just copy gaps.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No stage label, no channel rec, no message theme before `brand-brain` returns. Voice and banned-words are hard overrides throughout.
- **Triggers over time.** Every stage transition must be driven by an observable event or condition, not a clock. Call out time-only triggers as a risk.
- **One active stage, resolved by priority.** When triggers collide, run the trigger-priority table: a Save (P1) preempts everything, negative beats positive, and no user sits in two stages at once. An unresolved collision is the lifecycle equivalent of a non-deterministic route — don't ship it.
- **Personas are the lens.** A lifecycle map without personas produces generic column headers. Segment where the journey genuinely differs; don't over-split for the sake of appearing thorough.
- **Honest about unknowns.** Mark unknown triggers `[define]`, unconfirmed proof `[verify]`, and hypothetical stages `[hypothesis]`. A map with honest gaps is more useful than a confident-looking fabrication.
- **Point to execution, don't duplicate it.** The map names the right sibling skill for each stage. It does not write full sequence copy inline — that produces a cluttered, uncheckable output.
- **The map must be actionable.** Every stage block ends with a clear execution pointer. If it doesn't, the map is a decoration.

## What Not to Do

- Don't write full sequence copy in the map — that's `welcome-onboarding-email-sequence-builder`, `lead-nurture-drip-builder`, etc.
- Don't use time-based triggers as the primary stage entry condition without flagging the risk.
- Don't leave colliding triggers unresolved. If a user can fire two entry triggers in the same window, run the priority table and record which one wins — an ambiguous graph sends the wrong sequence.
- Don't build the map before `brand-brain` returns the active brand context.
- Don't call `icp-persona-builder` if rich personas already exist in brand context — re-calling wastes a run.
- Don't invent product milestones or activation signals — ask, or mark `[define]`.
- Don't produce a map so generic it fits any SaaS product. Every stage block should reference the brand's real offer language, real proof (or `[verify]`), and real ICP.
- Don't skip the gap analysis — it is where the strategic value lives for a user who already has partial lifecycle coverage.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or bootstrapped) before any stage was written?
- Voice and banned-words honored in all theme copy suggestions; unconfirmed proof marked `[verify]`?
- Each stage has: goal, entry trigger (event-based), priority band, exit trigger, channel mix, message theme family, proof pointer, and execution skill pointer?
- Trigger-priority table run: any two triggers a single user could fire in the same window resolved to one active stage (P1 Save preempts; negative beats positive)?
- Personas present (called `icp-persona-builder` if absent); stage blocks reference ICP language, not generic buyer language?
- Product milestones confirmed or marked `[define]`; time-only triggers flagged as risk?
- Gap & Priority Table included; unmapped or hypothesis stages clearly labeled?
- Map saved to `./lifecycle/[slug]-journey-map.md` (or offered, if user requested persistence)?
