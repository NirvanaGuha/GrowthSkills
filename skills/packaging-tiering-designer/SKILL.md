---
name: packaging-tiering-designer
description: >
  Turns a product feature list and customer segments into a defensible good-better-best tier
  structure — including a feature gating map, an ARPU-maximizing upgrade path, and the upgrade
  moment triggers that move buyers across tiers. Works for SaaS, plugins, media, and physical
  bundles. Two modes: Design (no tiers exist yet — outputs the full tier architecture from
  scratch) and Audit (existing tiers exist — scores them, flags leakage and compression, and
  returns an improvement diff). Calls `brand-brain` for voice, ICP, and offer mechanics; calls
  `offer-pricing-brain` for canonical pricing data; calls `competitor-price-benchmarking-analyst`
  to anchor tier boundaries against the market; calls `icp-persona-builder` for segment depth
  when personas are missing. Downstream skills — `pricing-page-optimizer`, `cta-variant-generator`,
  `upgrade-expansion-prompt-writer` — consume the output directly. Use whenever the user says
  "design our tiers," "restructure packaging," "what should go in each plan," "build a tier map,"
  "good-better-best," "feature gating," "free vs paid," "which features to gate," "upgrade path,"
  "packaging review," "audit our plans," or hands over a feature list and asks what to charge for.
---

# Packaging & Tiering Designer

A tier structure is a segmentation hypothesis made permanent in a pricing page. Get it wrong and
you either compress ARPU by giving power-user features away free, or you stall conversion by
over-gating features that prove value. This skill designs the tier architecture from first
principles — or audits an existing one — using the Van Westendorp / Good-Better-Best framework
as a spine and value-metric anchoring as the discipline.

It does not invent feature names or set exact dollar prices. It produces the structure — what
belongs in each tier, what fences separate them, and the upgrade moment where each fence should
fire. Price-setting from that structure is downstream.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's ICP, voice, banned words, offer
  mechanics, and existing pricing destinations. This skill does not re-derive brand context.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written
  brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no `brand.md` exists,
  ask the user for [product name, primary ICP + segment split, current feature list, and any existing
  tier/pricing constraints].
- **`offer-pricing-brain`** (required) — canonical source for existing tier definitions, free-trial
  mechanic, card-required/no-card, guarantee, and pricing page URLs. Never re-derive pricing from
  memory; pull it here.
- **`competitor-price-benchmarking-analyst`** (Step 1b) — runs if competitor context is missing;
  anchors tier boundary thresholds against the market so fences aren't set in a vacuum.
- **`icp-persona-builder`** *(optional)* — runs if segment personas are thin; produces the per-segment
  JTBD and WTP signal the tier map depends on.
- **`synthetic-persona-interview`** *(optional)* — pressure-tests draft tier fences: put each
  persona in front of the gating map and ask "do you upgrade or churn?"
- **`roi-business-case-calculator`** *(optional)* — quantifies the ARPU delta of a proposed change
  when the user needs a business case.
- **`pricing-page-optimizer`** *(downstream)* — consumes the output tier map to audit page presentation.
- **`cta-variant-generator`** *(downstream)* — writes tier-specific CTAs from the tier names and
  value props produced here.
- **`upgrade-expansion-prompt-writer`** *(downstream)* — writes the in-product upgrade prompts for
  each fence event produced here.

---

## How a run works

```
Step 0  Load the brand          ──► brand-brain + offer-pricing-brain
Step 1  Gather inputs           ──► feature list, segment map, competitor context
Step 2  Build the value metric  ──► the single dimension that scales with customer value
Step 3  Design or Audit         ──► tier architecture / gap + leakage report
Step 4  Fence map               ──► gating decisions per feature, upgrade-moment triggers
Step 5  ARPU path               ──► upgrade logic, expansion levers, land-and-expand moves
Step 6  Downstream handoff      ──► what to pass to pricing-page-optimizer / cta / upgrade prompts
```

---

### Step 0 — Load the brand (always first)

**Invoke `brand-brain`** (Skill tool, `skill: brand-brain`). It returns the ICP, voice, banned
words, offer mechanics, and existing tier/pricing destination URLs. Then **invoke `offer-pricing-brain`**
for the canonical tier definitions, limit values, and free-trial mechanic. Do not write the tier
map until both return.

Apply the ICP segment split directly to the tier architecture: each tier should map cleanly to
one primary segment (and at most one adjacent up-segment who outgrows it). A tier that maps to no
real segment is waste.

---

### Step 1 — Gather inputs

The minimum viable input set:

| Input | How to resolve if missing |
|---|---|
| Feature list | Ask the user; or read product docs / changelog |
| Customer segments (≥2) | Call `icp-persona-builder`; or ask user for JTBD + WTP signal |
| Competitor tier structure | Call `competitor-price-benchmarking-analyst` |
| Current ARPU or revenue split (optional) | Ask; mark estimates `[verify]` |
| Growth motion (PLG self-serve / sales-assisted / hybrid) | Ask; shapes free-tier generosity + fence placement |

Never invent WTP numbers. If real data is absent, use market comparables and mark them `[verify]`.

---

## Design mode (no tiers yet) — the Good-Better-Best build

### 2a — Choose the value metric

The value metric is the single dimension that (a) scales with the benefit the customer receives,
(b) the customer understands immediately, and (c) you can meter. Classic examples: seats, contacts,
events/mo, sites, API calls, monthly active devices.

Weak value metrics (storage GB, "features," generic "usage") compress ARPU because they don't
map to perceived value. Name the metric and defend it in one sentence. If two metrics compete,
pick the one closest to the outcome the customer buys.

### 2b — Set tier ceilings, not just floors

Each tier is a ceiling that the in-tier segment won't hit and the next-tier segment will. Work
backward from each segment's JTBD to the natural ceiling:

- **Free / Starter:** generous enough to activate value, restrictive enough to create a clear pain
  point. Activation > conversion. The fence fires when they hit the value-metric ceiling, not
  arbitrarily.
- **Core / Growth (the volume tier):** covers 60–80% of paid users [verify for your product].
  Price anchors against the competitor median. Features here should be "everything you need to
  succeed at the core job."
- **Pro / Business:** covers power users + small teams. The upgrade trigger is collaboration,
  automation, or compliance — things the Core tier buyer didn't need until they scaled.
- **Enterprise (optional):** custom limits + security + SLA + dedicated CSM. Never put features
  here that mid-market buyers need — it tanks conversion.

Output the tier table:

```
## Tier architecture — [brand slug]
Value metric: [name] · Growth motion: [PLG / sales-assisted / hybrid]

| Tier | Primary segment | Value-metric ceiling | Price signal | Activation goal |
|---|---|---|---|---|
| [name] | [segment] | [limit] | [free / $X / custom] | [what they must do] |
...

Tier naming note: [on-brand, benefit-led names — per brand voice]
```

### 2c — Feature gating map

For every feature in the list, assign it to the lowest tier where it legitimately belongs using
the **four-quadrant gating test**:

| Quadrant | Test | Decision |
|---|---|---|
| Prove value | Does it demonstrate the core product promise? | Free/Starter — never gate it |
| Drive adoption | Is it essential to the primary JTBD of the Core tier? | Core — include |
| Create stickiness | Does it increase switching cost or team lock-in? | Pro/Business — gate here |
| Enterprise-only | Compliance, SSO, SLA, dedicated support? | Enterprise only |

Output as a table: Feature | Quadrant | Assigned tier | Gate type (hard limit / soft nudge / locked) | Upgrade message (one line).

Do not gate "prove value" features. It is always wrong to lock a feature that demonstrates the
core promise — even if competitors charge for it.

---

## Audit mode (tiers exist)

Run when the user provides an existing tier structure. Score each tier across five dimensions:

| Dimension | Question | Score 0–3 |
|---|---|---|
| Segment fit | Does each tier map to exactly one real segment? | |
| Value-metric health | Does the metric scale with the value delivered? | |
| Fence placement | Do fences fire at natural pain points, not arbitrary limits? | |
| Feature leakage | Are stickiness/collaboration features in the wrong tier (too low)? | |
| Upgrade compression | Is the gap between any two adjacent tiers too small to justify switching? | |

Flag every issue with: **Tier · Dimension · Problem · Fix recommendation**.

Produce a diff table: Current state → Proposed state for each flagged item. Quantify estimated ARPU
impact where possible; mark estimates `[verify]`.

---

## Step 5 — ARPU-maximizing upgrade path

A tier structure without an upgrade path is a static snapshot. The upgrade path makes it a
revenue motion. For each adjacent tier pair, define:

1. **The fence event** — the specific action, metric threshold, or moment that creates the felt
   need to upgrade (e.g., "user hits 1,000 contact limit," "user tries to invite a second team member").
2. **The upgrade nudge** — where in-product the prompt fires, what it says (one sentence; feed to
   `upgrade-expansion-prompt-writer`).
3. **The expansion lever** — beyond seat-based or limit-based upgrades, is there a usage-based
   expansion play (more sites, more events, add-ons)?

Output:

```
## Upgrade path — [brand slug]
| From → To | Fence event | Nudge placement | Expansion lever |
|---|---|---|---|
...
```

Note any land-and-expand plays: accounts that enter at Starter but have enterprise team size signal
should be flagged for sales outreach at a defined usage threshold.

---

## Step 6 — Downstream handoff

Produce a handoff block at the end of every run. These skills consume the output directly — no
re-input needed, just pass the handoff block:

```
## Downstream handoff
- pricing-page-optimizer: [tier names, value metric, feature table]
- cta-variant-generator: [tier name, primary segment, activation goal, destination URL from offer-pricing-brain]
- upgrade-expansion-prompt-writer: [fence event per tier pair, upgrade nudge placement]
```

Save the full output to `./pricing/[slug]-tier-map.md` and confirm the path. The file is the single
source of truth for all downstream pricing work; downstream skills read it rather than re-deriving.

---

## Principles (Non-Negotiable)

- **Value metric first.** Every fence and every price signal derives from the value metric. Design
  the metric before you assign a single feature to a tier.
- **Tiers are segment hypotheses.** Each tier must map to a real, named segment. A tier that maps to
  nobody is a pricing debt.
- **Never gate proof-of-value features.** If a feature demonstrates the core promise, it belongs in
  Free or Starter — gating it reduces conversion without protecting ARPU.
- **Fences fire at pain, not at arbitrary limits.** A limit set at 500 contacts when the median
  activated user has 480 is a churn event, not an upgrade event.
- **Real numbers or `[verify]`.** Never invent WTP data, competitor prices, or ARPU benchmarks.
- **Brand-brain first.** ICP, voice, and existing offer mechanics come from `brand-brain` and
  `offer-pricing-brain`. Do not re-derive pricing from memory or general web knowledge.

## What Not to Do

- Don't set exact dollar prices — this skill produces the structure; pricing from that structure
  is owned by `offer-pricing-brain` and the operator.
- Don't create four or more tiers for a product with two distinct segments — complexity kills
  conversion. Three is almost always the ceiling; two is often right.
- Don't gate features by "what competitors charge for" alone — market position is a signal, not
  a justification.
- Don't design the upgrade path before the tier architecture is confirmed — the path depends on
  the fences.
- Don't produce a tier map without the downstream handoff block — the point is that `cta-variant-generator`
  and `upgrade-expansion-prompt-writer` work from it immediately.
- Don't reimplement brand scanning or pricing resolution — call `brand-brain` and `offer-pricing-brain`.

## Quality Checklist (self-review before presenting)

- `brand-brain` and `offer-pricing-brain` called and returned before any tier was designed?
- Value metric named and defended in one sentence?
- Each tier maps to exactly one primary segment (Design mode)?
- Feature gating map covers every feature in the input list, with quadrant assignment and gate type?
- Upgrade path covers every adjacent tier pair with a named fence event and nudge placement?
- All WTP data / ARPU benchmarks are real (cited) or marked `[verify]`?
- Downstream handoff block present and complete?
- Output saved to `./pricing/[slug]-tier-map.md`?
