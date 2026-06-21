---
name: referral-email-sequence-writer
description: >
  Writes a complete referral email drip sequence — invite, reminder, and reward
  confirmation — for any referral program, in the brand's real voice, anchored to
  a named incentive-legibility framework. Covers structural arc (invite → nudge →
  reward confirm), per-email anatomy (subject + preview text + body + CTA), copy
  critique on whether the incentive is legible (does the reader instantly know what
  they get for referring AND what their friend gets?), and CTA clarity audit per
  send. Composes `subject-line-preview-text-optimizer`, `cta-variant-generator`,
  `proof-vault`, and `lifecycle-email-push-copy-reviewer` rather than re-implementing
  their logic. Does NOT design the referral program mechanics (use `referral-program-brief-builder`)
  or build the trigger automation (use `usage-triggered-message-sequencer`). Invoke when
  the user says "write referral emails," "referral invite sequence," "referral drip,"
  "refer-a-friend emails," "write the remind + reward emails," "referral program copy,"
  or hands over a referral brief and asks for the sequence.
---

# Referral Email Sequence Writer

A referral program lives or dies on email. The mechanic can be perfect — and still fail because the invite email buried the incentive, the reminder landed as spam, or the reward confirmation felt like a form letter instead of a win. This skill writes the three-email spine that makes programs convert: **invite → reminder → reward confirmation**.

Every email is written in the brand's real voice (loaded from `brand-brain`), anchored to the **Dual Incentive Legibility Test** so readers know — within two seconds — what they get *and* what their friend gets, with subject lines and CTAs optimized through the library's dedicated skills.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — resolves the active brand; returns voice, banned words, offer mechanics, real proof, ICP, positioning. Does not proceed without it.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP + awareness stage, referral incentive mechanics (referrer reward, friend reward, redemption steps), and 3 voice adjectives + banned words before proceeding.
- **`subject-line-preview-text-optimizer`** — optimizes subject + preview text for each email in the sequence after drafting.
- **`cta-variant-generator`** — produces the primary CTA button label and one alternate per email; handles awareness-ceiling matching for referral context.
- **`proof-vault`** (optional, use if installed) — supplies real social-proof microcopy that can appear in the invite or reward confirmation. Synthesize inline if absent.
- **`lifecycle-email-push-copy-reviewer`** — reviews the completed sequence for brand voice, flow, CTA strength, urgency/clarity, and character-limit compliance before delivery.

---

## How a run works

```
Step 0  Load the brand            ──► brand-brain
Step 1  Intake the program brief  ──► incentive, audience segment, program URL
Step 2  Diagnose incentive legibility (gate)
Step 3  Draft the 3-email sequence
Step 4  Optimize subjects + CTAs  ──► subject-line-preview-text-optimizer + cta-variant-generator
Step 5  Review pass               ──► lifecycle-email-push-copy-reviewer
Step 6  Deliver + save
```

---

## Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand digest: voice adjectives, banned words, offer mechanics, real proof, ICP + awareness tendency. Use the returned voice and banned-words as hard overrides. Mark any unconfirmed number `[verify]`. Do not write a single email line before brand-brain returns.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP + awareness stage, referral incentive mechanics (referrer reward, friend reward, redemption steps), and 3 voice adjectives + banned words before proceeding.

---

## Step 1 — Intake the program brief

Collect (from the user's input or the program brief; ask only for what's genuinely missing):

| Input | Why it matters |
|---|---|
| **Referrer reward** | The hook of every invite; must be legible in one line |
| **Friend/referee reward** | The second half of the dual incentive — often omitted |
| **Redemption steps** | How many clicks between "refer" and "reward"? |
| **Referral link mechanics** | Unique link / promo code / dashboard, so CTA destination is real |
| **Audience segment** | Who gets the invite — best customers, all actives, a specific cohort |
| **Program timing** | Ongoing evergreen or a time-bounded push? |
| **Existing program URL** | Required for CTAs |

If the user is starting from a rough brief, note what's missing and ask in a single batched question.

---

## Step 2 — Dual Incentive Legibility Test (gate)

Before drafting, apply the **Dual Incentive Legibility Test** (DILT): within two seconds of reading the invite's subject + first 50 words, a reader must know:

1. **What the referrer gets** — amount, type (credit / cash / discount), and when.
2. **What the friend gets** — same clarity. If the program is one-sided (referrer-only), flag this; single-sided incentives reduce friend conversion.
3. **How many steps to claim** — if it's more than two clicks post-referral, name the friction and let the user decide whether to simplify.

If the incentive is unclear or incomplete, surface the gap and offer a fill-in before writing. Do not paper over vague mechanics with clever copy.

---

## Step 3 — Draft the three-email sequence

### The structural arc

| Email | Timing | Job | Emotional register |
|---|---|---|---|
| **1 — Invite** | Day 0 (trigger: loyalty milestone, NPS 9+10, or manual send) | Plant the seed; make the dual incentive unmissable | Warm, generous, peer-to-peer |
| **2 — Reminder** | Day 5–7 (only to non-clickers) | Re-surface the offer; add a soft deadline or social proof nudge | Light urgency, no pressure |
| **3 — Reward Confirm** | Within hours of reward trigger | Close the loop; celebrate the win; re-invite for another share | Celebratory, specific, re-engaging |

### Per-email anatomy

Each email is written as a complete, production-ready asset:

```
[EMAIL N — Name]
Subject: (placeholder; optimized in Step 4)
Preview text: (placeholder; optimized in Step 4)
From name: (from brand brief)

[Body — target ~120–200 words for invite/reminder; ~80–120 for reward confirm]
Opening: WIFM hook — referrer benefit, first sentence, no preamble
Dual incentive callout: one highlighted line or visual callout block
(referrer reward) + (friend reward) — both explicit
Social proof line: [real proof from proof-vault, or "[verify]" placeholder]
CTA: (placeholder; finalized in Step 4)
Microcopy: redemption mechanics in one line (e.g., "Reward lands in your account within 3 business days of your friend's first purchase")
Footer: unsubscribe + physical address (CAN-SPAM requirement, 15 U.S.C. § 7704)
```

**Invite (Email 1) craft notes:**
- Subject line angle: curiosity + specificity (exact reward amount) typically outperforms clever-vague for referral
- Peer framing: "We're asking you specifically" — earned-customer acknowledgment, not broadcast blast
- Dual-incentive callout: use a visually distinct block (bold text or a simple two-column summary) so it survives skim-reading
- One primary CTA; no secondary links competing for the click

**Reminder (Email 2) craft notes:**
- Open with a callback, not a repeat: "Still thinking about it?" / "Your [reward] is waiting" — not a re-send of Email 1
- Add one new element: a real testimonial, a deadline ("ends [date]"), or a usage-triggered social proof stat — not fabricated
- Shorter body than the invite; the decision was seeded, this is activation
- Same CTA destination; don't introduce a different link

**Reward Confirmation (Email 3) craft notes:**
- Lead with the win: "[Name], your [reward] is confirmed" in the subject + first line
- Specificity over enthusiasm: exact credit amount and where it appears, not just "Congrats!"
- Re-invite hook (optional but high-ROI): "Know someone else who'd love [brand]?" — keep it a soft three-word nudge, not a second sequence pitch
- No incentive ambiguity: tell them exactly when and where to redeem

---

## Step 4 — Optimize subjects + CTAs

After drafting all three emails:

1. **Invoke `subject-line-preview-text-optimizer`** for each email, passing the draft subject, preview text, audience segment, and incentive summary. Integrate the returned optimized lines.
2. **Invoke `cta-variant-generator`** for each email's primary CTA, passing the draft button copy, placement (email), awareness stage (product-aware to most-aware for a referral audience), and the offer/destination. Integrate the recommended CTA + one alternate per email.

---

## Step 5 — Review pass

Invoke `lifecycle-email-push-copy-reviewer` on the completed three-email sequence. Address any flagged issues (voice drift, CTA weakness, urgency clarity, character-limit violations) before delivering.

---

## Step 6 — Deliver + save

Present the sequence as three complete, production-ready email assets. If the user asks to save, write to `./referral-sequences/[brand-slug]-referral-sequence.md`. Never overwrite `brand.md`.

---

## Incentive Legibility Framework (DILT in practice)

The most common referral email failure mode is a legible incentive for the referrer but a vague or absent incentive for the friend. Research on two-sided referral programs (Dropbox's canonical "500 MB + 500 MB" structure [verify for current benchmarks]) shows that friend-side incentive clarity is the single largest driver of invite acceptance rate.

Apply these three legibility checks to every draft:

| Check | Pass condition |
|---|---|
| **L1 — Referrer reward** | Visible in first 50 words, with exact amount/type and timing |
| **L2 — Friend reward** | Explicitly stated alongside referrer reward; not buried in fine print |
| **L3 — Redemption steps** | At most two steps described in the email body ("Share your link → friend signs up → [reward method]") |

If L2 fails (single-sided program), note it as a conversion risk and recommend the user add a friend reward before launch. If L3 > 2 steps, flag as friction.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No copy before brand-brain returns. Its voice + banned-words override everything here.
- **Compose, don't duplicate.** Subject lines go through `subject-line-preview-text-optimizer`; CTAs go through `cta-variant-generator`; final review goes through `lifecycle-email-push-copy-reviewer`.
- **DILT before drafting.** Ambiguous incentives do not get papered over with better copy — they get fixed first.
- **Peer-to-peer register.** Referral emails work because they feel personal, not promotional. Avoid broadcast-ad tone.
- **Redemption transparency.** Every email tells the reader exactly how and when the reward arrives. Vague "you'll be rewarded" copy erodes trust.
- **CAN-SPAM compliance.** Every email includes a physical postal address and a clear unsubscribe mechanism (15 U.S.C. § 7704). Mark [verify] for any jurisdiction-specific additions (GDPR, CASL).
- **Real proof only.** Social-proof stats from `proof-vault` or marked `[verify]`; never invented.

---

## What Not to Do

- Don't write copy before `brand-brain` returns — and don't skip the DILT gate.
- Don't reimplement subject-line optimization or CTA generation here — delegate to the relevant skills.
- Don't design the referral program mechanics (tier structure, attribution logic, fraud rules) — that's `referral-program-brief-builder`.
- Don't build the trigger workflow or enrollment automation — that's `usage-triggered-message-sequencer`.
- Don't write a single-sided incentive invite without flagging the conversion risk.
- Don't use fake urgency ("Only 24 hours left!" when the program is evergreen) — if no deadline exists, don't invent one.
- Don't write the reminder as a re-send of the invite; it must add new information (proof, deadline, or social proof).

---

## Quality Checklist (self-review before delivering)

- `brand-brain` called and active brand loaded (or bootstrapped) before any copy was written?
- DILT applied: referrer reward visible in first 50 words, friend reward explicitly stated, redemption ≤2 steps?
- All three emails drafted: invite (Day 0) + reminder (Day 5–7, non-clickers only) + reward confirmation (within hours of trigger)?
- Each email has: optimized subject + preview text (via `subject-line-preview-text-optimizer`), on-brand body, dual-incentive callout, optimized CTA + one alternate (via `cta-variant-generator`), redemption-timing microcopy, CAN-SPAM footer?
- Reminder opens with new information (not a re-send); reward confirm leads with the specific win?
- `lifecycle-email-push-copy-reviewer` run; all flagged issues addressed?
- Voice + banned-words honored throughout; unconfirmed proof marked `[verify]`?
- Single-sided incentive risk flagged if applicable? Fake urgency absent?
