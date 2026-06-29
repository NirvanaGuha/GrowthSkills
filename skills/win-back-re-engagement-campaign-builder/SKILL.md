---
name: win-back-re-engagement-campaign-builder
description: >
  Takes a lapsed-segment definition and churn reasons and builds a complete win-back
  campaign: an escalating-incentive email + push sequence, a sunset branch that protects
  deliverability, and a reactivation offer logic map tied to churn reason. Operates on
  the RFM / lapse-window model — each touchpoint earns back trust before it asks for the
  transaction. Brand context, voice, proof, and offer mechanics come from `brand-brain`;
  copy for individual emails comes from composing `subject-line-preview-text-optimizer`,
  `push-notification-copy-generator`, and `cta-variant-generator`; the reviewer pass
  uses `lifecycle-email-push-copy-reviewer`. Invoke when the user says "win-back campaign,"
  "re-engagement sequence," "lapsed users/subscribers/customers," "churn win-back,"
  "sunset flow," "we need to wake up dormant customers," "re-activate," or hands over
  a churn-reason list and asks what to send.
---

# Win-Back & Re-engagement Campaign Builder

Lapsed users don't need a louder version of the message they already ignored. They need the right reason, at the right moment, from a brand that has earned the re-ask. This skill builds the full campaign architecture: segment definition, churn-reason routing, an escalating sequence that holds back the discount until trust is rebuilt, and a clean sunset branch that preserves list hygiene and deliverability if they don't come back.

Every touchpoint is written in the brand's real voice, anchored to its real offer, using real proof — because those come from `brand-brain`, not from this skill.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads voice, banned words, offer mechanics, proof, and ICP. Win-back copy does not reimplement brand resolution.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand voice adjectives, banned words, the primary reactivation offer (discount, feature, content), and the main churn reasons before proceeding.
- **`subject-line-preview-text-optimizer`** — optimizes subject lines and preview text for every email in the sequence; call per touchpoint.
- **`push-notification-copy-generator`** — writes push touchpoints where the channel is in scope.
- **`cta-variant-generator`** — produces CTA options for re-activation landing page / in-email button; call after sequence is drafted.
- **`lifecycle-email-push-copy-reviewer`** — review pass on the finished sequence for brand voice, CTA strength, urgency calibration, and character-limit compliance. Run after drafting, before presenting.
- **`proof-vault`** *(optional)* — surfaces testimonials and proof points to embed in mid-sequence social-proof beats.
- **`email-compliance-auditor-gdpr-can-spam`** *(optional but recommended)* — audits the sunset branch for compliant opt-out handling (CAN-SPAM physical address, one-click unsubscribe, honest-sender identity).

---

## How a run works

```
Step 0  Load brand context        ──► call brand-brain
Step 1  Define the lapsed segment ──► lapse window, RFM tier, churn-reason tags
Step 2  Map the escalation logic  ──► 3-zone incentive ladder + sunset branch
Step 3  Draft the sequence        ──► compose subject-line + copy + push touchpoints
Step 4  Review pass               ──► lifecycle-email-push-copy-reviewer
Step 5  Output the campaign pack  ──► sequence table + churn-reason routing + sunset spec
```

---

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill before drafting a single line. It returns voice adjectives, banned words, offer mechanics + destinations, real proof, ICP, and the brand slug. Hold back until it returns.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand voice adjectives, banned words, the primary reactivation offer (discount, feature, content), and the main churn reasons before proceeding.

---

### Step 1 — Define the lapsed segment

Before scheduling a single send, nail the three inputs that determine what the sequence should say:

**Lapse window** — what does "lapsed" mean for this brand?
- eCommerce / transactional: 60–90 days no purchase (adjust for AOF — annual purchase brands need a longer window).
- SaaS / subscription: 14–30 days no login or key-feature use; or first churn date.
- Content / newsletter: 60–90 days no open/click (use engagement-based definition, not just send date).

If the user hasn't defined this, ask. Use the RFM frame: segment lapsed contacts into At-Risk (still warm, last activity 30–60d), Hibernating (60–120d), and Lost (120d+). Sequence depth and incentive ceiling differ per tier.

**Churn-reason tags** — why did they lapse? Map to one of five canonical reasons:
1. Price/value misalignment — "too expensive," "didn't see ROI," feature unused
2. Life event / timing — "not ready," seasonal, role change
3. Competitive loss — switched to a named alternative
4. Experience/quality issue — bad onboarding, bug, support failure
5. Passive drift — just… stopped; no strong reason

Each tag routes to a different opening angle and offer in Step 3.

**Channel stack** — email only, email + push, or email + push + SMS? Confirm before drafting.

---

### Step 2 — Map the escalation logic (the RFB Ladder)

Our working model is the **RFB Ladder: Reconnect → Remind → Buy** (a house framework, not an industry standard). Three zones, escalating commitment, held-back discount.

```
Zone 1 — RECONNECT  (Touches 1–2)
  Goal: re-open the relationship, not sell.
  Tone: warm curiosity, low-pressure.
  Offer: zero. Content, value, insight, or a product update they missed.
  Push: optional — softer subject line, curiosity gap.

Zone 2 — REMIND  (Touch 3)
  Goal: surface the specific value they haven't used or the proof they may have missed.
  Tone: social proof + relevant result, matched to their churn-reason tag.
  Offer: soft — free resource, feature walkthrough, case study from someone in their situation.
  Subject line: reference their inactivity directly but without guilt-tripping.

Zone 3 — BUY  (Touch 4–5)
  Goal: close the re-activation with the strongest offer you're willing to make.
  Tone: honest urgency — real deadline, real offer.
  Offer: the incentive. Escalate in two beats:
    Touch 4: smaller incentive (%, trial extension, bonus credit)
    Touch 5: maximum incentive + explicit deadline
  Never reveal the max offer on Touch 4 — let the sequence do the work.

SUNSET BRANCH  (Touch 6, only if no engagement by T5)
  Goal: confirm opt-out intent cleanly; protect deliverability.
  Tone: respectful, zero pressure — "we'll stop emailing you."
  Offer: none, or a single "last chance" line that doesn't beg.
  Action: if no open/click → suppress from broadcast, retain in re-permission segment (90d).
  Compliance: CAN-SPAM physical address, one-click unsubscribe, honest sender identity [verify jurisdiction].
```

---

### Step 3 — Draft the sequence

For each touchpoint in the table below, compose the elements by calling sibling skills:

- **Subject line + preview text** → call `subject-line-preview-text-optimizer` with the churn reason, zone, and brand voice.
- **Push copy** (if in stack) → call `push-notification-copy-generator`.
- **CTA button copy** → call `cta-variant-generator` (Quick mode by default; Battery on request).
- **Proof beats (Zone 2–3)** → pull from `proof-vault` if available; inline `[verify]` otherwise.

**Churn-reason routing table** — override the Zone 1 opening angle per tag:

| Churn-reason tag | Zone 1 angle | Zone 2 proof beat | Zone 3 offer flavor |
|---|---|---|---|
| Price/value misalignment | Lead with a result, not a feature | ROI proof or saved-time stat | % discount or extended trial |
| Life event / timing | Acknowledge the gap without assumption | "Here's what's changed since you left" | Easy re-entry (low-friction, no long commitment) |
| Competitive loss | Don't mention the competitor — lead with a differentiator they may not have seen | Head-to-head proof point or migration story | Free migration / onboarding session [verify if this offer exists] |
| Experience/quality issue | Open with the fix: "We heard this was broken. It's fixed." | Before/after or changelog entry | Extended trial or credited refund |
| Passive drift | Lead with curiosity — "Here's what you've been missing" | Social proof from similar users | Standard incentive + lowest-friction CTA |

**Sequence table template** (fill per brand):

```
| Touch | Day | Zone | Channel | Subject line (draft) | Body angle | CTA | Incentive |
|-------|-----|------|---------|---------------------|------------|-----|-----------|
| T1    | 0   | Reconnect | Email | [SL from subject-line-preview-text-optimizer] | Value/miss angle | Soft — explore | None |
| T2    | 3   | Reconnect | Push  | [push copy] | Feature/content they haven't tried | Soft | None |
| T3    | 7   | Remind    | Email | [SL with proof] | Social proof, churn-reason-matched | See what's changed | None |
| T4    | 12  | Buy       | Email | [SL with soft urgency] | Incentive reveal, smaller | Claim offer | Small incentive |
| T5    | 17  | Buy       | Email + Push | [SL with deadline] | Max incentive + real deadline | Final deadline CTA | Max incentive |
| T6    | 22  | Sunset    | Email | "Should we stop emailing you?" | No pressure, permission-confirm | Stay in / Unsubscribe | None |
```

Days are defaults. Adjust cadence for annual-purchase-frequency brands (multiply by ~3).

---

### Step 4 — Review pass

After drafting, call `lifecycle-email-push-copy-reviewer` on the full sequence. It checks voice consistency, CTA strength, urgency calibration, and character-limit compliance. Incorporate its flags before presenting to the user.

Optionally call `email-compliance-auditor-gdpr-can-spam` on the sunset branch to confirm the opt-out mechanics are legally sound.

---

### Step 5 — Output the campaign pack

Present in this order:
1. **Segment summary** — lapse window, RFM tier, churn-reason tags used.
2. **Escalation logic map** — RFB Ladder with zones and offer gates.
3. **Sequence table** — all touchpoints with subject lines, body angles, CTAs, and incentives.
4. **Churn-reason routing note** — which tags were detected and what was overridden.
5. **Sunset branch spec** — trigger condition, suppression action, compliance flags.
6. **What to set up in your ESP/push platform** — enrollment trigger, exit conditions (any click → exit Zone 1, purchase → exit all), tag writes on engagement.

Save to `./win-back/[brand-slug]-win-back-campaign.md` if the user asks; inline by default.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No copy, no offer, no angle before `brand-brain` returns the active brand.
- **Hold back the discount.** The escalating-incentive logic only works if Zones 1–2 run without an offer. Don't collapse the ladder into a single coupon blast.
- **Route on churn reason.** Generic "we miss you" campaigns convert poorly. Match the angle to why they actually left.
- **Sunset is not failure.** A clean suppression is better than a tarnished sender score. Write the sunset branch with as much care as the re-activation.
- **Honest urgency only.** If Touch 5 says "expires Friday," it expires Friday. No evergreen fake deadlines.
- **Compose, don't duplicate.** Subject lines come from `subject-line-preview-text-optimizer`, push copy from `push-notification-copy-generator`, CTAs from `cta-variant-generator`. Don't rewrite those skills here.
- **Real proof or `[verify]`.** Every stat, testimonial, or result claim must be confirmed or flagged.

---

## What Not to Do

- Don't send a discount on Touch 1. You've skipped the trust-rebuilding and trained the segment to wait for coupons.
- Don't open with "We miss you" as the subject line — it's the most ignored re-engagement phrase in email marketing [verify open-rate data if citing].
- Don't build a one-size sequence for all churn reasons — at minimum split by price/value vs. passive drift; they need different angles.
- Don't skip the sunset branch — letting cold contacts linger in broadcast pools damages deliverability and sender reputation.
- Don't reimplement brand scanning or voice derivation here — that's `brand-brain`'s job.
- Don't invent a reactivation offer that doesn't exist in the brand's actual offer stack; check with `brand-brain` / `offer-pricing-brain` first.
- Don't make the sunset email sound desperate or passive-aggressive ("I guess you don't care"). Neutral and clean.

---

## Quality Checklist

- `brand-brain` called and returned the active brand before any copy was drafted?
- Lapse window and RFM tier confirmed (At-Risk / Hibernating / Lost)?
- Churn-reason tags identified and routing applied (at least one override from the routing table)?
- RFB Ladder structure intact — no incentive in Zone 1–2; escalating offer in Zone 3?
- `subject-line-preview-text-optimizer` called for email subject lines?
- `push-notification-copy-generator` called if push is in the channel stack?
- `cta-variant-generator` called for re-activation CTA?
- `lifecycle-email-push-copy-reviewer` ran as the review pass?
- Sunset branch present with exit-condition trigger, suppression action, and compliance note?
- All proof points confirmed or marked `[verify]`; no invented offers or fake urgency?
- Campaign pack saved or presented in the five-section output format?
