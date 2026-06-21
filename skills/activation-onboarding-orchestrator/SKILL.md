---
name: activation-onboarding-orchestrator
description: >
  Flagship orchestrator — a growth team in a box for activation. Product + new-user persona +
  activation milestone → a complete, ship-ready activation program: a documented aha-moment +
  activation metric, a mapped new-user lifecycle journey, a 7-email welcome/onboarding drip,
  usage-triggered behavioral message sequences, and in-app microcopy (empty states, tooltips,
  banners) — all on one brand's voice, QA'd, and saved as a bundled deliverable a buyer can run
  cold. It does NOT re-implement any stage: it CHAINS the existing sibling skills end-to-end
  (aha-moment-activation-metric-definer → lifecycle-journey-mapper →
  welcome-onboarding-email-sequence-builder → usage-triggered-message-sequencer →
  in-app-microcopy-writer-auditor), gates on weak output, and routes a QA loop before handing back.
  Brand context comes from the `brand-brain` skill, never invented here. Use when the user says
  "build my onboarding," "design our activation program," "new-user onboarding flow,"
  "onboarding emails + in-app copy," "get new users to the aha moment," "fix activation,"
  "onboarding orchestrator," or hands over a product + persona and asks for the whole activation
  motion in one run. It orchestrates and assembles — it does not replace the specialist skills'
  craft, and it does not touch retention/win-back/billing flows beyond the activation window.
---

# Activation / Onboarding Orchestrator

Point it at a product and a new-user persona; get back a complete activation program — the metric you're driving toward, the journey new users actually take, the email drip that walks them there, the behavioral nudges that fire on real usage, and the in-app copy at each step. One run, one bundled folder, one brand voice throughout. This is the skill a buyer points at and says "this replaces a junior marketer's week."

It is an **orchestrator**, not a writer. Every stage of real work is done by a specialist sibling skill that already exists. This skill's job is to load the brand once, run the stages in the right order, hand each stage's output to the next, gate on weak output, route the QA loop, and assemble the pieces into one coherent deliverable. It writes nothing a specialist owns.

**Scope guardrail:** this covers the *activation window* — first touch through the aha moment and early habit. It does not write churn-save, win-back, upsell, or billing flows (those are sibling skills outside this run). If the user asks for those, finish activation and point them to the right skill.

---

## Skills this calls

In pipeline order (each is invoked via the Skill tool — never re-implemented here):

1. **`brand-brain`** (required, Layer-0) — loads the active brand's voice, banned words, ICP, offer mechanics + destination URLs, and real proof. Run first; everything downstream inherits it.
2. **`aha-moment-activation-metric-definer`** — defines the aha moment, the activation metric formula, and a dashboard spec. The north star the whole program optimizes for.
3. **`lifecycle-journey-mapper`** — maps the new-user journey from signup to aha to early habit: stages, user goals, friction, and the message moments that need copy.
4. **`welcome-onboarding-email-sequence-builder`** — writes the time-based welcome/onboarding email drip (the spec's 7-email backbone) that walks users to the activation metric.
5. **`usage-triggered-message-sequencer`** — designs behavior-triggered sequences (event conditions, delays, channel routing) that fire on what the user does or fails to do.
6. **`in-app-microcopy-writer-auditor`** — writes and audits empty-state, tooltip, and banner copy for each in-product moment the journey surfaced.

Supporting calls (conditional — invoked only when a gate trips):

- **`content-qa-reviewer`** — QA's the assembled email + in-app copy against brand and clarity. Returns Approve / Revise; a Revise routes back to the owning stage.
- **`funnel-drop-off-analyzer`** and **`growth-diagnostic-deep-dive`** — called by the metric-definer when usage data is available and the aha moment must be found in data, not guessed.
- **`voice-of-customer-mining-pipeline`** — called when the persona is thin and the journey needs real user language for empty-state and email copy.
- **`cta-variant-generator`** / **`push-notification-copy-generator`** — the email and usage-trigger builders already call these for buttons and push lines; do not call them directly from here.

---

## How a run works

```
Stage 0  Load the brand        ──► call brand-brain (bootstraps on first use)
Stage 1  Define activation     ──► aha-moment-activation-metric-definer        → METRIC
Stage 2  Map the journey       ──► lifecycle-journey-mapper                    → JOURNEY
Stage 3  Write the email drip  ──► welcome-onboarding-email-sequence-builder   → EMAILS
Stage 4  Wire usage triggers   ──► usage-triggered-message-sequencer           → TRIGGERS
Stage 5  Write in-app copy      ──► in-app-microcopy-writer-auditor             → MICROCOPY
Stage 6  QA gate + assemble    ──► content-qa-reviewer loop, compile, save     → PROGRAM
```

### Stage 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. It returns the active brand's digest — voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning, ICP + awareness tendency — and the path to `brand.md`. If the brand is new, `brand-brain` bootstraps it first. **Produce nothing downstream until it returns.** Every later stage receives this digest so the program speaks in one voice and uses only real claims.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user to install `brand-brain` (preferred) or answer a 4-question mini-setup (what it is · ICP + awareness · offer mechanics + destination · 3 voice adjectives + banned words). Never write `brand.md` yourself — that is `brand-brain`'s job alone. Always prefer the call.

### Stage 1 — Define the activation metric (`aha-moment-activation-metric-definer`)

Pass the product description, the new-user persona, and any usage/cohort data the user supplied, plus the brand digest. The skill returns the **aha-moment definition, activation metric formula, and dashboard spec**. If usage data is present it will lean on `funnel-drop-off-analyzer` / `growth-diagnostic-deep-dive` to find the signal in data rather than guess.

> **Handoff →** the *activation metric* and *aha-moment behavior* become the explicit goal every later stage points at. The email drip's job is "drive this metric"; the usage triggers fire around this behavior; the in-app copy removes friction on the path to it.

### Stage 2 — Map the new-user journey (`lifecycle-journey-mapper`)

Pass the brand digest, the persona, and Stage 1's aha moment + metric. The skill returns the **stages from signup → aha → early habit**, each with the user's goal, the friction, and the message moments that need copy. If the persona is thin, it pulls real user language via `voice-of-customer-mining-pipeline`.

> **Handoff →** the journey's *message moments* become the concrete worklist: which moments are emails (→ Stage 3), which are behavioral triggers (→ Stage 4), and which are in-product surfaces (→ Stage 5). The orchestrator routes each moment to its owning stage; nothing in the journey is left without an artifact.

### Stage 3 — Write the onboarding email drip (`welcome-onboarding-email-sequence-builder`)

Pass the brand digest, the journey's email moments, and the activation metric. The skill returns the **7-email welcome/onboarding sequence** — timing, subject lines + preview text, body copy, and CTAs — each email's job tied to moving the user one step closer to the metric.

> **Handoff →** the sequence's structure (which emails depend on a user *not yet* having activated) feeds Stage 4: any "if they haven't done X by day N" branch becomes a usage trigger, so time-based and behavior-based messaging don't double-send.

### Stage 4 — Wire the usage triggers (`usage-triggered-message-sequencer`)

Pass the brand digest, the journey's behavioral moments, the activation metric, and the email sequence (so triggers complement, not collide with, the drip). The skill returns **event-conditioned sequences** — trigger event, delay, channel routing, suppression rules against the time-based drip.

> **Handoff →** the set of in-product moments referenced by triggers (empty states, feature tooltips, nudge banners) is collected for Stage 5.

### Stage 5 — Write the in-app microcopy (`in-app-microcopy-writer-auditor`)

Pass the brand digest and the full list of in-product surfaces from Stages 2 and 4. The skill returns **empty-state, tooltip, and banner copy** plus its own clarity/length/vagueness audit for each surface.

> **Handoff →** all five artifacts (metric, journey, emails, triggers, microcopy) flow into the Stage 6 QA + assembly gate.

### Stage 6 — QA gate and assemble

1. **QA the customer-facing copy.** Invoke `content-qa-reviewer` on the assembled emails + in-app copy against the brand digest. It returns **Approve** or **Revise** with specifics.
2. **Loop on Revise.** Route each issue back to its *owning stage* (an off-voice email → re-run Stage 3 on that email; a vague tooltip → Stage 5), re-run only what failed, re-QA. Cap at **2 loops**; if it still fails, flag the unresolved items for the human rather than shipping them.
3. **Coherence pass.** Confirm one voice across all artifacts, no duplicate sends between drip and triggers, and that every journey message moment produced an artifact.
4. **Assemble + save** the bundled deliverable (below), then surface the **human approval point**: present the program summary and ask the user to approve before they wire it into their ESP / push platform / product.

---

## Orchestration logic (gates, branches, the human)

- **Hard gate — brand first.** No stage runs before `brand-brain` returns. Its voice + banned-words override every specialist's defaults.
- **Data branch (Stage 1).** Usage data present → the metric is found in data (funnel/diagnostic). No data → the metric-definer marks its definition `[verify]` and the program proceeds on a clearly-labeled hypothesis, not a fabricated number.
- **Thin-persona branch (Stage 2).** Persona too sparse to map friction → mine real VoC language before mapping; never invent user motivations.
- **Weak-output gate (any stage).** If a stage returns something thin or off-brief (e.g. fewer email moments than the journey demands, or a metric with no formula), do not paper over it downstream — re-run that stage with tighter inputs, or surface the gap to the human. Garbage in one stage poisons every stage after it.
- **QA loop (Stage 6).** `content-qa-reviewer` → **Revise** routes back to the owning stage, max 2 loops, then escalate to the human. The orchestrator never self-approves copy it assembled.
- **Human approval point.** One, at the end: the human reviews the assembled program before activation. The orchestrator also pauses for the human mid-run if `brand-brain` had to bootstrap or a gate can't be resolved automatically.

---

## Bundled deliverable

Save to a **project-relative** path (leading `./` = the user's CWD/project, **never** the skill folder):

```
./activation-program/[brand-slug]/
├── 00-activation-program-summary.md   # one-page: metric, journey at a glance, what ships where, approval status
├── 01-activation-metric.md            # aha moment, metric formula, dashboard spec  (Stage 1)
├── 02-lifecycle-journey-map.md        # stages, friction, message-moment → artifact routing table  (Stage 2)
├── 03-onboarding-email-drip.md        # 7 emails: timing, subject, preview, body, CTA  (Stage 3)
├── 04-usage-triggered-sequences.md    # trigger events, delays, channels, suppression rules  (Stage 4)
├── 05-in-app-microcopy.md             # empty states, tooltips, banners + audit notes  (Stage 5)
└── 06-qa-report.md                    # content-qa-reviewer verdicts, loop history, any human flags  (Stage 6)
```

If the user prefers, compile the same content into one `./activation-program/[brand-slug]-program.md`. Always confirm the save path before writing; never overwrite an existing program folder without asking.

---

## Principles (Non-Negotiable)

- **Brand-brain first, always.** No artifact before `brand-brain` returns; its voice + banned-words win every conflict.
- **Orchestrate, don't reimplement.** Every stage's craft belongs to its specialist skill. This skill chains, gates, routes, and assembles — it does not rewrite emails, redefine metrics, or author microcopy itself.
- **One metric, one motion.** Every stage points at the Stage 1 activation metric. Emails, triggers, and copy that don't move it don't belong in the program.
- **Handoffs are explicit.** Each stage receives the prior stage's actual output, not a vague summary. The journey's message moments are the contract that nothing gets dropped.
- **Truth discipline.** Real proof and real numbers only; everything unconfirmed is `[verify]`. Never invent activation rates, user quotes, or aha moments.
- **Don't double-send.** Time-based drip and usage triggers are reconciled with suppression rules, not left to collide in the user's inbox.
- **Separate authoring from approval.** The QA pass is `content-qa-reviewer`, not the orchestrator grading its own assembly. The human approves before launch.

## What Not to Do

- Don't produce any artifact before `brand-brain` returns the active brand.
- Don't re-implement any stage — invoke the sibling skill; if one is missing, say so and stop rather than faking its output.
- Don't write `brand.md` yourself, and don't re-derive voice/ICP/proof inside this skill.
- Don't fabricate the aha moment or activation rate when there's no data — label it `[verify]` and proceed on a stated hypothesis.
- Don't ship a stage's weak output downstream — re-run it or escalate.
- Don't self-approve the assembled copy; route the QA loop and stop at the human approval point.
- Don't drift past the activation window into churn-save, upsell, or billing flows — point the user to those siblings instead.
- Don't save the deliverable inside the skill folder; use the `./activation-program/` project path.

## Quality checklist (self-review before handing back)

- `brand-brain` called first and the active brand loaded (or bootstrapped) before any stage ran?
- All five pipeline stages invoked **as skills**, in order, each fed the prior stage's real output?
- Stage 1 produced a metric *formula* (or a clearly-labeled `[verify]` hypothesis when no data) — not a vague aspiration?
- Every journey message moment from Stage 2 maps to an artifact (email, trigger, or microcopy) — nothing dropped?
- Drip (Stage 3) and triggers (Stage 4) reconciled with suppression rules so no user double-sends?
- In-app copy (Stage 5) covers every in-product surface the journey + triggers surfaced, and passed its own audit?
- `content-qa-reviewer` run on the assembled copy; any Revise routed to the owning stage (≤2 loops) or flagged for the human?
- One brand voice across all artifacts; only real proof used (rest `[verify]`)?
- Bundled deliverable saved to the project-relative `./activation-program/[slug]/` path, with a summary and the human approval point surfaced?
