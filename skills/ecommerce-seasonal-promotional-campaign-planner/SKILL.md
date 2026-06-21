---
name: ecommerce-seasonal-promotional-campaign-planner
description: >
  Takes a sale window, discount structure, and product selection and turns them into a complete,
  week-by-week multi-channel campaign calendar with ready-to-use copy blocks, channel-specific
  timing logic, and a rollout checklist — built on the RACE (Reach, Act, Convert, Engage) + Promotional
  Intensity Curve framework so every campaign has a proper pre-launch warm-up, peak, and post-sale
  retention tail. Outputs email sequence structure, web push notification copy, banner copy, SMS
  if applicable, and a go/no-go launch checklist — composing existing copy skills rather than
  re-implementing them. On-brand throughout via brand-brain. Use when the user says "plan my BFCM
  campaign," "build a Black Friday calendar," "map out our summer sale," "set up a flash sale
  sequence," "what emails should I send for the holiday sale," "design a coupon campaign," "I need
  a launch-week promotion plan," "help me plan [any seasonal event / promotional window],"
  or hands over a discount brief and asks what to do with it.
---

# eCommerce Seasonal & Promotional Campaign Planner

Give it a window, a deal, and a product selection. Get back a complete week-by-week campaign calendar — email cadence, push notification schedule, banner copy, channel timing, and a launch checklist — with the copy blocks already drafted by composing the library's specialist skills. Every asset is on-brand, phased correctly, and calibrated for the promotional intensity curve so you don't blow your list with Day 1 urgency and have nothing left for the peak.

This skill plans and orchestrates. It does not redesign your storefront, rebuild your ESP automation, or invent discounts that aren't in the brief.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's voice, ICP, banned words, real proof, and offer mechanics before any copy is written. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP, voice adjectives, banned words, and primary product/offer before proceeding.
- **`subject-line-preview-text-optimizer`** — generates and pressure-tests subject lines + preview text for each email send in the sequence.
- **`push-notification-copy-generator`** — writes the title/body/CTA for each web push notification in the schedule.
- **`cta-variant-generator`** — produces button labels and friction-reducer microcopy for key conversion moments (cart page, email, pop-up).
- **`campaign-brief-builder`** — (optional) produces a shareable brief document if the user needs one for agency/team handoff.
- **`campaign-qa-launch-checklist-generator`** — produces the go/no-go checklist at the end of the planning pass.
- **`lifecycle-email-push-copy-reviewer`** — reviews the assembled sequence for voice, flow, urgency pacing, and character-limit compliance after the first draft.
- **`utm-parameter-bulk-builder`** — (optional) generates a validated UTM URL sheet for every campaign link if the user requests tracking setup.

---

## How a run works

```
Step 0  Load the brand       ──► call brand-brain (bootstraps on first use)
Step 1  Scope the campaign   ──► parse brief; clarify gaps with ≤4 questions
Step 2  Map the calendar     ──► place phases + send slots on the week-by-week grid
Step 3  Draft copy blocks    ──► compose subject-line-preview-text-optimizer,
                                  push-notification-copy-generator, cta-variant-generator
Step 4  Review the sequence  ──► lifecycle-email-push-copy-reviewer pass
Step 5  Output the plan      ──► calendar + copy + checklist; offer UTM + brief add-ons
```

---

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. It returns the active brand digest — voice adjectives, banned words, offer mechanics, real proof, positioning, ICP + awareness tendency — and the path to `brand.md`. Do not draft a single copy line until `brand-brain` returns.

Obey its voice and banned words as hard overrides. Use only its real proof points; mark any others `[verify]`.

**Fallback if brand-brain is absent or returns no brand:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP, voice adjectives, banned words, and primary product/offer before proceeding.

---

### Step 1 — Scope the campaign (parse + minimal clarification)

Extract from the brief:
- **Sale window** — exact dates (start/end); if ambiguous, ask.
- **Discount structure** — flat %, tiered spend thresholds, BOGO, bundle, free shipping, gift-with-purchase, or coupon code.
- **Product selection** — hero SKU(s), full-catalog, or a curated set.
- **Channels in play** — email, web push, SMS, on-site banners, paid social (flag which are confirmed vs. "nice to have").
- **List / subscriber context** — warm list, cold list, frequency of previous sends (informs safe send cadence).
- **Goal** — revenue target, new customer acquisition, LTV extension, inventory clearance, or launch amplification.

Ask ≤4 clarifying questions if critical fields are missing. Never block the run over optional fields — default reasonably and note assumptions.

---

## The RACE + Promotional Intensity Curve framework

Every campaign is built on two interlocked structures:

**RACE phases** (Dave Chaffey / Smart Insights):
| Phase | Job | Timing relative to peak |
|---|---|---|
| **Reach** | Build awareness; seed the event with your warmest audiences | T−14 to T−7 |
| **Act** | Drive engagement; early-access offers, wishlist building, preview content | T−7 to T−1 |
| **Convert** | Full-force launch; urgency ladder activates | T0 to T+N |
| **Engage** | Post-sale retention; review ask, cross-sell, loyalty nudge | T+1 to T+14 |

**Promotional Intensity Curve** governs email + push frequency so urgency is earned:
- **Warm-up (Reach/Act):** 1–2 emails/week; 0–1 push/week. Tone: curious, low-commitment. No "LAST CHANCE."
- **Launch peak:** 1 email on Day 0 + 1 on Day 3 minimum; 1–2 push/day during peak window. Tone escalates to specific urgency with real countdown.
- **Tail urgency:** "Final 24h / Final hours" email + push only when true. One send max.
- **Post-sale (Engage):** 1 email within 48h of close; 1 follow-up 7 days out. Push optional. Tone resets to brand-warm.

**Segment splits always considered:**
- Purchasers during the event → skip urgency; move to post-sale track immediately.
- Non-openers on Day 3 → subject-line re-test (call `subject-line-preview-text-optimizer` for Variant B).
- Non-purchasers after close → win-back angle (coordinate with `abandon-flow-writer` if a cart-abandon flow is in play).

---

## Week-by-week calendar output format

```
## [Campaign Name] — [Sale Window] — [Brand]

### Phase 1 — Reach (T−14 to T−7)
| Date | Channel | Send / Asset | Goal | Subject Line / Copy Block | CTA |
|------|---------|--------------|------|---------------------------|-----|

### Phase 2 — Act (T−7 to T−1)
[same table structure]

### Phase 3 — Convert (T0 → close)
[same table structure — include Day 0, mid-sale, and close-day rows explicitly]

### Phase 4 — Engage (T+1 to T+14)
[same table structure — include purchaser track and non-purchaser track separately]

### Assumptions & constraints noted
[List any filled-in defaults and questions the user should confirm before activation]
```

Copy blocks are drafted inline, drawing on `subject-line-preview-text-optimizer`, `push-notification-copy-generator`, and `cta-variant-generator` — each called in turn as their asset type appears in the calendar.

---

## Campaign-type calibration

| Type | Key adjustments |
|---|---|
| **BFCM (Black Friday / Cyber Monday)** | Run Reach from T−21 (inbox is crowded; early-access angle earns opens). Convert phase splits: BF launch, weekend sustain, CM-specific send. Post-sale track critical for AOV lifts. |
| **Flash sale (24–72h)** | No Reach phase. Act = same-day or 24h pre-announcement. Convert is compressed (morning launch + midday push + 2h-left push). Engage still required. |
| **Product launch + promotional bundle** | Reach = waitlist + teaser content; Act = early-access for VIPs; Convert = launch-day email + paid social retargeting brief; Engage = social proof collection (call `review-testimonial-solicitation-sequence`). |
| **Coupon / percent-off campaign** | Always include expiry logic — real deadlines only. Highlight savings in dollar terms as well as percent (e.g., "Save $47" alongside "25% off") — concrete is more persuasive. |
| **Clearance / end-of-season** | Lead with scarcity of units, not a discount (frame: "Only 12 left at this price," not "MASSIVE SALE"). ICP-match the selection — don't spray the full catalog. |
| **Holiday gifting (Valentine's, Mother's Day, etc.)** | Shift awareness earlier — gifters plan. Lead with emotion, not discount. Shipping deadline is the real urgency lever, not the price cut. |

---

## Discount mechanics integrity

Before writing urgency copy, verify:

1. **Is the deadline real?** A "sale ends midnight" that auto-extends destroys sender reputation and FTC compliance. Flag if brief implies rolling/evergreen urgency.
2. **Is the discount defensible?** Crossed-out prices must reflect a genuine prior price (FTC 16 CFR Part 233, `[verify]` jurisdiction-specific rules). Never imply a fictional MSRP.
3. **Is the coupon code exclusive or public?** Exclusive codes in email enable segmentation; public codes risk affiliate/coupon-site attribution bleed — note it.
4. **Tiered thresholds:** Confirm the tier breakpoints match your actual AOV distribution or the tiers will sit unused. If AOV is unknown, flag the assumption.

Any integrity issue is surfaced in the plan's Assumptions section — not hidden, not auto-fixed.

---

## Artifacts saved

Save the full campaign plan to `./campaigns/[brand-slug]-[event-slug]-[YYYY-MM-DD].md` so it survives session reset and can be handed off. File includes: calendar table, all copy blocks, assumptions, and checklist.

Do not overwrite an existing file for the same event window — append a `-v2` suffix and note what changed.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No copy before the active brand context returns.
- **Compose, don't rebuild.** Subject lines come from `subject-line-preview-text-optimizer`. Push copy from `push-notification-copy-generator`. CTAs from `cta-variant-generator`. Don't re-implement them here.
- **Urgency is earned, not assumed.** The intensity curve governs tone per phase. "LAST CHANCE" on Day 1 is a lie — call it out.
- **Real deadlines only.** Evergreen countdown urgency is both a legal risk and a list-burndown.
- **Segment purchasers out immediately.** Sending Day 3 urgency to someone who already bought is brand damage.
- **Post-sale is not optional.** The Engage phase is where LTV and review velocity are built. It stays in the plan.
- **Truth only.** Real proof points or `[verify]`; never invent social proof, claimed savings, or "bestseller" designations.

## What Not to Do

- Don't write any copy before `brand-brain` returns the active brand.
- Don't send urgency copy (LAST CHANCE, FINAL HOURS) before the final 24h of the sale window.
- Don't build a single-blast campaign and call it a plan — every event needs at least Reach + Convert + Engage.
- Don't invent discounts not in the brief; don't imply fictional before-prices.
- Don't produce a calendar without noting which channels are confirmed vs. assumed.
- Don't use coupon codes without noting the attribution/bleed risk.
- Don't skip the `lifecycle-email-push-copy-reviewer` pass — it catches voice drift and char-limit overruns before the user touches a template.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or bootstrapped) before any copy drafted?
- Sale window, discount structure, product selection, and channel list confirmed or clearly noted as assumed?
- RACE phases correctly placed on the calendar with dates?
- Promotional Intensity Curve respected — no peak-level urgency in Reach/Act phases?
- Purchaser suppression logic noted in the Convert → Engage handoff?
- Non-opener re-send slot included with a Variant B subject line?
- `subject-line-preview-text-optimizer`, `push-notification-copy-generator`, and `cta-variant-generator` each called for their respective asset types?
- `lifecycle-email-push-copy-reviewer` pass completed on the draft sequence?
- Discount mechanics integrity check done — real deadlines, defensible pricing, coupon attribution note?
- Artifacts saved to `./campaigns/[brand-slug]-[event-slug]-[YYYY-MM-DD].md`?
- Assumptions section lists every filled-in default for user confirmation?
