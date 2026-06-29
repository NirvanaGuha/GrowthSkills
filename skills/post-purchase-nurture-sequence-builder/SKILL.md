---
name: post-purchase-nurture-sequence-builder
description: >
  Converts order data and an upsell/cross-sell catalog into a complete, sequenced post-purchase
  email nurture program — covering the thank-you, review ask, cross-sell, and loyalty nudge
  touchpoints. Built around a customer-lifecycle ladder (our working model): every email
  in the sequence is engineered to advance the customer one rung — from first-buyer to
  repeat-buyer to advocate — rather than treating post-purchase as an afterthought. Calls
  brand-brain for voice and proof, subject-line-preview-text-optimizer for subject lines,
  push-notification-copy-generator for parallel push variants, cta-variant-generator for
  email CTAs, and lifecycle-email-push-copy-reviewer to audit the finished draft. Produces a
  ready-to-import sequence spec (copy, timing, segmentation logic, subject lines, CTAs, and
  optional push variants) plus a send-cadence rationale. Use whenever the user says
  "post-purchase emails," "thank-you series," "review ask sequence," "cross-sell flow after
  order," "loyalty drip," "nurture new customers," "first-order follow-up," "increase repeat
  purchase rate," or hands over an order schema and asks what to send next.
---

# Post-Purchase Nurture Sequence Builder

The sale is not the finish line. The email program that runs after checkout determines whether a
buyer becomes a repeater, a referrer, or a one-and-done. This skill builds a sequenced
post-purchase nurture program grounded in our Lifecycle Ladder — a working model we use here for
moving customers from first purchase through repeat purchase to active advocacy, one intentional
touchpoint at a time.

Every sequence is in the brand's real voice, uses real proof, and is sized to the upsell catalog
at hand. Siblings handle the component craft so the output here stays integrated and coherent
rather than cobbled together.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — resolves voice, ICP, banned words, offer mechanics, and
  real proof for the active brand. Do not write a single email until it returns.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
  brand's `brand.md` directly; if none exists, ask the user for brand voice adjectives, banned
  words, ICP description, and the primary offer/guarantee before proceeding.
- **`subject-line-preview-text-optimizer`** — call once per email to finalize subject + preview
  text (2–3 variants each); synthesize inline only if unavailable.
- **`cta-variant-generator`** — call for each email's primary CTA button; pass placement=email and
  awareness stage; synthesize inline only if unavailable.
- **`push-notification-copy-generator`** — call to produce parallel push variants for emails
  2 and 3 (review ask + cross-sell) when the brand uses push; skip if not applicable.
- **`proof-vault`** — pull the right proof points per email stage (social proof for review ask,
  outcome proof for cross-sell); synthesize from brand-brain digest if unavailable.
- **`lifecycle-email-push-copy-reviewer`** — call as a final review pass on the full drafted
  sequence before delivering; flag any brand-voice, CTA, urgency, or character-limit issues.

---

## How a run works

```
Step 0  Load brand (brand-brain) — no output until brand is resolved
Step 1  Gather inputs — order schema, upsell catalog, cadence constraints
Step 2  Map the Lifecycle Ladder — assign one rung objective to each email
Step 3  Draft each email — delegate subject lines, CTAs, push variants to siblings
Step 4  Review pass — call lifecycle-email-push-copy-reviewer on the full draft
Step 5  Deliver — sequence spec + send-cadence rationale + optional push variants
```

---

## Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`), passing the user's request and
any named brand. It returns voice adjectives, banned words, offer mechanics, real proof, ICP, and
the path to `brand.md`. Do not write any email copy until it returns.

Obey the returned voice and banned-words as hard overrides. Mark any proof not in the brand digest
as `[verify]`. Use the offer mechanics to anchor the cross-sell and loyalty emails to real,
linkable destinations.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
brand's `brand.md` directly; if none exists, ask the user for brand voice adjectives, banned words,
ICP description, and the primary offer/guarantee before proceeding.

---

## Step 1 — Gather inputs

Collect (ask only what is missing; infer what you can):

| Input | Why it matters |
|---|---|
| Order schema or example order | Confirms what fields are merge-tag-able (product name, price, delivery ETA) |
| Upsell / cross-sell catalog | Product names, URLs, price delta from first order; used in Email 3 |
| Existing customers' repeat-purchase rate (`[verify]` if unavailable) | Sets the urgency calibration for the loyalty nudge |
| Email platform (Klaviyo, Drip, HubSpot, etc.) | Governs character limits, merge-tag syntax, conditional block support |
| Push enabled? (yes/no) | Whether to produce parallel push variants |
| Any send-time or quiet-hours constraints | Informs cadence table |

Do not ask for brand voice, ICP, proof, or offer — `brand-brain` owns those.

---

## The Lifecycle Ladder (our working model)

Each email maps to one rung. A rung is not a theme — it is a specific job the email does to
advance the customer relationship. Never skip a rung; never assign two jobs to one email.

```
Rung 1 — Confirm & Reassure      ► Email 1 (Thank-You)
Rung 2 — Activate Social Proof   ► Email 2 (Review Ask)
Rung 3 — Expand the Relationship ► Email 3 (Cross-Sell / Upsell)
Rung 4 — Lock in Loyalty         ► Email 4 (Loyalty Nudge)
```

Optional Rung 0 (pre-delivery): a shipping-confirmation email with delivery context and a
soft "while you wait" recommendation. Include only if the brand has a meaningful lead time
(>2 days) and the platform supports order-status triggers.

---

## Email-by-email spec

### Email 1 — Thank-You (Rung 1: Confirm & Reassure)

**Timing:** send immediately on order confirmation trigger, or within 15 minutes.

**One job:** make the customer feel the purchase was the right call. Kill buyer's remorse before
it starts. No selling here — any attempt to upsell in the thank-you email trains the customer to
distrust the brand's gratitude.

**Content blueprint:**
- Subject: genuine thanks + order confirmation signal (not promotional; avoids promo tab).
- Body: confirm the order (product name, order number via merge tag), set delivery expectations,
  one proof point that reinforces the purchase decision (top review, award, guarantee).
- CTA: low-commitment — "Track my order" or "See what's coming" (no buy-more CTA).
- Microcopy: support contact or easy-return reassurance (real policy; mark `[verify]` if unknown).

Delegate subject line variants to `subject-line-preview-text-optimizer` (tone: warm/confirmatory,
not promotional). Delegate CTA to `cta-variant-generator` (awareness: most-aware, placement:
email, commitment: low — tracking/account action).

### Email 2 — Review Ask (Rung 2: Activate Social Proof)

**Timing:** 5–7 days post-delivery (or 7–9 days post-order if no delivery data). Research
benchmark: review request response rate peaks at 5 days post-delivery [verify exact figure per
platform].

**One job:** turn a happy customer into a proof source. This compounds — each review reduces CAC
for the next buyer.

**Content blueprint:**
- Subject: personal ask framing (from a person, not "the brand") — brief, curious, not begging.
- Body: one sentence on why their experience matters; link directly to the review platform (G2,
  Capterra, Google, Trustpilot, or app store — pull from brand's known platforms). No fluff.
- Incentive: include only if brand's review policy allows it and the platform's ToS permits
  incentivized reviews. Mark any incentive as `[verify]` until confirmed.
- CTA: "Leave a quick review" (≤5 words on the button). One CTA only.

Delegate subject line to `subject-line-preview-text-optimizer` (tone: personal, low-pressure).
Delegate CTA to `cta-variant-generator` (awareness: most-aware, commitment: medium).
Optionally produce parallel push variant via `push-notification-copy-generator` (≤40 chars title,
≤90 chars body, one CTA URL).

Pull relevant social-proof context (outcome stats, aggregate rating) from `proof-vault` or
brand-brain digest.

### Email 3 — Cross-Sell / Upsell (Rung 3: Expand the Relationship)

**Timing:** 10–14 days post-order, after the review window has closed. Do not cross-sell before
the review ask — it poisons the gratitude signal.

**One job:** show the next logical product or tier that makes the first purchase more valuable.
This is not a catalog blast. It is one curated recommendation with a clear "why this, why now"
anchor tied to what they already bought.

**Content blueprint:**
- Subject: product benefit or outcome framing; no "you might also like" language.
- Opening: bridge from the first purchase ("You've been using [Product] for X days…" or
  "Customers who bought [Product] often find the next step is…").
- Recommendation: one primary cross-sell or upsell from the catalog. Show price delta, not just
  price. Name the use-case gap it fills.
- Social proof: one outcome-focused review or stat from `proof-vault` for the recommended product.
- CTA: "Add [Product Name]" or "See how it works" — product-specific, not generic.
- Fallback if catalog is thin: use a "bundle discount" or "refill reminder" mechanic instead.

Delegate subject line to `subject-line-preview-text-optimizer` (tone: benefit-forward).
Delegate CTA to `cta-variant-generator` (awareness: product-aware, placement: email, commitment:
high — direct product add/buy).
Optionally produce parallel push variant via `push-notification-copy-generator`.

### Email 4 — Loyalty Nudge (Rung 4: Lock in Loyalty)

**Timing:** 21–30 days post-order (before the typical repeat-purchase window closes for the
category). Adjust to category purchase cycle if known.

**One job:** make the second purchase feel like the start of an identity, not just another
transaction. Reference the first order by name. Signal that the brand remembers and rewards
loyalty — but only with a real mechanic (loyalty tier, VIP status, discount, exclusive access).
Do not invent a program that does not exist.

**Content blueprint:**
- Subject: milestone or identity framing ("You've officially joined the [tier/group]" or
  "Your next order ships differently").
- Body: acknowledge the first purchase; introduce the loyalty mechanic (real URL + real terms);
  if no program exists, use a simple "returning customer" offer with real coupon code.
- CTA: "Claim my [benefit]" or "Shop as a [tier name]" — identity-anchored, not transactional.
- If no loyalty program exists: flag this gap to the user; offer a simple coupon mechanic as a
  placeholder, and note that `upsell-cross-sell-campaign-builder` can build a fuller program.

Delegate subject line to `subject-line-preview-text-optimizer` (tone: celebratory/exclusive).
Delegate CTA to `cta-variant-generator` (awareness: most-aware, commitment: high).

---

## Cadence table (deliver with the sequence)

| Email | Trigger | Send window | Rung | Push variant? |
|---|---|---|---|---|
| 0 (optional) | Shipment confirmed | Immediately | Pre-rung | No |
| 1 | Order confirmed | ≤15 min | Confirm & Reassure | Optional |
| 2 | 5–7 d post-delivery | — | Activate Social Proof | Yes |
| 3 | 10–14 d post-order | — | Expand Relationship | Yes |
| 4 | 21–30 d post-order | — | Lock in Loyalty | No |

Include merge-tag placeholders in the delivered spec using the platform's actual syntax (e.g.,
`{{ order.name }}` for Klaviyo, `{{contact.first_name}}` for HubSpot). Flag if platform is unknown.

---

## Step 4 — Review pass

After drafting all emails, call `lifecycle-email-push-copy-reviewer` on the full sequence. Pass
the full draft as input. Incorporate flagged issues (voice violations, weak CTAs, character-limit
overruns, missing unsubscribe hooks) before delivering. Note any issues the reviewer flagged that
were intentionally preserved, and why.

---

## Output format

Deliver as a single document saved to `./lifecycle/post-purchase-sequence-[brand-slug].md`
(create `./lifecycle/` if absent). Structure:

```
# Post-Purchase Nurture Sequence — [Brand]
Generated: [date] | Platform: [ESP] | Push: [yes/no]

## Cadence table
## Email 1 — Thank-You
  Subject variants (from subject-line-preview-text-optimizer)
  Preview text variants
  Body copy
  CTA (from cta-variant-generator)
  Microcopy
  Merge tags used
## Email 2 — Review Ask
  [same structure]
  Push variant (if applicable)
## Email 3 — Cross-Sell
  [same structure]
  Push variant (if applicable)
## Email 4 — Loyalty Nudge
  [same structure]
## Reviewer notes (from lifecycle-email-push-copy-reviewer)
## Open gaps and [verify] items
```

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No email copy before brand-brain returns the active brand.
- **One job per email, one rung at a time.** Mixing the review ask with a cross-sell undercuts both.
- **Never invent a loyalty program or incentive.** If none exists, say so and offer a placeholder with a gap note.
- **Real proof or `[verify]`.** Delivery time estimates, review-response benchmarks, repeat-purchase rates — cite or flag every number.
- **Compose, don't duplicate.** Subject lines, CTAs, push variants, and the review pass all live in sibling skills; call them.
- **Timing is a design decision.** Deliver the cadence rationale alongside the copy — the "when" is half the sequence.

---

## What not to do

- Don't upsell in the thank-you email. Rung 1 is for reassurance only.
- Don't send the review ask before delivery is confirmed or before 5 days post-delivery.
- Don't cross-sell before the review window closes (keeps the ask clean).
- Don't fabricate a loyalty tier, VIP program, or discount that the brand hasn't confirmed.
- Don't reimplement brand scanning, subject-line craft, CTA batteries, or push copy — call the siblings.
- Don't write generic nurture copy ("We value your business") — every email references the actual product, actual order, and actual next step.
- Don't deliver without running the reviewer pass.

---

## Quality checklist (self-review before delivering)

- [ ] `brand-brain` called and brand loaded (or fallback applied) before any copy was written?
- [ ] Voice and banned-words honored throughout all four emails?
- [ ] Each email assigned exactly one Lifecycle Ladder rung with no rung overlap?
- [ ] Cadence table delivered with trigger logic and platform merge-tag syntax?
- [ ] Subject lines and preview text produced via `subject-line-preview-text-optimizer`?
- [ ] CTAs produced via `cta-variant-generator` with correct awareness stage and placement?
- [ ] Push variants produced via `push-notification-copy-generator` for Emails 2–3 (if applicable)?
- [ ] All unconfirmed figures (delivery benchmarks, incentive policies, repeat-purchase rates) marked `[verify]`?
- [ ] No invented loyalty programs, incentives, or proof points?
- [ ] `lifecycle-email-push-copy-reviewer` called on the full draft and its flags addressed?
- [ ] Output saved to `./lifecycle/post-purchase-sequence-[brand-slug].md`?
