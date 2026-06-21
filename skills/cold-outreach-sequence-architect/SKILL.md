---
name: cold-outreach-sequence-architect
description: >
  Designs and writes full multi-touch cold outreach sequences for a target ICP segment — email + LinkedIn,
  personalized per persona, mapped to a stated solution and channel mix. Takes an ICP segment definition,
  your solution, and a channel mix preference; returns a complete, ready-to-send sequence: subject lines,
  email bodies, LinkedIn connection notes, follow-up messages, and the messaging angle behind each touch.
  Sequences are built on the Spear Selling / PAVE framework so every message earns attention with a
  specific, relevant insight rather than generic flattery or product-first pitching. Calls brand-brain for
  voice and proof, account-dossier-builder for account-level research, and objection-library-builder for
  pre-emptive objection handling. Artifacts save to ./outreach/. Use when someone says "write cold emails,"
  "build an outreach sequence," "LinkedIn + email cadence," "cold prospecting copy," "SDR playbook," or
  "personalized outreach for [ICP/segment/persona]."
---

# Cold Outreach Sequence Architect

Build sequences that open conversations, not sequences that get filtered. Every touch earns the next one: a specific insight, a clear connection to a pain the persona actually has, and a single low-friction ask. No spray-and-pray, no "just checking in," no invented flattery.

This skill architects the sequence strategy and writes every message. It does not source the prospect list (that is `account-list-builder-icp-scorer`) and does not decide whether the channel is right for the funnel stage (that is the human's call — but it will flag if the ask exceeds the trust level the channel can support).

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — voice, banned words, offer mechanics, proof, positioning, ICP. No message written before this returns.
- **`account-dossier-builder`** (required per target account, Step 1) — company profile, tech stack, pain hypotheses, recent news. Personalization anchors come from here, never fabricated.
- **`objection-library-builder`** (Step 2) — pre-emptive objection handling woven into middle and late touches.
- **`icp-persona-builder`** (Step 2, if personas are not already in brand.md) — confirms role-level pains, language, and awareness stage per persona.
- **`cta-variant-generator`** (Step 4) — generates the sequence's CTA options (meeting ask, content offer, reply CTA) at the right commitment ceiling.
- **`subject-line-preview-text-optimizer`** (Step 4) — stress-tests subject lines on open-rate mechanics and spam risk.

---

## How a run works

```
Step 0  Brand context      ──► call brand-brain (voice, proof, offer, ICP)
Step 1  Account research   ──► call account-dossier-builder per target company
Step 2  Persona + objects  ──► call icp-persona-builder (if absent) + objection-library-builder
Step 3  Sequence strategy  ──► PAVE framework: map touches, angles, channel, cadence
Step 4  Write every asset  ──► call cta-variant-generator + subject-line-preview-text-optimizer
Step 5  Self-review        ──► quality checklist; save to ./outreach/
```

---

## Step 0 — Brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns the active brand's digest: voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning, ICP. Do not write a single message before it returns.

**Fallback if brand-brain is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`. If neither exists, ask the user for: solution + differentiator · ICP role/company-type · 3 voice adjectives + banned words · pricing/offer mechanic · 1–2 real proof points.

---

## Step 1 — Account research

For every named target company, invoke `account-dossier-builder` (Skill tool). It returns: business model, tech stack, recent news, funding, headcount signals, and hypothesized pains. This is the **only** source of personalization facts. Never fabricate specifics — if research returns nothing concrete, use segment-level insight (role + industry pattern) and mark it as such, not as account-specific.

If the user provides a list, process the top account in full and note the pattern for scaling.

---

## Step 2 — Persona and objections

Confirm the target persona's role-level pain, vocabulary, and awareness stage. If `personas.md` is in the brand's companion files, read it. If not, invoke `icp-persona-builder`.

Invoke `objection-library-builder` to get the top 3–5 objections for this ICP + solution pairing. These feed into the mid-sequence and penultimate touches.

---

## Step 3 — Sequence strategy: the PAVE framework

Every sequence is built on four angles that cycle across touches. Map the touch order before writing a word:

| Letter | Angle | What the message does |
|---|---|---|
| **P** — Problem | Opens with the pain, not the product | Earns relevance by naming something real the persona feels |
| **A** — Aspiration | Connects to the outcome they want | Moves from pain to possibility without pitching yet |
| **V** — Validation | Social proof, data, or a customer parallel | Builds credibility; uses only real proof (else `[verify]`) |
| **E** — Escalation | Addresses the objection and sharpens the ask | Clears the last barrier; one specific, low-friction CTA |

**Sequence structure (default 5-touch, 10–14 days):**

| Touch | Channel | Day | Angle | CTA commitment |
|---|---|---|---|---|
| T1 | Email | Day 1 | P — specific insight + pain | Soft: reply / resource |
| T2 | LinkedIn | Day 3 | A — connection note, shared context | Connection only |
| T3 | Email | Day 6 | V — proof point / customer parallel | Medium: 15-min call |
| T4 | LinkedIn | Day 9 | E — objection reframe + nudge | Reply CTA |
| T5 | Email | Day 13 | P (new angle) — pattern interrupt / breakup | Ultra-soft or close |

Adjust the structure for: channel mix constraints, enterprise (longer, more V touches), SMB (faster, fewer touches), warm intent signals (compress), or inbound-assisted (skip T1 cold opener, enter at T2–3).

---

## Step 4 — Write every asset

For each touch, produce:

**Email touches:**
- Subject line (A/B pair) → optimized via `subject-line-preview-text-optimizer`
- Preview text (≤90 chars)
- Body (≤150 words for T1; ≤120 for follow-ups; mobile-first — short paragraphs, no walls of text)
- CTA → via `cta-variant-generator` at the right awareness/commitment ceiling
- Personalization slots labeled `[SLOT: company_name]`, `[SLOT: recent_news]`, etc. for mail-merge

**LinkedIn touches:**
- Connection request note (≤300 chars — personalized, no pitch, finds genuine common ground)
- Follow-up message if connection accepted (≤500 chars)

**Per-touch annotation:**
- Angle and why it fits this persona at this point in the sequence
- Objection addressed (if any)
- What a positive signal looks like (reply type, click, connect) and how to route it

---

## Craft rules: what separates good sequences from noise

**The specificity test.** Every T1 must pass: "Could this have been written without knowing anything about this company or person?" If yes, rewrite it. One specific insight (a product change, a funding round, a job post, a stated priority) beats five generic value props.

**Pain before product.** T1 names a problem; it does not pitch. The product enters at T3 at the earliest, via a proof point, not a feature list.

**Commitment ladder.** T1–2 asks only for attention or a connection. T3 asks for 15 minutes. T5 offers a door to exit gracefully. Never ask for more than the channel + trust level can support.

**LinkedIn is not email.** Connection notes get one genuine shared-context line plus an implicit reason to connect — no CTA, no pitch. The message after acceptance is still conversational.

**Breakup messages earn replies.** T5 explicitly gives permission to say no. That psychological release is why breakup emails have the highest reply rates. Write it honestly: "If this isn't relevant, say the word and I'll stop."

**Voice and proof are non-negotiable.** Every message is in the brand's voice (from brand-brain). Proof is real or `[verify]`. No invented customers, invented results, or invented flattery.

---

## Deliverable format

Save to `./outreach/<brand-slug>-<icp-slug>-sequence.md`:

```
# Cold Outreach Sequence — [Brand] × [ICP Segment]
Generated: [date]  ·  Brand: [slug]  ·  Persona: [role/title]  ·  Company type: [segment]
Sequence type: [5-touch default | custom]  ·  Channel mix: [email + LinkedIn | email-only | etc.]

## Sequence strategy
[PAVE angle map — one table row per touch]

## Assets

### T1 — Email (Day 1)  ·  Angle: Problem
Subject A: ...
Subject B: ...
Preview: ...
Body:
---
[body text with [SLOT:] labels]
---
CTA: ...
Personalization slots: [list]
Route if replies: [what a positive signal looks like]

### T2 — LinkedIn (Day 3)  ·  Angle: Aspiration
Connection note: ...
Post-connect message: ...

[... T3–T5 in same format ...]

## Objections addressed
[Which objections from objection-library-builder appear in which touches]

## Scaling notes
[How to apply this sequence to the rest of the account list; where to fork for a second persona]
```

---

## Principles

- **Research before writing.** No personalization without a real fact from account-dossier-builder or the user. "Your company is growing fast" is not personalization.
- **Brand-brain first, no exceptions.** Voice + banned words override everything. A message that contradicts them gets rewritten, not submitted.
- **Commitment ladder is sacred.** Never exceed what the channel + relationship stage can reasonably support.
- **Pain before product.** The sequence earns the right to pitch; it does not start there.
- **Honest proof only.** Every statistic, customer name, and result must be real or marked `[verify]`. Invented social proof erodes trust the moment it's checked.
- **One CTA per message.** Never give a prospect two things to do; give them one easy thing to do.

## What Not to Do

- Do not write T1 before `brand-brain` returns and `account-dossier-builder` has run for the target.
- Do not personalize with invented facts ("I saw your company was recently in the news about X" when you don't know).
- Do not pitch the product in T1 or T2 — earn the conversation first.
- Do not write more than 150 words per email body; if it's longer, cut.
- Do not use the same angle across back-to-back touches — vary PAVE deliberately.
- Do not use banned words from the brand digest; do not import adjectives from a competitor's messaging.
- Do not skip the breakup message — it is the highest-ROI touch in the sequence.

## Quality Checklist

- `brand-brain` called and digest loaded before any message was written?
- `account-dossier-builder` ran for the target; all personalization slots are real, not fabricated?
- PAVE angles mapped and each touch advances the angle correctly?
- T1 passes the specificity test (cannot be generic-copy-pasted to any company)?
- Commitment ladder honored: soft → medium → escalation, channel-appropriate?
- LinkedIn notes ≤300 chars, no pitch, genuine common ground?
- CTAs generated via `cta-variant-generator` at the right ceiling?
- Subject lines stress-tested via `subject-line-preview-text-optimizer`?
- Proof is real or `[verify]`; no invented customers or results?
- Objections from `objection-library-builder` woven into T3–T4?
- Saved to `./outreach/<brand-slug>-<icp-slug>-sequence.md` with all slots labeled?
