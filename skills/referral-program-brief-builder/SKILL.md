---
name: referral-program-brief-builder
description: >
  Takes product details and ICP notes and produces a complete referral program spec:
  incentive structure (reward type, value, one-sided vs. two-sided), mechanics (share flow,
  claim flow, fraud gates), eligibility rules, and tracking requirements — with rationale
  tied to the brand's real offer and customer economics. Applies the Viral Loop Design
  framework (Virality Coefficient → Incentive Fit → Friction Audit → Measurement Spec)
  so a junior marketer can hand the brief to engineering or a third-party referral platform
  and get a program built. Integrates brand voice, proof, and ICP from brand-brain;
  pulls customer economics from ltv-cac-payback-calculator; generates share-copy and
  CTA variants via cta-variant-generator; surfaces objections and trust signals from
  proof-vault and objection-library-builder. Use when someone says "build a referral
  program," "refer-a-friend spec," "design our advocate program," "referral incentive
  structure," "K-factor / viral loop," "customer referral brief," or asks how to turn
  customers into acquisition channels.
---

# Referral Program Brief Builder

Give it product details and ICP notes, get a complete referral program brief. Not a vague "try a double-sided reward" suggestion — a full spec covering incentive mechanics, eligibility, share flow, fraud controls, and tracking requirements that an engineer or a platform like ReferralHero, Friendbuy, or Impact can implement without a second meeting.

The framework is **Viral Loop Design**: size the virality opportunity first, then pick incentives, then cut friction, then define measurement. You can't optimize what you haven't specified.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — active brand's voice, ICP, offer, proof, banned words, and positioning. All copy and framing follows what it returns.
- **`ltv-cac-payback-calculator`** (Step 1) — customer LTV and blended CAC ground the incentive budget. Don't set reward values before knowing what a customer is worth.
- **`proof-vault`** (Step 2) — existing proof points and social-proof assets to embed in share messaging and program landing copy.
- **`objection-library-builder`** (Step 2) — known advocate objections (privacy concern, low trust in brand, reward not compelling enough) that the program design must pre-empt.
- **`cta-variant-generator`** (Step 4) — share CTAs, referral-page hero CTA, and reward-claim CTA variants.
- **`icp-persona-builder`** (Step 1, if persona depth is needed) — confirms advocate persona and what motivates sharing for this specific ICP.

---

## How a run works

```
Step 0  Load brand           ──► brand-brain (required)
Step 1  Anchor the economics ──► ltv-cac-payback-calculator → CAC ceiling for reward budget
                                 icp-persona-builder → advocate motivation profile (if needed)
Step 2  Load proof + objects ──► proof-vault + objection-library-builder
Step 3  Apply Viral Loop Design framework (core work — see below)
Step 4  Generate share copy  ──► cta-variant-generator → share CTAs + hero CTA + claim CTA
Step 5  Produce the brief    ──► structured document, saved to ./referral/[brand-slug]-referral-brief.md
```

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If none exists, ask the user for: product name + URL, ICP, core offer mechanics, LTV estimate, and 3 voice adjectives. Never produce the brief before brand context exists.

---

## Framework: Viral Loop Design

### Phase 1 — Virality Opportunity Sizing

Before picking a reward, answer: is this product referrable? Assess three gating criteria:

| Gate | Question | Threshold |
|---|---|---|
| **Trust threshold** | Do customers get enough value to credibly recommend it? | Only customers past activation / first value moment make credible advocates |
| **Social fit** | Is the product something people naturally mention in conversation or networks? | High-social-fit products (consumer, community-adjacent B2B) = larger K-factor ceiling |
| **Audience overlap** | Does the ICP's peer network likely contain more ICP members? | Low overlap → referral program is misallocated budget → flag and document |

If any gate fails, document the constraint before proceeding. A referral program built on the wrong foundation will generate unqualified signups or no signups at all.

**K-factor estimate:** K = (avg. referrals sent per advocate) × (conversion rate of referred lead). Document the assumed inputs and what K > 1 would require. K > 1 means the program is self-sustaining; target is ≥ 0.3 for meaningful program contribution at typical scale.

### Phase 2 — Incentive Structure Design

Use the **Incentive Fit Matrix** to select reward type. Wrong reward type is the single biggest referral program failure mode.

| Customer type | Reward that fits | Reward that backfires |
|---|---|---|
| Intrinsically motivated (loves brand, community ID) | Recognition, early access, status | Cash — cheapens the identity |
| Value-driven (ROI-first, measures everything) | Discount on renewal, account credit, plan upgrade | Swag or status badges |
| Network-effect product (collaboration tools, marketplaces) | Two-sided reward (both referrer + referee get value) | One-sided — referee has no reason to convert |
| Transactional (price-sensitive, low emotional attachment) | Cash or gift card | Credits usable only in-product |

**Reward mechanics to specify:**

- **Structure:** One-sided (referrer only) vs. two-sided (referrer + referee). Two-sided almost always outperforms for B2B SaaS and marketplaces; one-sided works for strong brand advocates and consumer.
- **Reward type:** Account credit / discount / cash (PayPal, Stripe) / gift card / plan upgrade / donation / status/badge. Justify against Incentive Fit Matrix.
- **Reward value:** Floor = meaningful (≥10% of monthly invoice or equivalent perceived value). Ceiling = ≤30% of blended CAC (from `ltv-cac-payback-calculator`; else `[verify CAC]`). Document the math.
- **Reward trigger:** On referee signup, on referee first payment, or on referee completing X actions. Fraud risk increases with early triggers; conversion rate decreases with late triggers. Specify the tradeoff explicitly.
- **Expiry:** Credit/discount expiry prevents balance sheet liability; no expiry increases perceived value. State the chosen policy and why.

### Phase 3 — Mechanics & Friction Audit

Referral programs die in the share flow and the claim flow. Specify both completely.

**Share flow:**
1. Entry point(s) — where in the product/customer journey does the invite prompt appear? (Post-activation, post-upgrade, NPS promoter follow-up, billing confirmation, anniversary email — pick the highest-intent trigger, not every page.)
2. Share methods — unique referral link (required), email invite (with pre-filled subject + body), social share buttons (platform-specific — LinkedIn for B2B, WhatsApp/SMS for consumer), in-product invite by email address.
3. Pre-filled message — specify the default share message (tone-matched to brand voice from `brand-brain`; share CTAs from `cta-variant-generator`). Customizable vs. locked: locked message = better fraud control; customizable = higher share rate. Specify which and why.

**Claim flow:**
1. Referee journey — what happens when the referred link is clicked? (Dedicated landing page vs. standard signup page with pre-filled coupon? Specify the recommended approach and minimum copy requirements.)
2. Reward delivery — automatic (triggered by platform) vs. manual (ops team approves). Specify the SLA for reward delivery.
3. Status visibility — advocate must be able to see referrals sent, referrals converted, and reward balance. No visibility = advocates stop sharing.

**Friction audit checklist** (flag anything blocking these):
- [ ] Unique link accessible within 2 taps/clicks from product dashboard
- [ ] Share flow works on mobile
- [ ] Referee landing page loads fast and has a clear CTA (no generic homepage redirect)
- [ ] Reward delivery is automated or has a ≤48h SLA
- [ ] Reward balance visible in-dashboard without contacting support

### Phase 4 — Eligibility & Fraud Rules

Eligibility gates separate high-value advocates from churned users and bad actors.

**Advocate eligibility (specify all):**
- Minimum tenure or usage threshold before eligibility (e.g., active for ≥30 days, completed onboarding, on paid plan). Tie to activation definition from `aha-moment-activation-metric-definer` if available.
- Plan eligibility — free users, paid users, or all? Free advocates generate cheap signups that rarely convert; paid advocates generate expensive but higher-quality referrals. Document the tradeoff.
- Geography — any excluded markets (GDPR email-invite restrictions, PayPal payout gaps)?

**Fraud controls (minimum viable set):**
- Block self-referral (same email domain, same device fingerprint, same IP within X hours).
- Minimum referee action before reward triggers (account must verify email, add payment method, or complete first purchase — not just sign up).
- Rate limit on invites per advocate per day (cap at 10–20 for early programs; adjustable).
- Manual review trigger threshold: if any single advocate generates >5% of all referred signups in a 30-day window, flag for review.

### Phase 5 — Tracking & Measurement Spec

**Required tracking events** (platform-agnostic; map to GA4 / CRM / referral platform):

| Event | Properties | Used for |
|---|---|---|
| `referral_link_generated` | advocate_id, program_slug | Baseline: how many advocates entered the funnel |
| `referral_link_shared` | advocate_id, share_method | Share rate per entry point and method |
| `referral_link_clicked` | referee_id, advocate_id | Click-through rate per share method |
| `referral_signup_completed` | referee_id, advocate_id | Referee conversion rate |
| `referral_qualified` (reward trigger) | referee_id, advocate_id, trigger_event | True referral conversion (fraud-filtered) |
| `reward_issued` | advocate_id, reward_type, reward_value | Reward delivery health |
| `reward_redeemed` | advocate_id | Reward ROI (are rewards actually valued?) |

**KPIs to track from launch:**
- K-factor (recalculate monthly against initial assumption)
- Advocate activation rate (% eligible customers who share at least once)
- Referral conversion rate (clicks → qualified referrals)
- CAC via referral vs. blended CAC (the program's unit-economics verdict)
- Reward redemption rate (proxy for incentive fit)

**Platform recommendations** (document which fits the brand's stack):
- Self-serve / SMB: ReferralHero, Viral Loops, GrowSurf
- Mid-market / API-first: Friendbuy, Impact (partnership cloud), Rewardful (for SaaS)
- Enterprise: Extole, Influitive (advocacy-first)
- DIY: custom unique-link + webhook to CRM; feasible for small programs but add fraud controls manually.

---

## Output format

Save the full brief to `./referral/[brand-slug]-referral-brief.md`. Deliver inline as well if the session is interactive. Structure:

```
# Referral Program Brief — [Brand]
Generated: [date] | Brand via brand-brain: [slug]

## Virality Assessment
[K-factor estimate, gating criteria verdicts]

## Incentive Structure
[Structure, type, value, trigger, expiry — with rationale]

## Mechanics
### Share Flow
### Claim Flow
### Friction Audit
[Checklist with pass/flag/fix per item]

## Eligibility & Fraud Rules

## Tracking Spec
[Events table + KPI targets]

## Platform Recommendation

## Share Copy & CTAs
[From cta-variant-generator — default share message, referee landing CTA, reward-claim CTA]

## Open Questions / Requires Verification
[Anything marked [verify]; items requiring legal/ops sign-off]
```

---

## Principles

- **Economics first.** Never set reward values before knowing LTV and blended CAC. A reward that exceeds 30% of CAC requires explicit sign-off — document it, don't hide it.
- **Incentive fit over incentive size.** A well-matched small reward outperforms a mismatched large one. The Incentive Fit Matrix is non-negotiable.
- **One entry point, done well.** The best referral programs have one perfectly placed trigger, not five mediocre ones. Default to the highest-intent moment; let data drive expansion.
- **Fraud controls are table stakes.** A program without fraud gates will be gamed within weeks. Specify controls before launch, not after the first incident.
- **K-factor is a hypothesis.** Publish your assumed inputs (invites per advocate, conversion rate); measure against them. Admit when the program underperforms rather than redefining success.
- **Brand voice in every share message.** The referred message is often a prospect's first brand impression. It follows brand voice from `brand-brain` exactly; no generic "check this out" templates.

## What Not to Do

- Don't design the incentive before assessing virality gating criteria. A great program on an un-referrable product wastes engineering and budget.
- Don't recommend a reward value without anchoring to LTV/CAC from `ltv-cac-payback-calculator`.
- Don't use double-sided rewards by default — they fit network-effect products and price-sensitive ICPs; they're noise for strong identity-driven brands.
- Don't skip fraud controls to ship faster. Self-referral and fake signups will invalidate the program's data within the first month.
- Don't leave share copy to the engineering team. Pre-fill it, brand-voice it, and lock or guide it.
- Don't conflate a referral program (customer advocates) with an affiliate program (paid publishers). This skill covers the former; the latter belongs in the partnership layer.
- Don't output the brief before `brand-brain` returns. Voice, proof, and ICP are not optional inputs.

## Quality Checklist

- [ ] `brand-brain` called and active brand loaded before any output?
- [ ] `ltv-cac-payback-calculator` called (or CAC acknowledged as `[verify]`) before setting reward value?
- [ ] `proof-vault` and `objection-library-builder` consulted for share-message proof and advocate objections?
- [ ] `cta-variant-generator` called for share CTA, landing CTA, and claim CTA?
- [ ] All three virality gates assessed (trust threshold, social fit, audience overlap)?
- [ ] Incentive type justified against the Incentive Fit Matrix (not just "try double-sided")?
- [ ] Reward value anchored to CAC ceiling with the math shown?
- [ ] Share flow AND claim flow fully specified (not just the reward)?
- [ ] Fraud controls specified (self-referral block, rate limit, manual-review trigger)?
- [ ] All seven required tracking events documented with properties?
- [ ] Platform recommendation made (even if "DIY with caveats")?
- [ ] All unconfirmed numbers marked `[verify]`; open items flagged for legal/ops?
- [ ] Brief saved to `./referral/[brand-slug]-referral-brief.md`?
