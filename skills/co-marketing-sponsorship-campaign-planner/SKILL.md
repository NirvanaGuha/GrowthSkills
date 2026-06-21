---
name: co-marketing-sponsorship-campaign-planner
description: >
  Takes a partner brand or sponsorship opportunity — along with the shared audience, campaign
  window, and proposed offer or exchange — and produces a complete, execution-ready co-marketing
  or sponsorship campaign plan. Outputs: aligned goals and success metrics (with clear ownership),
  an asset-split and responsibility matrix, a co-branded content and promotion calendar, a
  plain-language deal memo covering key terms (deliverables, timelines, cancellation, IP rights,
  exclusivity), and a promotion timeline with activation checkpoints. Applies the GIVE/GET
  Partnership Framework to ensure the value exchange is balanced and the audience overlap is real
  before any asset production begins. Calls brand-brain for voice and positioning, then calls
  competitive-intelligence-dossier if the partner brand is also a market player, and composes
  campaign-brief-builder + cta-variant-generator for on-brand asset briefs. Use whenever the user
  says "co-marketing plan," "sponsor this event / newsletter / podcast," "partner campaign,"
  "collab with [brand]," "joint webinar," "co-branded content," "sponsorship proposal," "deal
  memo," "partnership activation plan," or hands over a partner contact and asks what to build.
---

# Co-Marketing & Sponsorship Campaign Planner

A partner conversation without a plan is just a promise. This skill turns an inbound or outbound partnership opportunity into a full execution kit — goals, asset split, calendar, deal memo, and promotion timeline — so nothing falls through the cracks between "we should do something together" and the campaign going live.

The anchor is the **GIVE/GET Framework**: before any asset production, it pressure-tests whether the value exchange is genuinely balanced, whether the audience overlap is real and sized, and whether the timing actually works for both sides. If it doesn't, the skill flags it rather than building a plan on a weak foundation.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads active brand's voice, positioning, ICP, proof, and banned words. Does not resolve brand context itself.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP, core offer, primary value prop, and voice adjectives before proceeding.
- **`competitive-intelligence-dossier`** (conditional) — call if the partner brand overlaps with your competitive space; avoid partnership terms that inadvertently legitimize a competitor's positioning.
- **`campaign-brief-builder`** — generates per-asset creative briefs for co-branded assets; call once the asset split is agreed.
- **`cta-variant-generator`** — writes on-brand CTAs for each co-branded touchpoint (joint landing page, email header, event registration).
- **`partner-onboarding-checklist-generator`** (optional) — if this is a new partner relationship, hand off post-deal to generate the onboarding checklist.
- **`project-plan-generator-reviewer`** (optional) — for complex multi-month sponsorship activations, hand off the calendar to generate a full milestone-tracked project plan.

---

## How a run works

```
Step 0  Load brand context           ──► brand-brain (required)
Step 1  GIVE/GET audit               ──► validate the value exchange before planning anything
Step 2  Audience overlap sizing      ──► real or estimated; flag if unconfirmed
Step 3  Goals + metrics matrix       ──► by owner (your brand vs. partner brand)
Step 4  Asset split + responsibility ──► what each side makes, owns, approves
Step 5  Co-branded content calendar  ──► week-by-week activation map
Step 6  Deal memo (plain language)   ──► terms, deliverables, rights, kill clause
Step 7  Promotion timeline           ──► with pre-launch, live, and post-campaign phases
Step 8  Output + save                ──► inline summary + save full plan
```

---

## Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns voice adjectives, banned words, offer mechanics, real proof, positioning, and ICP. Apply these as hard overrides throughout — especially in deal memo language, co-branded copy, and CTA generation. Do not draft any campaign artifacts until brand-brain returns.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP, core offer, primary value prop, and voice adjectives before proceeding.

---

## The GIVE/GET Framework

The core diagnostic before any plan is built. Evaluate four dimensions:

### 1. Value Balance (GIVE/GET audit)
Map every deliverable to a side and assign a rough value (audience size × engagement rate × channel weight). A deal is worth building when the ratio sits between 0.7 and 1.3. Outside that range, either renegotiate or flag the imbalance explicitly.

| Side | Giving | Getting | Estimated value |
|------|--------|---------|-----------------|
| Your brand | e.g. 2 newsletter placements (40k subs, 35% OR) | e.g. 1 webinar co-host slot + partner list access | [assess] |
| Partner brand | e.g. 1 webinar co-host slot + list access (12k, 28% OR) | e.g. 2 newsletter placements | [assess] |

### 2. Audience Overlap Reality Check
Real overlap requires: (a) same or adjacent ICP, (b) similar awareness stage, (c) non-cannibalizing offers. If the user has no data, estimate from public signals (follower counts, Similarweb, LinkedIn audience data) and mark `[verify]`. Flag if overlap is <10% — it's a reach play, not a co-marketing play, and the plan changes accordingly.

### 3. Timing Fit
Map the proposed window against both brands' existing campaign calendars. Identify blackout periods (BFCM, product launches, fiscal close), shared moments of relevance (industry events, seasonal peaks), and the minimum viable lead time for each asset type (co-branded email: 2 weeks; joint webinar: 4 weeks; co-authored content: 3 weeks; event sponsorship: 6–12 weeks).

### 4. Strategic Fit
Score 1–5 on: brand voice compatibility, category adjacency (complementary vs. competing), past partnership signals (has the partner run co-marketing before?), and exclusivity risk (will either brand's other partners object?). Below a score of 3 on any dimension: flag and ask before proceeding.

---

## Goals + Metrics Matrix

Separate your brand's goals from the partner's — they're rarely identical, and misaligning them is the most common cause of post-campaign friction.

```
## Goals & Success Metrics

| Goal | Metric | Owner | Baseline | Target |
|------|--------|-------|----------|--------|
| Email list growth | Net new subscribers from partner | Your brand | [current] | [target] |
| Pipeline / leads | MQLs attributed to campaign | Your brand | — | [target] |
| Partner brand awareness | Reach / impressions in partner's channel | Partner | — | [target] |
| Shared conversion | Joint landing-page CVR | Both | — | [target] |

Measurement: UTM taxonomy per brand (see utm-parameter-bulk-builder if available); agree on a shared reporting date.
```

---

## Asset Split & Responsibility Matrix

Every co-marketing plan dies without this. Map every deliverable:

```
## Asset Split

| Asset | Creator | Approver | Deadline | Distribution channel |
|-------|---------|----------|----------|---------------------|
| Joint landing page copy | Your brand | Partner | [date] | Both brand sites |
| Co-branded email (your list) | Your brand | Partner (logo + quote) | [date] | Your ESP |
| Co-branded email (partner list) | Partner | Your brand (logo + quote) | [date] | Partner's ESP |
| Social graphics (both brands) | Assigned (by negotiation) | Both | [date] | Both accounts |
| Webinar/event run-of-show | Your brand (if hosting) | Partner | [date] | Zoom/platform |
| Press release or joint announcement | Negotiated | Both legal/comms | [date] | Both PR channels |
```

Approval SLA: establish a maximum 2 business day review turnaround in the deal memo; campaigns die in review queues.

---

## Co-Branded Content Calendar

A week-by-week activation map from announcement to post-campaign wrap. Format:

```
## Campaign Calendar — [campaign name], [window]

### Pre-launch (T-minus)
- T-4 weeks: Finalize deal memo; both brands sign off on asset list
- T-3 weeks: Share brand kits; each team produces their deliverables
- T-2 weeks: Cross-brand review pass; align on UTMs and tracking
- T-1 week:  Load assets into ESPs/schedulers; soft-promote to internal teams

### Live window
- Week 1: Launch email to both lists (Day 1); co-branded social post (Day 2–3)
- Week 2: Mid-campaign check-in (open rates, registrations, CTR) — adjust if needed
- Week 3: Reminder / amplification push; any live event/webinar

### Post-campaign
- T+3 days: Pull UTM data and partner-attributed metrics
- T+1 week: Joint debrief note (wins, misses, whether to renew)
- T+2 weeks: Case study or testimonial exchange (optional, agree in advance)
```

---

## Deal Memo (Plain Language)

Not a legal contract — a structured summary that prevents the most common disputes. Instruct the user to have their legal counsel review before signing any formal agreement.

```
## Deal Memo — [Your Brand] x [Partner Brand]
Campaign: [name] | Window: [dates] | Signed by: [names/titles] | Date: [date]

### Deliverables
[List exactly what each party commits to — asset type, quantity, channel, deadline]

### Audience access
[Which lists / channels each brand may use; whether partner data is shared or only activated by the partner on your behalf]

### Creative approvals
[Who approves what; turnaround SLA; what constitutes a veto vs. a suggestion]

### Exclusivity
[Category exclusivity: does either brand agree not to run a similar campaign with a direct competitor during the window? Duration?]

### IP & branding rights
[Each brand retains ownership of its own assets; co-branded assets jointly owned; neither party may use co-branded assets post-campaign without written consent]

### Performance expectations
[Agreed minimums, if any — e.g., "Partner will send to a list of no fewer than X subscribers." Non-performance clause.]

### Cancellation / kill clause
[Either party may cancel with X days written notice; deliverables completed to that point are non-refundable in time/cost; in-progress assets revert to creator]

### Reporting
[Both parties share UTM/attribution data within Y days of campaign end; neither party may publish joint results without the other's review]

### Renewal
[First right of refusal for a follow-on campaign; evaluation date]
```

---

## Promotion Timeline

A single linear view of the full arc — pre-deal through post-wrap — so both brands can align their own calendars.

```
## Promotion Timeline

[6–12 wks out]  Initial outreach / inbound → GIVE/GET audit → strategic fit score
[4–6 wks out]   Deal memo drafted and signed; campaign brief issued
[3–4 wks out]   Brand kits exchanged; assets in production
[2–3 wks out]   Cross-review pass; UTMs built; ESPs loaded; social scheduled
[1 wk out]      Internal previews; last-chance changes frozen
[Launch day]    Coordinated simultaneous send/post (sync times across time zones)
[Live window]   Mid-campaign check-in at 50% mark; daily metric pull if lead-gen campaign
[Campaign end]  UTM data pull; partner debrief within 72 hours
[T+2 wks]       Joint results summary; renewal discussion; archive co-branded assets
```

---

## Principles (Non-Negotiable)

- **GIVE/GET before any asset.** Never build a plan on an unvalidated value exchange.
- **Brand-brain first.** Voice, positioning, and banned words from the active brand override everything — including a partner's preferences.
- **Audience overlap must be real.** Estimate if needed, mark `[verify]`, but never assume it.
- **Own what you commit.** Assign every deliverable to an owner. Unowned deliverables become missed deliverables.
- **Separate goals by brand.** Your KPIs and the partner's KPIs will diverge — track them separately and compare only shared metrics.
- **Truth discipline.** Real audience sizes, real engagement rates, or `[verify]`. Never inflate reach to win a partner.
- **Kill clause is mandatory.** Every deal memo must have one.

---

## What Not to Do

- Don't produce an asset split or calendar before the GIVE/GET audit clears. A lopsided deal produces a lopsided plan.
- Don't assume the partner's audience is your ICP. Validate overlap before building messaging.
- Don't reimplement brand scanning — call `brand-brain`.
- Don't write a legal contract. The deal memo is a structured summary; flag legal review explicitly.
- Don't let "aligned on the idea" substitute for a signed or at least countersigned deal memo before asset production starts.
- Don't use the partner's brand voice in your assets; each brand writes its own half.
- Don't skip the post-campaign debrief step. Renewals are won or lost there.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or fallback triggered) before any output?
- GIVE/GET ratio assessed; imbalance or missing data flagged, not papered over?
- Audience overlap confirmed or marked `[verify]` with source noted?
- Goals matrix separates your brand's KPIs from the partner's?
- Every asset in the split has an assigned creator, approver, deadline, and channel?
- Co-branded calendar covers pre-launch, live, and post-campaign phases?
- Deal memo includes: deliverables, audience access, approvals SLA, exclusivity, IP rights, kill clause, reporting, renewal?
- Promotion timeline is linear and both brands can sync their own calendars to it?
- Brand voice, positioning, and banned words honored throughout?
- Plan saved to `./partnerships/[partner-slug]-campaign-plan.md`?
