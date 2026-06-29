---
name: upsell-cross-sell-campaign-builder
description: >
  Customer segment + target product → multi-touch email plus in-app message campaign copy with
  segment logic, subscription/AOV hooks, and revenue-expansion CTA. Takes a defined customer
  segment (by product tier, purchase history, usage behavior, or lifecycle stage) and a target
  upgrade or add-on product, then builds a complete multi-touch campaign: segment qualification
  logic, revenue-expansion angle selection, sequenced emails and in-app messages, and CTAs sized
  to the right commitment level. Calls brand-brain for voice/ICP/offer context, offer-pricing-brain
  for tier mechanics, proof-vault for social proof and outcome data, cta-variant-generator for
  per-touchpoint CTAs, and subject-line-preview-text-optimizer before finalizing email subject
  lines. Use when the user says "upsell campaign," "cross-sell sequence," "drive upgrades,"
  "expansion revenue," "move customers to the next tier," "promote this add-on to existing users,"
  "AOV campaign," "expansion MRR," or hands over a customer segment and asks for retention/growth
  messaging.
---

# Upsell & Cross-Sell Campaign Builder

Revenue expansion from existing customers is the highest-ROI motion in SaaS and eCommerce. This skill builds multi-touch email + in-app campaigns that move the right customers toward the right upgrade at the right moment — using the brand's real offer mechanics, real proof, and a commitment ladder calibrated to customers who already trust the product.

The working model used here is **ACRA** (our own house mnemonic, not an established framework): **Anchor** (reinforce existing value), **Create a Gap** (surface the limit they're hitting), **Resolve the Gap** (show the upgrade closes it), **Act** (CTA sized to the evidence). Every touchpoint earns its place on this ladder — no cold pitching inside a relationship.

---

## Skills this calls

- **`brand-brain`** (required, always first) — voice, ICP, banned words, offer mechanics, proof, positioning.
- **`offer-pricing-brain`** (required) — tier structure, upgrade paths, AOV mechanics, guarantee/trial terms.
- **`proof-vault`** (required) — customer outcomes, upgrade-specific social proof, revenue/ROI data for the target tier.
- **`cta-variant-generator`** (required per touchpoint) — CTA labels + friction-reducer microcopy sized to each email's awareness level.
- **`subject-line-preview-text-optimizer`** (required before final delivery) — subject + preview text for every email.
- **`usage-triggered-message-sequencer`** *(optional)* — if campaign is behavior-triggered (usage threshold, feature not adopted), call this to define trigger conditions and delays.
- **`in-app-microcopy-writer-auditor`** *(optional)* — if in-app banners or tooltips are part of the mix.
- **`lifecycle-journey-mapper`** *(optional)* — if the full expansion journey needs mapping before sequencing.
- **`a-b-multivariate-test-designer`** *(optional)* — to design subject-line or CTA A/B tests on the sequence.

---

## How a run works

```
Step 0  Load brand context    ──► brand-brain (voice, ICP, offer, proof)
Step 1  Load offer mechanics  ──► offer-pricing-brain (tiers, upgrade paths, AOV levers)
Step 2  Load proof            ──► proof-vault (upgrade outcomes, tier-specific social proof)
Step 3  Qualify the segment   ──► define who qualifies, who is excluded, and why
Step 4  Select the angle      ──► pick the primary expansion angle using ACRA + angle matrix
Step 5  Build the sequence    ──► email + in-app scaffold (3–5 touches, roles, delays)
Step 6  Write the copy        ──► full copy per touch: subject, preview, body, CTA
Step 7  Optimize surfaces      ──► subject-line-preview-text-optimizer, cta-variant-generator
Step 8  Self-review + output
```

---

## Step 0 — Load brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`) before writing a single word of copy. It returns the active brand's digest: voice adjectives, banned words, ICP + awareness tendency, offer mechanics + destination URLs, and real proof. Obey the voice and banned-words as hard overrides; use only the returned real proof (mark any unconfirmed claim `[verify]`).

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If none exists, ask the user to install `brand-brain` (preferred) or answer a 4-question mini-setup (product + ICP + offer mechanics + voice adjectives + banned words), then proceed. Prefer the call.

---

## Step 1–2 — Offer mechanics and proof

**`offer-pricing-brain`:** resolve the upgrade path — what the customer currently has, what the target tier adds, the price delta, whether a trial or guarantee removes friction. If not installed, ask the user for tier names, feature delta, and price.

**`proof-vault`:** pull upgrade-specific outcomes — time-to-value after upgrade, revenue or usage lift, named customers if approved. Mark anything unconfirmed `[verify]`. Thin proof is a campaign risk; flag it rather than fabricate.

---

## Step 3 — Segment qualification

Before writing copy, define the segment precisely. Without this, personalisation is cosmetic.

```
Segment definition
  Who qualifies  : [tier / product / purchase history / usage signal]
  Who is excluded: [already on target tier; churned; trial-only; under X days]
  Segment size   : [verify with CRM/analytics owner]
  Entry trigger  : [time-based | usage-threshold | behavior-event | manual]
  Exit conditions: [converted | unsubscribed | hard-bounced | disqualified by upgrade]
```

If the user has not defined the segment, ask two questions only: (1) what product/tier do they currently have, and (2) what event or attribute flags them as ready for this offer.

---

## Step 4 — Expansion angle selection (ACRA + angle matrix)

Match the primary angle to the segment's evidence. Use the matrix; pick one primary angle and one supporting angle per touchpoint. Never stack more than two angles in a single message.

| Expansion angle | Best evidence signal | Worst use |
|---|---|---|
| **Capacity / limit hit** | Usage near or at tier ceiling | Segment far from limits |
| **Feature gap** | High usage of a feature gated to next tier | Segment hasn't touched adjacent feature |
| **Outcome uplift** | Upgrade proof shows measurable improvement | No outcome data exists |
| **Peer comparison** | Cohort data — similar customers on higher tier | Unverified benchmarks |
| **Risk / downside of staying** | Documented business cost of current tier | No cost evidence |
| **New capability unlock** | Product launch or recently-added tier feature | Not relevant to segment's job |
| **Seasonal / milestone** | Anniversary, renewal window, business season | Forced/fake urgency |

If no angle has solid evidence, flag it before writing copy. A campaign with no real hook is noise.

---

## Step 5 — Sequence scaffold

Default: **3-touch email + 1 in-app message**. Extend to 5 touches only if the price point or complexity warrants it (high-ACV upgrade, multi-stakeholder buy). Never exceed 5 touches in a single window.

```
Touch 1 — Anchor (Email)
  Role: Reinforce existing value + open the gap. Soft. No price yet.
  Delay from trigger: D+0 or D+1
  Commitment ceiling: low-medium (awareness, curiosity)

Touch 2 — Gap + Resolution (Email)
  Role: Name the specific limit the customer is hitting; show the upgrade closes it.
  Use proof from proof-vault here (outcome + social proof).
  Delay: D+3–5
  Commitment ceiling: medium (evaluation)

Touch 3 — In-App Message
  Role: Contextual, in-moment reinforcement when the customer hits the exact limit.
  Trigger: usage event (threshold hit, feature-gated action attempted).
  Short. One CTA. No nurture copy.

Touch 4 — Direct Offer (Email)
  Role: Full offer — tier name, feature delta, price, guarantee/trial. Primary buy/upgrade CTA.
  Delay: D+7–10 from Touch 1
  Commitment ceiling: high (transaction)

Touch 5 — Last-Call (Email, optional)
  Role: Genuine urgency only (renewal window, seat limit, price change). No fake scarcity.
  Delay: D+14 from Touch 1 or 3 days before a real deadline
  Only include if a real constraint exists.
```

Provide exact placeholder tokens for personalization fields: `{{first_name}}`, `{{current_plan}}`, `{{target_plan}}`, `{{usage_metric}}`, `{{upgrade_url}}`. Do not fabricate dynamic field values.

---

## Step 6 — Copy per touchpoint

Write each touch in full. Structure per email:

```
Subject line          : [primary] / [variant B — see step 7]
Preview text          : [complement — adds info, does not repeat subject]
From name             : [brand or named sender if relevant to ICP]

Opening               : personal acknowledgment of existing use (one sentence, no flattery)
Body                  : ACRA progression — anchor, gap, resolution, act
  - Anchor  : 1–2 sentences referencing what they already have/do
  - Gap     : specific limit or friction (evidence-backed or [verify])
  - Resolve : how the upgrade closes it (feature delta + proof)
  - Act     : transition to CTA
CTA block             : [from cta-variant-generator — button label + microcopy]
Opt-down line         : "Not the right time? [Stay on {{current_plan}}]" — gives an exit that isn't unsubscribe
```

For the in-app message:

```
Trigger event         : [specific event name in your product analytics]
Message type          : banner / modal / tooltip
Headline              : [15 words max — name the gap or the unlock]
Body                  : [2 sentences max]
CTA                   : [single action — "Upgrade now" or equivalent per brand voice]
Dismiss               : [always include — "Maybe later"]
```

---

## Step 7 — Surface optimization

Before finalizing:

1. **Call `subject-line-preview-text-optimizer`** on every email subject + preview pair. Apply recommendations. At minimum deliver two subject-line variants per email so the sender can A/B or choose.
2. **Call `cta-variant-generator`** for each email's CTA block (button label + microcopy). Pass the awareness level of each touch; the tool will size the commitment ask correctly.

---

## Output format

Deliver to `./campaigns/upsell-[segment-slug]-[target-product-slug]/`:

```
campaign-brief.md       — segment definition, angle rationale, sequence map, KPIs
sequence/
  touch-1-anchor.md     — subject variants, preview, body, CTA block
  touch-2-gap.md
  touch-3-in-app.md
  touch-4-offer.md
  touch-5-lastcall.md   — only if a real deadline exists
```

Inline delivery is fine for quick requests; always offer to save when a full sequence is produced.

---

## Principles

- **Existing trust is the asset.** Customers already believe the product works. Upsell copy assumes competence, not skepticism — open with what they've achieved, not what they're missing.
- **One angle per touch.** Do not cram gap + resolution + urgency into a single email. Each touch moves one step on the ACRA ladder.
- **Commitment matches the touch.** Touch 1 asks for attention. Touch 4 asks for a transaction. Never flip the order.
- **Real proof or nothing.** A fabricated upgrade outcome destroys more trust than no proof at all. Use only `proof-vault` output; mark anything unconfirmed `[verify]`.
- **Genuine urgency only.** No fake countdown timers, no invented seat scarcity. If there is no real deadline, do not manufacture one. Include Touch 5 only when a genuine constraint exists.
- **Opt-down, not unsubscribe.** Every email gives the customer a way to say "not now" without opting out entirely. This preserves the relationship for a future window.
- **Segment precision is copy quality.** Vague segments produce vague copy. Refuse to write before the segment qualifier is defined.

---

## What not to do

- Do not start writing copy before `brand-brain` returns the active brand. Voice and banned-words are non-negotiable.
- Do not invent tier names, pricing deltas, or feature differences — pull from `offer-pricing-brain` or ask the user directly.
- Do not write a Touch 5 last-call without a real, documented deadline or constraint.
- Do not address the customer as if they are a cold prospect — they have a product relationship; acknowledge it.
- Do not stack more than two persuasion angles in a single message.
- Do not skip `subject-line-preview-text-optimizer` — subject lines written without that review often underperform on open rate, which gates everything else.
- Do not use countdown urgency language, scarcity claims, or social-proof statistics unless `proof-vault` or the brand brief confirms them.

---

## Quality checklist (self-review before presenting)

- [ ] `brand-brain` called first; voice + banned-words honored throughout; only real proof used (`[verify]` on anything unconfirmed)?
- [ ] `offer-pricing-brain` sourced tier names, feature delta, and price — nothing invented?
- [ ] `proof-vault` output referenced for upgrade outcomes; thin proof flagged if absent?
- [ ] Segment defined with qualification logic, exclusion criteria, entry trigger, and exit conditions?
- [ ] ACRA angle selected from the matrix with evidence backing; no angle unsupported by data?
- [ ] Sequence scaffold correct — Anchor → Gap/Resolution → In-App → Direct Offer → Last-Call (only if real deadline)?
- [ ] Each touch has one primary angle; commitment ceiling matches the touch position?
- [ ] `cta-variant-generator` called per touchpoint; commitment level passed as context?
- [ ] `subject-line-preview-text-optimizer` called on all email subject/preview pairs; two variants per email?
- [ ] Opt-down line present in every email; in-app message has a dismiss action?
- [ ] Output saved (or offered to save) to `./campaigns/upsell-[segment-slug]-[target-product-slug]/`?
