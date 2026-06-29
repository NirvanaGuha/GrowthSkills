---
name: meeting-prep-follow-up-pack
description: >
  Turns a prospect's LinkedIn profile, account dossier, or raw meeting notes into two polished
  sales-enablement assets: a pre-call brief (research digest, stake-holder context, agenda,
  opening questions grounded in SPIN/MEDDIC-style discovery, and anticipated objections) and a
  personalized post-meeting follow-up email (recap, confirmed next steps, attributed action items,
  and curated resources) routed by call outcome. Chains account-dossier-builder for firmographic
  depth, objection-library-builder for reframe handles, and cta-variant-generator for the follow-up
  email's closing CTA tier. Saves both artifacts to
  ./outreach/<account-slug>/. Use when the user says "prep me for this call," "write a pre-call
  brief," "draft the follow-up," "help me research before a sales meeting," "send the recap email,"
  "meeting prep," "pre-call research," or hands over a LinkedIn URL, meeting notes, or a transcript
  and asks for either asset.
---

# Meeting Prep & Follow-Up Pack

Two assets, one run: a brief that makes you sound like you've been tracking this account for months, and a follow-up that closes the loop before the prospect's coffee goes cold. Every fact is sourced. Every CTA is on-voice. Nothing is invented.

This skill produces and saves. It does not manage brand context, build the account profile from scratch, or derive objection reframes — those live in the specialist skills it chains.

---

## Skills this calls

- **`brand-brain`** (required, always first) — resolves the active brand's voice, banned words, ICP, offer/pricing, proof, positioning.
- **`account-dossier-builder`** (required for Prep mode) — builds or retrieves the structured account profile (business model, tech stack, recent news, pain hypotheses) from the prospect's company URL. Do not re-derive this here.
- **`objection-library-builder`** (required for Prep mode) — returns the brand's indexed objection reframes to populate the brief's objection-prep section.
- **`cta-variant-generator`** (required for Follow-Up mode) — writes the follow-up email's closing CTA in the brand's voice, anchored to the confirmed next step.
- *(optional)* `proof-vault` — if available, pull case-study proof matched to the prospect's industry/pain for the follow-up resources section. Synthesize inline from the brand's `proof.md` if absent.

---

## How a run works

```
Step 0  Load the brand       ──► brand-brain (voice, ICP, offer, proof, banned words)
Step 1  Pick the mode        ──► Prep (pre-call brief) | Follow-Up | Both
Step 2  Chain specialist skills as needed
Step 3  Draft the asset(s)
Step 4  Self-review; save to ./outreach/<account-slug>/
```

### Step 0 — Brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns the active brand's digest — voice, banned words, offer mechanics, destination URLs, real proof, ICP + awareness tendency. Do not produce any copy until it returns.

**Fallback if brand-brain is absent or returns no brand:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for the prospect's LinkedIn/name + company URL (Prep) and the meeting notes + confirmed next steps (Follow-Up).

### Step 1 — Pick the mode

| Mode | Trigger | Input needed |
|---|---|---|
| **Prep** (default) | "prep me," "pre-call brief," "research this prospect" | Prospect LinkedIn URL or name + company URL |
| **Follow-Up** | "write the follow-up," "send the recap," meeting notes / transcript in | Meeting notes or transcript, confirmed next steps |
| **Both** | "full pack," "prep + follow-up," or both inputs present | Both of the above |

When input is ambiguous, default to Prep and offer Follow-Up at the end.

---

## Prep mode — the pre-call brief

### BEACON — the brief's section checklist (house model)

BEACON is this skill's checklist for what a complete brief contains — not an external sales methodology. It's a labeling device for six sections; the mechanics that make two of them non-generic (Angles and Conversation) live in the two lookup tables below. The discovery questions in **C** are written in the **SPIN** (Rackham, *SPIN Selling*) / **MEDDIC** tradition — situation/problem/implication probes that surface metrics, pain, and the economic buyer — not freeform curiosity.

**B — Background.** Company essentials sourced from `account-dossier-builder` (size, model, stack, funding, recent news), carrying its FACT/INFER triage through. Never fabricate; mark gaps `[verify]`.

**E — Entry point.** The prospect's role, reported-to chain, likely mandate, and why they took the meeting. Infer from title + company stage; flag inferences explicitly.

**A — Angles (computed, not improvised).** Don't free-associate pain. Read the prospect's role and the company's stage off the dossier, then look up the two angles to lead with in the **Angle Selector** below. Each chosen angle still needs its own evidence line (job-post language, tech-stack signal, news) and the matching product angle from `brand-brain`.

**C — Conversation openers (signal-driven).** 3–5 SPIN/MEDDIC-style probes, each built from a real signal via the **Signal → Probe Engine** below — never generic discovery ("What are your goals?"). Every question carries the signal that generated it.

**O — Objections.** Pull top 3 relevant objections from `objection-library-builder`; include one-sentence reframe handle per objection.

**N — Next step.** Pre-set the ideal outcome of this meeting: what does "winning" look like, and which CTA tier (see the Follow-Up matrix) should you ask for at the end?

#### Angle Selector — prospect role × company stage → lead with these 2 pain angles

Read role from the dossier's org/people layer and stage from its revenue/news layer, then lead with the two angles in the cell. These are *priors* — override any when a dossier signal contradicts, and note the override.

| Role \ Stage | Early (seed–Series A, under ~50) | Growth (Series B–C, scaling GTM) | Mature (Series D+ / public / PE) |
|---|---|---|---|
| **Economic buyer** (VP/C-level, owns budget) | Speed-to-revenue · low ops overhead | Efficient growth (CAC/payback) · forecastability | Margin/consolidation · risk + compliance |
| **Champion / functional lead** (Dir/Head, owns the metric) | Quick win they can show up · doing more with a thin team | Hitting the number · proving the channel scales | Defending budget · standardizing across teams |
| **End user / IC** (operator who'd use it) | Time saved on manual work · fewer tools to stitch | Less firefighting · cleaner handoffs across the funnel | Reliability at scale · escaping legacy-tool drag |

If role and stage are both `[verify]`, lead with the brand's single strongest ICP pain and say so — don't guess a cell.

#### Signal → Probe Engine — turn a research signal into an opening question

For each high-signal fact from the dossier, pick the matching probe type and instantiate it. This is the reusable engine behind the Conversation step; the dossier's FACT signals are the only valid inputs.

| Signal in the dossier | Probe type (SPIN/MEDDIC) | Question shape to instantiate |
|---|---|---|
| Hired N reps / opened N roles in a function | Scale-pain (implication) | "You've added N [role] this quarter — what starts breaking in [their process] at that headcount?" |
| New VP/C-level in the past ~90 days | Mandate (economic buyer) | "With [name] coming in to run [function], what's the mandate you're being measured against this year?" |
| Funding round / new budget signal | Initiative + metric | "Post-[round], where's the pressure to show return — and on what number?" |
| Tech-stack tool detected (competitor or adjacent) | Status-quo / displacement | "You're on [tool] for [job] — what's it not doing that put this meeting on the calendar?" |
| Migration / replatform / launch in the news | Implication + timing | "With [launch/migration] underway, what happens to [pain] if it's not solved before then?" |
| Public goal / earnings / exec post | Decision criteria | "You've said publicly [goal] — what has to be true for a tool to count as moving that?" |
| No strong signal (thin dossier) | Generic-but-true | One honest situational question; do **not** fabricate a signal to dress it up. |

```
## Pre-Call Brief — [Prospect Name], [Company] — [Date]
### Background (via account-dossier-builder; FACT/INFER preserved)
### Entry Point
### Pain Angles  (role × stage → Angle Selector; overrides noted)
### Opening Questions  (each: signal → probe type → question)
### Objection Prep (via objection-library-builder)
### Target Next Step  (winning outcome + CTA tier to request)
```

Save to `./outreach/<account-slug>/prep-brief-[date].md`.

---

## Follow-Up mode — the post-meeting email

### RECAP → CONFIRM → EQUIP → NEXT (house email structure)

The four sections are constant; **what goes in EQUIP and NEXT branches by how the call actually went.** Before drafting, read the notes and classify the call outcome, then resolve EQUIP (proof to attach) and NEXT (which CTA tier to request from `cta-variant-generator`) off the matrix below. The CTA tiers map directly to `cta-variant-generator`'s commitment ceiling — you are picking the tier here and letting that skill write the line.

**RECAP.** 3–5 bullet recap. Use the prospect's language (their actual quotes in the notes), not internal framing. One sentence of value acknowledgment — what they said matters to them.

**CONFIRM.** Explicit attribution of each action item and decision: who owns it, what it is, by when. No vague "we'll circle back." If the notes are ambiguous, flag it and ask the user to confirm before sending.

**EQUIP & NEXT — route by call outcome:**

| Call outcome (classify from the notes) | Proof artifact to attach (EQUIP) | CTA tier to request (NEXT) → from cta-variant-generator |
|---|---|---|
| **Strong intent** — they named timeline, budget, or asked "what's next" | The closest-fit customer story / ROI proof from `proof-vault` matched to their pain | **High — trial/buy.** Book the next concrete step (pilot kickoff, contract review, mutual action plan) |
| **Lukewarm** — interested, no urgency, vague timeline | One focused case study + a relevant comparison/help doc that advances evaluation | **Medium — evaluate.** Low-pressure but real: "see it on your data," loop in the named champion |
| **Objection-stalled** — a specific concern blocked momentum | Proof that directly counters the stated objection (story/benchmark) + the reframe handle from `objection-library-builder` | **Low — educate.** Resolve the objection first; ask only for a short follow-up to address it, no big commitment |
| **No-show / cut short** — meeting didn't happen or ended early | None, or one light, generically useful resource — don't over-equip a non-conversation | **Lowest — re-engage.** Acknowledge briefly, offer 2 concrete reschedule slots, no pitch |

EQUIP: real URLs only; `[verify]` any link not confirmed live. NEXT: always one primary action + one low-friction fallback, anchored to the confirmed next step in the notes.

```
## Follow-Up Email — [Prospect Name] — [Date]
Subject: [specificity > cleverness — include their company name or the exact outcome discussed]
---
[Warm one-line opener referencing a specific moment from the call]

**What we covered:**
- …

**Action items:**
| Owner | Item | Due |

**For your reference:**
- [resource 1 — matched to their pain]

[CTA from cta-variant-generator]

[Sign-off]
```

Save to `./outreach/<account-slug>/follow-up-[date].md`.

---

## Personalization discipline

The single rule: every personalized claim must trace to a real signal. Acceptable signals: LinkedIn bio, company news, job postings, the dossier from `account-dossier-builder`, something the prospect actually said in the notes. Flattery without evidence ("I've been following your work closely") is a lie and damages trust. If there's no real signal, write a generic-but-true line rather than a personalized-but-fabricated one.

---

## Principles

- **Brand-brain first.** Voice, banned words, offer, and proof come from the brand, not from inference. No copy before it returns.
- **Sourced or silent.** Every account fact either has a source or is `[verify]`-tagged. Never invent a company detail, a pain hypothesis presented as fact, or a proof point.
- **Prospect's language over internal framing.** Recap emails use words the prospect used; they should read their own priorities back to themselves.
- **Action items are attributed and time-bound.** "We'll follow up" is not an action item. Name the owner and the date.
- **Angles are computed, probes are signal-built.** Lead angles come from the role × stage selector; opening questions come from the Signal → Probe engine. Don't improvise either when the dossier gives you the inputs.
- **The follow-up branches on outcome.** A no-show and a strong-intent call do not get the same proof or the same ask. Classify, then route off the matrix.
- **Compose, don't duplicate.** Account profiling is `account-dossier-builder`'s job. Objection reframes are `objection-library-builder`'s job. Closing CTAs are `cta-variant-generator`'s job. Call them.

## What Not to Do

- Don't fabricate firmographic details, pain hypotheses as confirmed fact, or proof points — use `[verify]` or ask.
- Don't write the follow-up email before confirming ambiguous action items with the user.
- Don't reimplement account research, objection indexing, or CTA logic — chain the specialist skills.
- Don't use generic openers or closings ("Hope this email finds you well," "Looking forward to connecting").
- Don't pitch a high-commitment CTA into a lukewarm or objection-stalled call — match the tier to the outcome.
- Don't present BEACON as an external sales methodology — it's this skill's section checklist; the real discovery lineage is SPIN/MEDDIC.
- Don't store artifacts inside the skill folder; save to `./outreach/<account-slug>/`.
- Don't apply brand voice from memory — always call `brand-brain` first.

## Quality Checklist

- `brand-brain` called and digest loaded before any copy written?
- Prep: `account-dossier-builder` called; all background facts sourced or `[verify]`-tagged?
- Prep: Lead angles taken from the role × stage Angle Selector (overrides noted), not free-associated?
- Prep: Each opening question traces signal → probe type → question via the Signal → Probe engine?
- Prep: Objections pulled from `objection-library-builder` (not invented); reframe handles included?
- Follow-Up: Call outcome classified, and EQUIP + NEXT routed off the outcome matrix?
- Follow-Up: Every action item has an owner, a description, and a due date?
- Follow-Up: Resources are real, relevant, and linked to a confirmed pain signal from the notes?
- Follow-Up: CTA tier matches the outcome, written by `cta-variant-generator`, anchored to the confirmed next step?
- Both artifacts saved to `./outreach/<account-slug>/` with dated filenames?
- No fabricated flattery — every personalized claim traces to a real signal?
