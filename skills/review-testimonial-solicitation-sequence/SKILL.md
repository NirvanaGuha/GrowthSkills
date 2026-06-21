---
name: review-testimonial-solicitation-sequence
description: >
  Turns satisfied customers into a steady stream of G2, Capterra, Google, and app-store reviews —
  and pulls quote-ready testimonials for sales and content. Input a customer segment (recently
  activated, post-support-win, power-user, referrer, churned-and-returned), their outcome or
  milestone, and the target platform(s); output a personalized 2–3 touchpoint email or in-app
  sequence with direct review links, value-exchange framing, and a timing/trigger spec. Built on
  the Ask-at-the-Peak framework so requests land at the moment of maximum satisfaction, not at
  some arbitrary post-close interval. Composes `proof-vault`, `cta-variant-generator`,
  `subject-line-preview-text-optimizer`, `email-compliance-auditor-gdpr-can-spam`, and
  `lifecycle-email-push-copy-reviewer` rather than reimplementing their logic. Saves the finished
  sequence to `./advocacy/[segment-slug]-review-sequence.md`. Use when the user says "get more
  reviews," "ask customers for testimonials," "G2 review campaign," "Capterra reviews," "star
  ratings," "social proof sequence," "customer advocacy email," or hands over a customer segment
  and asks for a review ask.
---

# Review & Testimonial Solicitation Sequence

Most review asks fail for one of three reasons: wrong timing (too early, too late, or calendar-blind), wrong framing (asks for a favor, not a value exchange), or wrong channel (email when in-app would have been 4x higher). This skill fixes all three. It requests at peak satisfaction, offers a clear value exchange, and matches the delivery surface to where the customer already is.

The output is a production-ready 2–3 touchpoint sequence — email, in-app, or both — with direct deep-links, on-brand copy, compliance clearance, and a stated A/B hypothesis the sender can test on send #1.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads voice, banned words, ICP, offer, and proof so every message sounds like the brand. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP description, product outcome language, voice adjectives, and banned words before proceeding.
- **`proof-vault`** — pull confirmed customer outcomes and quote fragments to anchor the ask in social proof; synthesize inline when absent.
- **`cta-variant-generator`** — generate the primary "Leave a review" button copy and 1–2 alternates at the right commitment level for each touchpoint; synthesize inline when absent.
- **`subject-line-preview-text-optimizer`** — optimize the subject line and preview text for each email touchpoint; synthesize inline when absent.
- **`email-compliance-auditor-gdpr-can-spam`** — gate the sequence before delivery: CAN-SPAM physical address, one-click unsubscribe, no deceptive subjects, GDPR lawful-basis note if region is EU/UK; synthesize inline when absent.
- **`lifecycle-email-push-copy-reviewer`** — final review pass on voice, flow, CTA strength, and char-limit compliance; synthesize inline when absent.

---

## How a run works

```
Step 0  Load the brand               ──► brand-brain (required)
Step 1  Intake: segment + outcome    ──► 5 questions, 2 minutes
Step 2  Audit proof inventory        ──► proof-vault
Step 3  Design the ask-peak trigger  ──► Ask-at-the-Peak framework
Step 4  Write the sequence           ──► 2–3 touchpoints, full copy
Step 5  Optimize subjects + CTAs     ──► subject-line-preview-text-optimizer + cta-variant-generator
Step 6  Compliance gate              ──► email-compliance-auditor-gdpr-can-spam
Step 7  Editorial review             ──► lifecycle-email-push-copy-reviewer
Step 8  Save + present               ──► ./advocacy/[segment-slug]-review-sequence.md
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. It returns the active brand's digest — voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning, ICP + awareness tendency — and the path to `brand.md`. Do not write any copy until it returns.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP description, product outcome language, voice adjectives, and banned words before proceeding.

### Step 1 — Intake

Ask only what's missing. Required inputs:

| Input | Why it matters |
|---|---|
| **Customer segment** | Determines the ask-peak trigger and personalization angle |
| **Outcome / milestone** | The specific win that creates maximum satisfaction to leverage |
| **Target platform(s)** | G2, Capterra, Google Business, App Store, Play Store, Trustpilot, or "testimonial only" |
| **Delivery channel** | Email, in-app tooltip/modal, push notification, or mixed |
| **Sending region** | EU/UK → GDPR lawful-basis note; US → CAN-SPAM; global → both |

If the user provides all five upfront, skip the questions and proceed.

---

## Ask-at-the-Peak framework

The framework has three components. Run all three before writing a single line of copy.

### 1. Identify the peak moment (segment map)

Map the customer segment to its natural satisfaction peak. Ask at the wrong moment and response rates drop by 60%+ [verify exact rate; directionally supported by Trustpilot and G2 vendor data].

| Segment | Peak moment | Trigger signal |
|---|---|---|
| **Recently activated** | First meaningful outcome (Aha moment) | Activation event (e.g., "first campaign sent", "first 100 subscribers") |
| **Power user / high-usage** | After a usage streak or milestone (30-day / feature milestone) | Usage event threshold crossed |
| **Post-support-win** | Within 24h of a ticket marked resolved (CSAT ≥ 4) | Ticket close + satisfaction score |
| **Referrer / advocate** | After a successful referral converts | Referral conversion event |
| **Churned-and-returned** | 7–14 days after successful reactivation | Return + first value event |
| **Renewal / expansion** | Within 48h of upgrade or renewal | Billing event (upgrade/renewal) |

### 2. Value-exchange framing (not a favor ask)

Customers do not owe reviews. Every ask must answer "why should I?" from their perspective:

- **Reciprocity:** they got value from the product; a review helps others in the same situation find it.
- **Community identity:** "your experience helps [ICP descriptor] like you cut through the noise."
- **Tangible exchange:** platform incentive disclosure where the platform allows it (G2 gift card programs, etc. — confirm platform's current policy before writing).
- **Micro-commitment ladder:** email → landing page → review form. Never start at the review form.

### 3. Platform deep-link spec

Each touchpoint must include a direct link that lands the reviewer as close to the review field as possible. Avoid home-page links — each extra click costs ~15–20% completion [verify; consistent with CRO benchmark literature].

| Platform | Deep-link format |
|---|---|
| **G2** | `https://www.g2.com/products/[your-slug]/reviews#reviews` — or vendor-provided review link from G2 admin |
| **Capterra** | `https://reviews.capterra.com/new/[your-id]` — from Capterra vendor portal |
| **Google Business** | `https://g.page/r/[CID]/review` — from Google Business Profile → "Get more reviews" |
| **App Store (iOS)** | `itms-apps://itunes.apple.com/app/id[APPID]?action=write-review` |
| **Google Play** | `https://play.google.com/store/apps/details?id=[package]&reviewId=0` |
| **Trustpilot** | `https://www.trustpilot.com/evaluate/[your-domain]` |

**Collect the actual links from the user in Step 1** — never fabricate an ID. Placeholder: `[REVIEW-LINK: confirm in platform admin]`.

---

## Sequence architecture

### 2-touchpoint (default, B2C / low-friction)

| Touch | Timing | Channel | Goal |
|---|---|---|---|
| **T1: The ask** | At the peak trigger | Email or in-app | Earn the click to the review page |
| **T2: The nudge** | +5–7 days if no click | Email | Soft reminder with a new angle |

### 3-touchpoint (recommended for B2B / high-value segment)

| Touch | Timing | Channel | Goal |
|---|---|---|---|
| **T1: The prime** | 24–48h before the ask | Email or in-app tooltip | Warm up: share their outcome back to them |
| **T2: The ask** | At the peak trigger | Email | The direct request with deep-link |
| **T3: The nudge** | +5–7 days if no click | Email | Alternate angle + one more shot |

For in-app delivery: replace T1 with a tooltip surfaced at the trigger event, T2 with a modal (max 1 per session), no T3 in-app (use email instead to avoid fatigue).

---

## Copy craft rules

**T1 / Prime (warm-up):** Reference the customer's specific outcome by name. Do not ask for anything. Mirror their language from onboarding or support notes where available. One sentence max on what happens next. Subject: curiosity/value, not transactional.

**T2 / Ask:** Open with the outcome, one sentence. Make the value exchange explicit. One primary CTA button (3–7 words) + one low-friction text-link alternative ("takes 2 minutes"). Include the deep-link. Include a trust line: "Your honest experience — positive or constructive — helps [ICP descriptor] make confident decisions." Never offer incentives on platforms that prohibit them (G2 allows; Google prohibits; check Capterra's current policy).

**T3 / Nudge:** Change the angle — not a repeat, not just a reminder. Options: social-proof angle ("X customers from [industry] already shared theirs"), urgency angle ("we're within reach of [milestone] reviews on G2"), or reciprocity angle (share something new of value, then re-ask once). Hard stop: never send a fourth touch for the same ask in the same window.

**Testimonial-only flow:** When the user wants a quote-pull rather than a platform review, add a 2-question reply mechanism: (1) "In one or two sentences, what's the specific outcome you've gotten?" (2) "Can we use your words in our marketing with your name and role?" This is simpler legally than a full review and faster for the customer.

---

## Compliance notes (non-exhaustive — always run the auditor)

- **CAN-SPAM (US):** physical mailing address in footer; clear sender identity; functional unsubscribe; no deceptive subject lines. Note: review-solicitation emails are commercial messages and fall under CAN-SPAM.
- **GDPR/UK GDPR:** lawful basis for contacting (legitimate interest or contract performance is usually sufficient for existing customers; document it); honor opt-outs within 1 month.
- **FTC Endorsement Guides (16 CFR Part 255, updated 2023):** if offering incentives (gift cards, discounts) in exchange for reviews, the review must disclose the material connection. No conditioning of the incentive on a positive review. G2's Incentivized Review program satisfies this via their disclosure widget — confirm current platform mechanics.
- **Google's Prohibited Practices:** Google explicitly prohibits review gating (asking for a review only if the customer indicates they'll leave a positive one) and incentivized reviews. Sequence must solicit honest feedback unconditionally.
- **App Store / Play Store:** both prohibit incentivizing ratings. In-app prompts must use the native `SKStoreReviewRequest` (iOS) / `ReviewManager` (Android) API to avoid rejection.

---

## Principles

- **Ask-at-the-Peak only.** Wrong timing is the #1 failure mode. Identify the trigger event before writing copy; don't default to "30 days after close."
- **Value exchange, not a favor.** Every ask answers "why should I?" before asking for anything.
- **Deep-link or don't ask.** An extra navigation step costs completion. Collect real links from the user; never fabricate IDs.
- **Angle-rotate on the nudge.** A repeated ask is spam. T3 must offer something new — a social-proof hook, a milestone, new value.
- **Compliance before send.** Always run `email-compliance-auditor-gdpr-can-spam` before declaring the sequence ready.
- **Brand-brain first.** No copy before brand context is loaded. Its voice + banned-words override everything here.
- **Honest-feedback framing.** Copy must solicit honest reviews unconditionally — never gate, never condition, never imply a positive review is required.

---

## What not to do

- Don't send more than 3 touches for the same ask in the same 30-day window.
- Don't start the sequence with a platform deep-link (cold click kills conversion — warm first).
- Don't fabricate review-platform IDs or deep-link URLs; use `[REVIEW-LINK: confirm in platform admin]` as a placeholder.
- Don't offer incentives on platforms that prohibit them (Google, App Store, Play Store).
- Don't use review gating language ("If you had a good experience, we'd love a review") — this violates Google's guidelines and the FTC's endorsement rules.
- Don't reuse the same angle on T3; a copy-paste reminder with "just following up" is the lowest-ROI sequence move in customer marketing.
- Don't skip the compliance gate before marking the sequence done.
- Don't write copy before `brand-brain` returns the active brand.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or fallback completed) before any copy?
- Ask-at-the-Peak trigger identified and named for this segment?
- Platform deep-links collected from user (or placeholders explicitly flagged)?
- Value-exchange framing present in T2 — not a favor ask?
- T3 (if present) uses a distinct angle from T1/T2?
- Incentive language absent on prohibited platforms (Google, App Store, Play Store)?
- No review-gating language anywhere in the sequence?
- `subject-line-preview-text-optimizer` called (or inline equivalent applied) on every email touchpoint?
- `cta-variant-generator` called (or inline equivalent) for the primary CTA and at least one alternate?
- `email-compliance-auditor-gdpr-can-spam` run (or inline equivalent) before marking ready?
- `lifecycle-email-push-copy-reviewer` final pass completed?
- Sequence saved to `./advocacy/[segment-slug]-review-sequence.md`?
