---
name: win-back-sweep-orchestrator
description: >
  Flagship orchestrator: turns a lapsed/churned base into a complete, segmented win-back SWEEP —
  not one campaign, but a different win-back series per behavioral segment, each with its own
  churn-reason-matched angles, escalating reactivation-offer logic, a cancel-moment save sequence,
  and a send-time + cadence schedule — compiled into one bundled deliverable a marketer can run cold.
  It does NOT do the work itself: it chains four real sibling skills end to end —
  `segmentation-rfm-strategy-builder` (defines the lapsed tiers), `win-back-re-engagement-campaign-builder`
  (builds a sequence per tier), `churn-save-sequence-writer` (the cancel-moment save), and
  `send-time-cadence-recommender` (when + how often to fire each sequence) — and loads brand context
  once via `brand-brain` so every segment is on-voice, real-offer, real-proof. Use when the user says
  "build my whole win-back program," "win-back sweep," "re-engage my lapsed base end to end," "segment
  my churned users and write the win-back," "reactivation program," "I have a churned list — do the
  whole thing," or hands over an orders/ESP export plus churn-reason tags and wants the complete
  retention recovery system, not a single email. One run = a junior marketer's week of work.
---

# Win-Back Sweep Orchestrator

A growth team in a box for lapsed-base recovery. You point it at a churned/lapsed list (orders export, ESP schema, or even a plain description) plus whatever churn-reason tags you have, and it returns the **whole program**: the segments, a tailored win-back series for each one, the cancel-moment save flow, and the send schedule that fires them — all in the brand's real voice, anchored to the brand's real offers and real proof.

This skill is a **conductor, not a player**. It does not write subject lines, score RFM tiers, or invent offers itself. It sequences four specialist sibling skills, passes each the previous stage's output, gates on quality, loops on weak output, and compiles everything into one bundled deliverable saved to your project folder. Every claim it can't confirm is marked `[verify]` — it never invents customers, numbers, offers, or proof.

---

## Skills this calls

The pipeline, in order. Each is a real installed skill; invoke each via the **Skill tool** and pass it the handoff named below. **Never re-implement a stage's work here** — if a stage is somehow unavailable, say so and stop rather than faking its output.

| # | Skill | Its job in the sweep | Receives | Hands off |
|---|---|---|---|---|
| 0 | **`brand-brain`** (required) | Load brand context once for every downstream stage | the request + any named brand | voice, banned words, offer mechanics + destinations, real proof, ICP, positioning, slug |
| 1 | **`segmentation-rfm-strategy-builder`** | Define the lapsed tiers from the data | brand digest + the user's orders/ESP/events input | RFM table + named segments (At-Risk, Can't-Lose, Hibernating, Lost…) with sizes, cut-points, per-tier action playbook |
| 2 | **`win-back-re-engagement-campaign-builder`** (run **per lapsed segment**) | Build a full escalating win-back sequence for one tier | one segment definition + that tier's churn-reason tags + brand digest | a per-tier campaign pack: RFB-ladder sequence table, churn-reason routing, sunset branch |
| 3 | **`churn-save-sequence-writer`** | The cancel-moment save flow (the upstream catch, before they become "lapsed") | the active cancellation reason codes + retention offers + brand digest | a 2–3 step save sequence + escalation routing, saved to `./retention/churn-save/` |
| 4 | **`send-time-cadence-recommender`** | When to fire each sequence and how often, without burning the list | the segment list + any open/click history + channel mix | per-segment send-window + cadence schedule, gap rules, sunset trigger |

Optional, when installed and relevant: `lifecycle-email-push-copy-reviewer` (the cross-segment review gate — see Stage 5), `email-compliance-auditor-gdpr-can-spam` (audit every sunset branch), `proof-vault` (real proof for mid-sequence beats), `offer-pricing-brain` (confirm reactivation offers exist before any sequence promises one).

---

## How a run works

```
Stage 0  Load brand        ──► brand-brain (once; every stage reads it)
Stage 1  Segment the base  ──► segmentation-rfm-strategy-builder → named lapsed tiers
              │  GATE: which tiers are worth a sequence? (skip tiny/Lost-only → sunset)
              ▼
Stage 2  Per-tier sequences ──► win-back-re-engagement-campaign-builder  (loop, ~parallel)
              │  for EACH selected tier, passing that tier's churn-reason tags
              ▼
Stage 3  Save flow          ──► churn-save-sequence-writer  (the cancel-moment catch)
              ▼
Stage 4  Timing             ──► send-time-cadence-recommender  (windows + cadence per tier)
              ▼
Stage 5  Review gate        ──► lifecycle-email-push-copy-reviewer  (Revise → loop to Stage 2)
              ▼
Stage 6  Compile + human OK  ──► one bundled deliverable in ./win-back-sweep/  → approve to ship
```

### Stage 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. It returns the digest — voice adjectives, banned words, offer mechanics + destination URLs, real proof, ICP + awareness tendency, positioning, and the brand slug — plus the path to `brand.md`. **Hold all downstream stages until it returns.** This context flows into every sibling call, so the whole sweep speaks in one voice and quotes one set of real offers/proof.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; else ask the user for the brand's voice adjectives + banned words, primary reactivation offer(s), ICP, and the main churn reasons before proceeding. This is a thin read, not a reimplementation — never write `brand.md` yourself. Prefer the call.

### Stage 1 — Segment the lapsed base

Invoke **`segmentation-rfm-strategy-builder`**, passing the brand digest + the user's data (orders/events CSV, ESP schema, or description). It returns the RFM score table and the named tiers with sizes and a per-tier action playbook. **Do not score RFM or invent segment names here** — that is its job.

**GATE — pick the tiers worth a sweep.** From the returned playbook, the win-back-relevant tiers are typically **At-Risk, Can't-Lose, Need-Attention, Hibernating**, and **Lost** (sunset-only). Apply:
- Skip any tier under a viable audience floor (default ~200 contacts; flag it instead of building a sequence no one will receive).
- Route **Lost** straight to a single sunset/re-permission send, not a full multi-touch series — it's a deliverability decision, not a revenue one.
- If the data audit (Stage 1's own flags) shows the inputs are unreliable (open-recency masquerading as purchase-recency, zero-order contaminated quintiles), **stop and surface it** before building sequences on a broken model.

### Stage 2 — Build a win-back sequence per selected tier (the loop)

For **each** selected tier, invoke **`win-back-re-engagement-campaign-builder`**, passing: that tier's definition (RFM profile, size, lapse window) + the churn-reason tags that apply to it + the brand digest. It returns that tier's campaign pack — the RFB-ladder sequence, churn-reason routing, and sunset branch.

- These per-tier runs are **independent → batch them** (issue the calls together; each is self-contained). They do not depend on each other, only on Stage 1.
- **Differentiate, don't duplicate.** Can't-Lose (former Champions) gets a high-personalization, low-discount series; Hibernating gets a lighter, single-incentive series; At-Risk gets the full escalating ladder. If two tiers come back near-identical, the segmentation was too coarse — note it and merge, don't ship twins.
- Keep the per-tier churn-reason routing intact: the same tier can carry different reason tags, and the angle must match the reason, not just the tier.

### Stage 3 — The cancel-moment save flow

Invoke **`churn-save-sequence-writer`** with the active cancellation reason codes + the brand's retention offers + the digest. This is the **upstream catch** — the messages that fire at the cancel/downgrade moment, *before* a user ever lands in a lapsed tier. It saves its own artifact to `./retention/churn-save/`; reference it in the bundle so the sweep covers the full arc (catch-at-cancel → win-back-when-lapsed → sunset-if-gone). If the brand has no cancel-moment trigger wired (e.g. a pure newsletter), say so and skip this stage rather than inventing a cancel flow.

### Stage 4 — Send timing and cadence

Invoke **`send-time-cadence-recommender`** with the full tier list from Stage 1 + any open/click history the user provided + the channel mix. It returns per-segment send windows, cadence, gap rules, and a sunset trigger. Fold its windows into each tier's sequence table from Stage 2 so the deliverable says not just *what* each touch is but *when* it fires and *how often*. With no history, it will return COR-default estimates flagged `[estimated — validate after 4 sends]`; carry that flag through, do not launder it into certainty.

### Stage 5 — Cross-segment review gate (the loop-back)

Run the assembled per-tier sequences through **`lifecycle-email-push-copy-reviewer`** in one pass (voice consistency, CTA strength, urgency calibration, character limits, banned words). This is the **separate review lane** — the orchestrator authors via siblings, then a different skill approves; never self-approve the copy in the same pass.

- If the reviewer returns **Approve** → proceed to Stage 6.
- If it returns **Revise** on a tier → **loop back to Stage 2 for that tier only** with the reviewer's notes appended to the brief, re-run, re-review. Cap at 2 revise loops per tier; if it still fails, ship it flagged `[needs human edit: <reason>]` rather than spinning forever.
- Optionally route every sunset branch through `email-compliance-auditor-gdpr-can-spam` before compiling.

### Stage 6 — Compile the bundle and hand to the human

Compile everything into the bundled deliverable (below) and present a one-screen summary: tiers covered, sequences built, total touches, offers used, open `[verify]` items. **The human approves before anything ships** — this skill produces the runnable program; it does not push to an ESP or send. End by asking which tier they want to load into their platform first, and whether to run `bulk-scheduling-csv-builder` on the schedule.

---

## Bundled deliverable

Save to the **project-relative** `./win-back-sweep/` (the user's CWD — never the skill folder). One folder, run-cold-ready:

```
./win-back-sweep/
  00-sweep-overview.md          # the map: tiers, sizes, which got sequences, offers, [verify] list, run order
  01-segments-rfm.md            # from Stage 1 (the RFM table + named tiers + playbook)
  02-sequence-at-risk.md        # one per selected tier (Stage 2) — sequence table + routing + sunset
  02-sequence-cant-lose.md
  02-sequence-hibernating.md
  03-churn-save-flow.md         # the cancel-moment save (Stage 3 / mirrors ./retention/churn-save/)
  04-send-schedule.md           # windows + cadence per tier (Stage 4)
  05-review-notes.md            # reviewer verdict per tier + any [needs human edit] flags (Stage 5)
```

`00-sweep-overview.md` is the cover sheet a buyer reads first: the segment map, the per-tier run order, the consolidated offer ladder, the cadence at a glance, and every unresolved `[verify]`. If saving isn't possible, output `00` inline and note the paths the rest would occupy. Confirm the save in one line.

---

## Orchestration principles (Non-Negotiable)

- **Conduct, never replay.** Every stage's work belongs to its sibling skill. This orchestrator routes inputs, gates outputs, loops on failure, and compiles — it does not re-derive RFM, re-write subject lines, or re-invent offers.
- **Brand-brain once, read everywhere.** Load context a single time at Stage 0 and pass it down; never let a downstream skill re-bootstrap a different brand mid-sweep.
- **A sweep is plural by definition.** The deliverable is a *different* sequence per behavioral tier with reason-matched angles — not one campaign relabeled. If tiers come back identical, the segmentation was too coarse; merge and say why.
- **Author and approve in separate lanes.** Stage 2 authors; Stage 5 (`lifecycle-email-push-copy-reviewer`) approves. Never self-approve copy in the authoring pass.
- **Gate before you build.** Don't generate a sequence for a tier under the audience floor, for a broken-data model, or for Lost (sunset it). Surface the gate; don't quietly build dead deliverables.
- **Loop, but bounded.** Revise → re-run the failing tier with the notes attached; cap at 2 loops, then flag for human edit. Never spin indefinitely.
- **Honest urgency, real offers, real proof.** No fake deadlines, no invented coupons, no fabricated stats. Confirm offers via `brand-brain` / `offer-pricing-brain`; mark everything unconfirmed `[verify]`.
- **The human ships.** Output a runnable program and stop. This skill never sends, schedules into a live ESP, or pushes — it hands the bundle over for approval.

## What not to do

- Don't produce any sequence before `brand-brain` returns the active brand.
- Don't re-implement segmentation, sequence-writing, save-copy, or timing logic inline — call the four siblings; if one is unavailable, stop and say so.
- Don't build a single "all lapsed users" campaign — that defeats the sweep; segment first, then build per tier.
- Don't send a discount on the first touch of any tier, or collapse the escalating ladder into a coupon blast (that's the sequence builder's rule — preserve it, don't override it).
- Don't ship near-identical sequences across tiers; don't exceed channel ceilings or cadence fatigue thresholds from Stage 4.
- Don't self-approve the copy — the review gate is a separate skill and a separate pass.
- Don't launder a `[estimated]` send window or an unconfirmed offer into a stated fact.
- Don't write any artifact into the skill folder or `brand.md` — the bundle goes to `./win-back-sweep/`.
- Don't auto-send or push to a platform — assemble, summarize, and wait for the human.

## Quality checklist (self-review before presenting)

- `brand-brain` loaded once at Stage 0; the same brand/voice/offers carried into every sibling call?
- `segmentation-rfm-strategy-builder` produced named tiers with sizes; the tier-selection gate applied (floor, Lost→sunset, broken-data halt)?
- `win-back-re-engagement-campaign-builder` run **per selected tier** (not once globally), with that tier's churn-reason tags routed in?
- Per-tier sequences are genuinely differentiated (Can't-Lose ≠ Hibernating ≠ At-Risk), not relabeled twins?
- `churn-save-sequence-writer` run for the cancel-moment catch (or explicitly skipped with reason)?
- `send-time-cadence-recommender` run; windows + cadence folded into each tier's sequence; `[estimated]` flags preserved?
- `lifecycle-email-push-copy-reviewer` ran as a separate approval pass; Revise verdicts looped back (≤2) or flagged `[needs human edit]`?
- Every offer confirmed real (or `[verify]`); no fake urgency; no invented proof; sunset branches compliant?
- Bundle saved to `./win-back-sweep/` with `00-sweep-overview.md` as the cover sheet; nothing written to the skill folder?
- Final summary presented and the human asked to approve before anything ships?
