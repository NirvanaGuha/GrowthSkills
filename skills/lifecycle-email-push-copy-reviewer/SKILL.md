---
name: lifecycle-email-push-copy-reviewer
description: >
  Editorial review skill for lifecycle email sequences and web/app push notification copy. Takes a drafted
  sequence (welcome, onboarding, win-back, post-purchase, upgrade, transactional, re-engagement) or push
  notification variants and returns a structured, opinionated critique covering brand voice, narrative flow,
  CTA strength, urgency/clarity balance, character-limit compliance, and send-cadence logic. Does NOT
  rewrite the copy from scratch — it audits it, scores it, and hands back a prioritized fix list so the
  author knows exactly what to repair and why. Calls brand-brain to load the active brand's voice and
  banned-word list before evaluating a single line. Call this skill when you say "review my email sequence,"
  "QA this push copy," "is this on-brand," "check these emails before we send," "proofread my drip,"
  "audit my lifecycle copy," "is the tone right," "do these emails flow," or hand over a sequence draft
  and ask whether it's ready to send.
---

# Lifecycle Email & Push Copy Reviewer

An editorial QA pass for lifecycle email sequences and push notification copy — before they ship. Brand voice, narrative arc, CTA mechanics, urgency honesty, and compliance flags in one structured pass; a prioritized fix list the writer can action immediately.

This skill **reviews**. It does not rebuild sequences from scratch, generate ESP logic, or configure send-time rules — those live in the sibling skills listed below. If a sequence doesn't exist yet, start with `welcome-onboarding-email-sequence-builder`, `lead-nurture-drip-builder`, or `push-notification-copy-generator`, then run this reviewer.

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads the active brand's voice adjectives, banned words, offer mechanics, real proof, and positioning. The reviewer never evaluates copy without this context. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their brand's voice adjectives, banned words, offer/guarantee details, and ICP before proceeding.
- **`email-compliance-auditor-gdpr-can-spam`** *(call when compliance flag is raised)* — full GDPR/CAN-SPAM pass; this reviewer surfaces the flag, that skill resolves it.
- **`subject-line-preview-text-optimizer`** *(call when subject/preview lines score below 3/5)* — rewrites weak subject + preview pairs.
- **`cta-variant-generator`** *(call when a CTA scores below 3/5)* — generates stronger CTA options for the flagged step.
- **`proof-vault`** *(reference when proof is cited but unverified)* — confirms or flags proof claims.

---

## How a run works

```
Step 0  Load brand context   ──► brand-brain (always before touching copy)
Step 1  Ingest the sequence  ──► email sequence OR push batch; identify type + stage
Step 2  Run the VOICE framework  ──► 5-dimension score per message
Step 3  Evaluate the arc     ──► inter-message flow, cadence, fatigue risk
Step 4  Flag compliance      ──► unsubscribe, proof substantiation, urgency honesty
Step 5  Compile fix list     ──► P1/P2/P3 priority, actionable; offer sibling calls
Step 6  Final verdict        ──► Send-ready / Fix then send / Do not send
```

---

## Step 0 — Load brand context (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`) before reading a line of copy. It returns the active brand's voice adjectives, banned words, offer mechanics, real proof, positioning, and ICP.

**Fallback if brand-brain is absent or returns no brand:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their brand's voice adjectives, banned words, offer/guarantee details, and ICP before proceeding.

Use the returned voice as the yardstick for every voice score. A banned word anywhere in the sequence is an automatic P1 issue.

---

## Step 1 — Ingest and classify

Identify:
- **Sequence type** — welcome/onboarding · post-purchase · lead-nurture · win-back/re-engagement · upgrade/upsell · event/webinar · transactional · push batch
- **Stage in the lifecycle** — acquisition → activation → retention → reactivation
- **Channel** — email · web push · in-app push · SMS (flag if SMS — character rules differ)
- **Message count and declared cadence** (if provided)

If the sequence type or channel is ambiguous, ask in one line before proceeding.

---

## Step 2 — The VOICE Framework (score every message)

Score each message on five dimensions, 1–5 each. A score below 3 on any dimension is a fix candidate; below 2 is a P1 block.

| Dimension | What it evaluates | Scoring anchor |
|---|---|---|
| **V**oice fidelity | Matches the brand's adjectives; zero banned words; consistent register | 5 = indistinguishable from brand; 1 = could be anyone's copy |
| **O**pening pull | Subject + preview, or push title + body — stops the scroll, earns the open | 5 = specific, curiosity or value-led, no clickbait; 1 = generic or misleading |
| **I**ntent clarity | One job per message; reader knows exactly what they're being asked to do | 5 = single, obvious action; 1 = multiple competing asks or no ask |
| **C**TA strength | Button/link label is action-led, specific, and fits the awareness stage | 5 = strong verb + clear value, right commitment ceiling; 1 = "Click here" / "Submit" |
| **E**vidence honesty | Proof claims are real and attributed; urgency is genuine not manufactured | 5 = real proof, honest scarcity; 1 = invented numbers or fake countdown |

For push notifications: character-count compliance is a hard gate (not a dimension score). Standard web push limits: title ≤50 chars, body ≤125 chars on most platforms. Flag overages as P1.

**Per-message output format:**
```
Message [N] — [subject line or push title]
V: [1–5] · O: [1–5] · I: [1–5] · C: [1–5] · E: [1–5]  →  Avg: [X.X]
Issues: [bulleted; each issue includes the exact line and the fix direction]
```

---

## Step 3 — Sequence arc review

After scoring individual messages, evaluate the sequence as a whole:

**Narrative arc.** Does each message assume the right context — i.e., that the reader received the prior messages? Is there a logical emotional and informational build (orient → engage → activate → deepen → retain)? Or does the sequence restart the pitch on every send?

**Cadence and fatigue.** Are the declared or implied gaps between sends appropriate for the sequence type? Common benchmarks [verify for your audience and ESP]:
- Welcome: D+0 immediately → D+1 → D+3 → D+7
- Post-purchase: send within 1h of trigger → D+3 review ask → D+7 cross-sell
- Win-back: D+30 → D+37 → D+44 with sunset branch
- Lead-nurture: 2–4 day gaps to avoid unsubscribes

Flag if cadence appears to be guess-work rather than lifecycle-stage logic.

**Message redundancy.** Are any two messages making the same offer with the same angle? Redundancy is not reinforcement — it is fatigue. Flag and recommend a different angle or cut.

**Escalation logic.** Does the sequence escalate appropriately — lower-commitment asks early, higher-commitment asks once trust is established? A pricing hard-sell in message 2 of a welcome sequence is a P1 arc failure.

---

## Step 4 — Compliance and honesty flags

These are pass/fail gates, not scores. Any fail is a P1 block.

**Email compliance (surface flags; call `email-compliance-auditor-gdpr-can-spam` to resolve):**
- CAN-SPAM (15 USC §7704): physical mailing address required in every commercial email; unsubscribe mechanism must be present and honored within 10 business days; subject line must not be deceptive.
- GDPR (Art. 7): consent must be documented for EU recipients; pre-ticked opt-in boxes are prohibited; withdrawal must be as easy as consent.
- CASL: express or implied consent for Canadian recipients; sender identification required.

**Urgency and scarcity honesty:** Is any deadline, stock limit, or discount expiry verifiable? "Offer expires tonight" is a lie if the offer runs monthly — flag as P1. Real, verifiable urgency is fine.

**Proof substantiation:** Any statistic, customer result, or comparative claim must be sourced or marked `[verify]`. Invented social proof is a legal and brand-trust risk.

---

## Step 5 — Prioritized fix list

Compile all issues from Steps 2–4 into a single prioritized list:

- **P1 — Block before send:** compliance failures, banned words, fake urgency, invented proof, VOICE dimension ≤ 2, push character overages, arc-level escalation failures.
- **P2 — Fix before send (high impact):** VOICE dimensions 2–3, weak CTAs, redundant messages, cadence mismatch, unverified proof claims.
- **P3 — Consider fixing (polish):** minor voice drift, style inconsistency, subject line optimization opportunities.

For each issue: *which message → what the problem is → the exact fix direction*. If a sibling skill resolves it better, say so (e.g., "run `subject-line-preview-text-optimizer` on messages 2 and 4").

---

## Step 6 — Verdict

Close with one of three verdicts:

- **Send-ready.** No P1 or P2 issues; any P3 items are optional polish.
- **Fix then send.** P2 issues present; P1 issues absent. Fix list attached — address P2s, re-run if uncertain.
- **Do not send.** One or more P1 blocks. Sending as-is risks compliance exposure, brand damage, or audience trust erosion.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No copy touches until `brand-brain` returns the active brand. Its voice and banned-word list are hard overrides, not suggestions.
- **Review, don't rebuild.** Critique what exists; flag the fix direction; call the sibling that rewrites. This skill does not generate full sequences.
- **Score honestly.** A 4/5 that deserves a 2/5 wastes the operator's time. If copy is weak, say so and say why.
- **One P1 = Do Not Send.** Compliance blocks are not negotiable; fake urgency and invented proof destroy long-term deliverability and brand trust.
- **Specificity over volume.** Five precise, actionable issues beat fifteen vague impressions. Name the line; give the fix direction.
- **Real proof or `[verify]`.** Never confirm a stat or customer result this skill cannot verify — mark it and point to `proof-vault`.

---

## What Not to Do

- Don't evaluate copy before `brand-brain` returns the active brand context.
- Don't rewrite sequences from scratch — route to `welcome-onboarding-email-sequence-builder`, `lead-nurture-drip-builder`, or `push-notification-copy-generator`.
- Don't provide GDPR/CAN-SPAM resolution guidance beyond flagging — call `email-compliance-auditor-gdpr-can-spam`.
- Don't fabricate benchmarks (open rates, optimal cadence gaps) without a `[verify]` flag.
- Don't conflate a low-quality voice score with a compliance issue — score them separately.
- Don't mark a sequence "Send-ready" if any P1 exists, regardless of how minor it looks.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or fallback executed) before any evaluation?
- Every message scored on all five VOICE dimensions with specific issue notes, not just a summary?
- Sequence arc reviewed for narrative build, cadence, redundancy, and escalation logic?
- CAN-SPAM / GDPR / CASL compliance flags surface-checked; P1 blocks called out?
- Push notifications checked for character-count compliance (title ≤50, body ≤125)?
- Urgency and proof claims checked for honesty; invented claims marked P1?
- Fix list sorted P1 / P2 / P3 with message reference and fix direction for each item?
- Sibling skill calls recommended where applicable (subject-line optimizer, cta-variant-generator, compliance auditor)?
- Final verdict issued: Send-ready / Fix then send / Do not send?
- Artifacts saved to `./lifecycle-review/[sequence-slug]-review.md` if the user asked for a saved report?
