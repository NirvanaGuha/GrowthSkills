---
name: audience-targeting-spec-writer
description: >
  Turns a campaign goal and ICP into a platform-ready audience targeting and retargeting spec for
  Google Ads, Meta (Facebook/Instagram), and LinkedIn — covering cold audiences, warm retargeting
  layers, custom/lookalike signals, window logic, exclusion rules, and negative audiences. Produces
  structured, implementation-ready specs your media buyer or agency can activate without a briefing
  call. Does NOT write ad copy (hand to ad-copy-variant-generator or social-ad-copy-writer) and does
  NOT calculate bid/budget (hand to bid-budget-pacing-checker or channel-roi-scorecard) — it does
  one thing: define WHO gets served the ad, on WHICH signal, with WHAT exclusions, and WHY. Calls
  brand-brain to anchor the ICP, and data-qa-measurement-gotcha-checker to flag attribution/pixel
  gaps before the spec ships. Use when the user says "build my audiences," "write a targeting spec,"
  "who should I target," "set up retargeting," "Meta/Google/LinkedIn audience layers," "custom
  audiences," "lookalike audiences," "define audience segments for paid," or hands over a campaign
  brief and asks what to target.
---

# Audience Targeting Spec Writer

Give it a campaign goal and ICP, get a platform-ready targeting spec — not just "use interest
targeting" but precise audience layers with signal sources, window logic, inclusion/exclusion
rules, and a plain-English rationale for every layer. Each spec is anchored to the brand's real
ICP, not a hypothetical, because brand context comes from `brand-brain`.

This skill specifies audiences. It does not write ad copy, set bids, or build the campaign
structure. If the pixel is broken or attribution windows are misconfigured, it says so before
the spec ships — it will not let a well-built audience be poisoned by bad measurement.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — resolves the active brand's ICP, positioning, proof, and
  offer. Audience targeting without a real ICP is guesswork; do not begin until `brand-brain`
  returns.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
  brand's `brand.md` directly; if none exists, ask the user for ICP description (job title, company
  size/type, pain triggers), campaign goal, and offer/landing-page URL before proceeding.
- **`data-qa-measurement-gotcha-checker`** (Step 4, gate) — validates pixel health, attribution
  window settings, and conversion event configuration before the spec is finalized. A targeting spec
  built on a broken pixel is wasted spend.
- **`icp-persona-builder`** *(optional)* — if brand-brain returns a thin or low-confidence ICP,
  call this skill to deepen the persona before building audience layers.
- **`channel-roi-scorecard`** *(optional, downstream)* — hand the completed spec to this skill to
  model expected CPL/CPA and validate channel-level spend allocation.
- **`ad-copy-variant-generator`** or **`social-ad-copy-writer`** *(downstream)* — once audiences
  are defined, these write the matching copy. Message-match between audience intent and ad copy is
  the #1 creative lever; the spec should be handed to them explicitly.
- **`linkedin-campaign-spec-builder`** *(downstream, LinkedIn only)* — for LinkedIn campaigns,
  pass the completed targeting layers to this skill for the full campaign object spec (objective,
  bid type, ad format, copy brief).

---

## How a run works

```
Step 0  Load the brand + ICP  ──► call brand-brain
Step 1  Clarify scope          ──► goal, budget tier, platform(s), funnel stage
Step 2  Build audience layers  ──► TOFU cold / MOFU warm / BOFU hot + exclusions
Step 3  Define retargeting     ──► signal sources, windows, sequence logic
Step 4  Run the measurement gate ──► call data-qa-measurement-gotcha-checker
Step 5  Output the spec        ──► platform-ready tables + a rationale note per layer
Step 6  Flag downstream steps  ──► copy, bids, creative brief handoffs
```

---

### Step 0 — Load the brand (always first)

**Invoke `brand-brain`** (Skill tool, `skill: brand-brain`). Use the returned ICP digest — job
function, seniority, company type/size, pain triggers, awareness stage, and typical acquisition
channel — as the audience targeting foundation. Obey brand voice and banned words in any spec
narrative. Use only real proof for rationale copy; mark anything unconfirmed `[verify]`.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
brand's `brand.md` directly; if none exists, ask the user for ICP description, campaign goal, and
offer URL before proceeding.

---

### Step 1 — Scope the brief

Before building layers, lock four inputs (ask only for what's missing):

| Input | Why it changes the spec |
|---|---|
| **Campaign goal** | Awareness → broad cold TOFU; Conversion → hot BOFU retargeting-first |
| **Platforms** | Google, Meta, LinkedIn, or a combination (different signal systems) |
| **Funnel stage emphasis** | Full-funnel vs. pure retargeting vs. cold acquisition only |
| **Budget tier** | <$2K/mo → 1–2 platforms, minimal layers; $10K+ → full-funnel multi-layer |

For B2B SaaS: default to LinkedIn (job-function/seniority) + Google (keyword intent) as the
primary pair. For eCommerce/B2C: default to Meta (behavioral/lookalike) + Google (Shopping +
search intent). State the default; let the user override.

---

## The TOFU/MOFU/BOFU Layer Model (standard paid-media convention)

Every spec is organized into three layers regardless of platform. The layer defines the relationship
the prospect has with the brand or category — it drives window logic, messaging angle, and
exclusion rules.

### Layer 1 — TOFU Cold (Reach new ICP-matched prospects)

**Signal type:** behavioral interest, job function/seniority, intent keywords, lookalike/similar
audiences from proven converters.

**Google:** In-market audiences (from the relevant IAB category + custom intent keywords from
top-of-funnel queries), customer match lookalike if seed ≥ 1,000, life events if relevant.

**Meta:** Core audience (interest stacks from the ICP's professional or lifestyle signals) +
Lookalike 1–3% from the brand's pixel purchasers or high-value custom audience. **Important:**
post-iOS 14.5, avoid over-relying on narrow interest stacks; use Advantage+ audience with signals
where ROAS data supports it, but keep a manual comparison split until data accumulates.

**LinkedIn:** Job Function + Seniority + Company Size (ABM: Company List or Industry). Use
Matched Audiences only when a quality list exists (>300 matched). Layered Company Size narrows
to budget-qualified accounts.

**Exclusions (cold):** all existing customers (by email list or pixel event), current trial/free
users, recent visitors (>30-day pixel window already in MOFU/BOFU).

### Layer 2 — MOFU Warm (Re-engage engaged non-converters)

**Signal type:** site visitors (page-depth segmented), video viewers (25–75%), lead-form openers
who did not submit, email list non-openers vs. openers-not-converted.

**Window logic** (enforce these — misconfigurations are the #1 retargeting waste):

| Engagement type | Window |
|---|---|
| Site visitor (any page) | 30 days |
| Pricing / plan / demo page visitor | 14 days |
| Blog/content consumer | 60 days |
| Video view 25%+ | 45 days |
| Lead form opener (no submit) | 7 days |
| Email click → site, no convert | 14 days |

**Exclusions (warm):** customers, converters (by the defined conversion event), current trial users,
anyone who already entered a lower-funnel sequence (BOFU).

### Layer 3 — BOFU Hot (Close high-intent near-converters)

**Signal type:** demo-request page visitors (no submit), cart abandoners (eCommerce), pricing page
3+ visits, free trial users (product-qualified, not yet converted), high-score leads in CRM
(if Customer Match available).

**Window:** 7–14 days maximum. Intent signal decays fast; beyond 14 days move to MOFU.

**Exclusions (hot):** existing paying customers by email list + pixel purchase event. Do NOT
exclude trial users here — they ARE the BOFU audience for SaaS.

**Special case — retargeting sequences:** when budget allows, build BOFU → MOFU handoff:
if a user converts on a BOFU micro-commitment (watches a demo video, starts a trial) but does not
reach the hard conversion, exclude them from BOFU and enroll them in MOFU with a different angle.
Map this as an explicit sequence in the spec.

---

### Step 4 — Measurement gate (call data-qa-measurement-gotcha-checker)

**Before finalizing the spec**, invoke `data-qa-measurement-gotcha-checker`. Common gotchas that
corrupt audience targeting:

- **Pixel not firing on conversion page** → BOFU custom audiences are empty, lookalikes are
  poisoned with non-converters.
- **Attribution window mismatch** → Meta 7-day click / 1-day view vs. Google last-click creates
  double-counting; document which window the spec is built for.
- **Google Ads conversion import from GA4 vs. native** → can create duplicate conversions used
  for Smart Bidding signals, which mis-trains tROAS/tCPA.
- **iOS 14.5+ signal loss (Meta)** → Conversions API (CAPI) required for reliable pixel events;
  flag if CAPI is not confirmed live.
- **LinkedIn Insight Tag scope** — Insight Tag must be on all pages, not just thank-you page, for
  correct visit-based retargeting.
- **(not set) in GA4** → if traffic sources show high (not set) %, the referral path from paid is
  broken; UTMs may be missing or overwritten.

If the gate reveals critical gaps (pixel not firing on conversion page; no CAPI on Meta), halt
and surface a fix-first recommendation before shipping the spec. A targeting spec built on broken
signals is actively misleading.

---

## Output format

Deliver a spec document, not a narrative. Each platform gets its own block.

```
## Audience Targeting Spec — [Brand] · [Campaign Goal] · [Date]

### Measurement gate
[ Pass / Conditional / Blocked — findings from data-qa-measurement-gotcha-checker ]

### Platform: [Google | Meta | LinkedIn]

#### Layer 1 — TOFU Cold
| Segment name | Signal / targeting method | Rationale | Size est. | Exclusions |
| ... |

#### Layer 2 — MOFU Warm
| Segment name | Signal / source | Window | Rationale | Exclusions |
| ... |

#### Layer 3 — BOFU Hot
| Segment name | Signal / source | Window | Rationale | Exclusions |
| ... |

#### Sequence logic (if applicable)
[ If BOFU → MOFU handoff is designed, map it here ]

### Downstream handoffs
- Ad copy: hand this spec to `ad-copy-variant-generator` / `social-ad-copy-writer`
- Bids: hand to `bid-budget-pacing-checker` or `channel-roi-scorecard`
- LinkedIn campaign object: hand to `linkedin-campaign-spec-builder`
```

Save spec to `./targeting/[brand-slug]-targeting-spec-[YYYY-MM-DD].md` unless the user says
inline only.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No audience layer is built before the ICP is loaded from `brand-brain`.
  Generic "interest: marketing" targeting is not a spec — it is a placeholder.
- **Windows are mandatory.** Every retargeting layer has an explicit window. No window = no layer.
- **Exclusions are not optional.** Every layer documents what it excludes or it is incomplete.
- **Measurement gate before ship.** A targeting spec without a pixel/attribution check is
  actively harmful; it directs spend toward signals that may not exist.
- **Platform mechanics matter.** LinkedIn Company Size ≠ Meta audience size ≠ Google reach; never
  copy-paste the same audience definition across platforms without translating the signal type.
- **ICP anchors everything.** If the ICP is thin, call `icp-persona-builder` before proceeding.
- **Truth only.** Real audience signals or `[verify]`; never invent size estimates.

---

## What Not to Do

- Do not write ad copy or set bids — name the downstream skills and hand off.
- Do not reimplement brand scanning or ICP building — call `brand-brain` and `icp-persona-builder`.
- Do not produce a spec with missing exclusions on customer lists — it will waste spend on people
  who already converted.
- Do not recommend broad interest stacks as a standalone TOFU strategy on Meta post-iOS 14.5
  without flagging CAPI dependency.
- Do not skip the measurement gate for "quick" runs — a broken pixel cannot be retroactively fixed.
- Do not list audience layers without windows; never leave window logic to the buyer's judgment.
- Do not treat LinkedIn and Meta as equivalent — B2B professional targeting on LinkedIn is
  job-function/seniority-first; Meta is behavioral/lookalike-first.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and ICP loaded (or fallback path taken) before any audience is defined?
- All three layers present for each platform, or explicit rationale for why a layer is omitted?
- Every retargeting layer has a defined window and stated exclusion list?
- Customer and trial-user exclusions applied at every layer?
- `data-qa-measurement-gotcha-checker` called; measurement gate result shown in spec header?
- Platform-specific signal translation completed (not copy-pasted across Google/Meta/LinkedIn)?
- Downstream handoffs to ad-copy, bids, and LinkedIn campaign spec named explicitly?
- Spec saved to `./targeting/` unless user requested inline?
