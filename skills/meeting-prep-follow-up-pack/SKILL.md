---
name: meeting-prep-follow-up-pack
description: >
  Turns a prospect's LinkedIn profile, account dossier, or raw meeting notes into two polished
  sales-enablement assets: a pre-call brief (research digest, stake-holder context, agenda,
  opening questions, and anticipated objections) and a personalized post-meeting follow-up email
  (recap, confirmed next steps, attributed action items, and curated resources). Chains
  account-dossier-builder for firmographic depth, objection-library-builder for reframe handles,
  and cta-variant-generator for the follow-up email's closing CTA. Saves both artifacts to
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

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`. If none exists, ask the user to install `brand-brain` or answer a 4-question mini-setup before proceeding.

### Step 1 — Pick the mode

| Mode | Trigger | Input needed |
|---|---|---|
| **Prep** (default) | "prep me," "pre-call brief," "research this prospect" | Prospect LinkedIn URL or name + company URL |
| **Follow-Up** | "write the follow-up," "send the recap," meeting notes / transcript in | Meeting notes or transcript, confirmed next steps |
| **Both** | "full pack," "prep + follow-up," or both inputs present | Both of the above |

When input is ambiguous, default to Prep and offer Follow-Up at the end.

---

## Prep mode — the pre-call brief

### Framework: BEACON

**B — Background.** Company essentials sourced from `account-dossier-builder` (size, model, stack, funding, recent news). Never fabricate; mark gaps `[verify]`.

**E — Entry point.** The prospect's role, reported-to chain, likely mandate, and why they took the meeting. Infer from title + company stage; flag inferences explicitly.

**A — Angles.** 2–3 pain hypotheses mapped to the brand's ICP from `brand-brain`. Each hypothesis names the pain, the evidence for it (job post language, tech-stack signal, recent news), and the relevant product angle.

**C — Conversation openers.** 3–5 specific, researched opening questions — not generic discovery ("What are your goals?") but informed probes ("I noticed you hired three SDRs in Q1 — are you running into [pain] at that scale?"). Questions must cite their signal source.

**O — Objections.** Pull top 3 relevant objections from `objection-library-builder`; include one-sentence reframe handle per objection.

**N — Next step.** Pre-set the ideal outcome of this meeting: what does "winning" look like, and what CTA should you ask for at the end?

```
## Pre-Call Brief — [Prospect Name], [Company] — [Date]
### Background (via account-dossier-builder)
### Entry Point
### Pain Angles
### Opening Questions (with signal source)
### Objection Prep (via objection-library-builder)
### Target Next Step
```

Save to `./outreach/<account-slug>/prep-brief-[date].md`.

---

## Follow-Up mode — the post-meeting email

### Framework: RECAP → CONFIRM → EQUIP → NEXT

**RECAP.** 3–5 bullet recap of what was discussed. Use the prospect's language (pulled from their actual quotes in the notes), not internal framing. One sentence of value acknowledgment — what they said matters to them.

**CONFIRM.** Explicit attribution of each action item and decision: who owns it, what it is, by when. No vague "we'll circle back." If the notes are ambiguous, flag it and ask the user to confirm before sending.

**EQUIP.** 1–2 curated resources directly relevant to the pain angles surfaced in the meeting: a case study from `proof-vault` (or `brand.md`'s proof section), a relevant help doc, or a comparison page. Real URLs only; `[verify]` any link not confirmed live.

**NEXT.** The closing CTA from `cta-variant-generator`: one primary action (book the demo, share with the champion, schedule the pilot kickoff) and one low-friction fallback. Anchored to the confirmed next step from the meeting notes.

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
- **Compose, don't duplicate.** Account profiling is `account-dossier-builder`'s job. Objection reframes are `objection-library-builder`'s job. Closing CTAs are `cta-variant-generator`'s job. Call them.

## What Not to Do

- Don't fabricate firmographic details, pain hypotheses as confirmed fact, or proof points — use `[verify]` or ask.
- Don't write the follow-up email before confirming ambiguous action items with the user.
- Don't reimplement account research, objection indexing, or CTA logic — chain the specialist skills.
- Don't use generic openers or closings ("Hope this email finds you well," "Looking forward to connecting").
- Don't store artifacts inside the skill folder; save to `./outreach/<account-slug>/`.
- Don't apply brand voice from memory — always call `brand-brain` first.

## Quality Checklist

- `brand-brain` called and digest loaded before any copy written?
- Prep: `account-dossier-builder` called; all background facts sourced or `[verify]`-tagged?
- Prep: Opening questions cite a named signal, not generic discovery boilerplate?
- Prep: Objections pulled from `objection-library-builder` (not invented); reframe handles included?
- Follow-Up: Every action item has an owner, a description, and a due date?
- Follow-Up: Resources are real, relevant, and linked to a confirmed pain signal from the notes?
- Follow-Up: CTA from `cta-variant-generator`, anchored to the confirmed next step?
- Both artifacts saved to `./outreach/<account-slug>/` with dated filenames?
- No fabricated flattery — every personalized claim traces to a real signal?
