---
name: partner-onboarding-checklist-generator
description: >
  Takes a partner type (affiliate, co-marketing, integration/tech, reseller, or influencer) plus
  product and deal details, and produces a step-by-step onboarding checklist with owner assignments,
  milestone gates, and a kickoff brief — so the first 30 days are structured, nothing falls through
  the cracks, and the partner is revenue-active as fast as possible. Organized with our MAPS lane scheme
  (Milestones, Assets, People, Systems) — a house checklist model, not an external framework — built to
  drive down the one metric that predicts partner survival: Time-to-First-Value (TTFV), the moment the
  partner drives their first real, attributed outcome. Composes brand-brain for voice/ICP, sales-partner-collateral-creator
  for talk tracks and one-pagers, and co-marketing-sponsorship-campaign-planner for joint launch
  activities — rather than rebuilding those from scratch. Saves a reusable checklist artifact per
  partner type. Use when the user says "onboard a partner," "partner checklist," "affiliate setup,"
  "integration onboarding," "new reseller," "partner kickoff," "what do we need to activate
  a partner," or hands over a partner deal brief and asks what happens next.
---

# Partner Onboarding Checklist Generator

Most partner relationships die in the first 30 days — not from a bad deal, but from a missing asset, an unassigned owner, or a system never provisioned. This skill structures the gap between "deal signed" and "partner actively driving revenue."

The metric that actually predicts whether a partner survives is **Time-to-First-Value (TTFV)** — the time from kickoff to the partner's first real, attributed outcome (first tracked conversion, first registered deal, first published listing, first live post). TTFV is a recognized customer-success metric ([Lincoln Murphy / Sixteen Ventures, "Customer Onboarding and TTFV"](https://www.sixteenventures.com/customer-onboarding-ttfv/)); we apply it to partner onboarding. The trap it names is the one partner programs fall into constantly: onboarding *completion* — "training done," "integration configured," "portal granted" — is an internal milestone, not value. First value is the partner saying "this is working." Every task here either pulls TTFV forward or it gets cut.

To make TTFV operational, we organize each checklist across four lanes — **MAPS**: Milestones, Assets, People, Systems. MAPS is a house checklist scheme, not an external framework; it is just the mnemonic we use so no onboarding silently drops one of the four dimensions that gate first value. Every output is checklist-first, owner-assigned, and gated at the moments that actually predict activation.

This skill generates the checklist, kickoff agenda, and launch readiness gate. It does not write the collateral, build the co-marketing plan, or produce the talk tracks — it delegates those to purpose-built siblings.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's voice, ICP, offer mechanics, and proof so every partner-facing asset and communication is on-brand. Does not reimplement brand resolution. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP description, core offer/pricing, and 3 voice adjectives before proceeding.
- **`sales-partner-collateral-creator`** — called in Phase 2 (Assets) to produce one-pagers, co-sell talk tracks, and Loom pitch script for the partner's reps. Compose, don't duplicate.
- **`co-marketing-sponsorship-campaign-planner`** — called for co-marketing partner types to plan joint launch activities, asset split, and shared calendar.
- **`campaign-brief-builder`** — called when the partner needs a launch campaign brief to take to their audience.
- **`lifecycle-journey-mapper`** — optional; used when the partner's end-customer journey needs mapping for integration or tech partners.
- **`sop-builder-reviewer`** — optional; called to produce a repeatable SOP for the onboarding process itself (useful when the same partner type recurs).

---

## How a run works

```
Step 0  Load the brand         ──► brand-brain (always first)
Step 1  Classify the partner   ──► affiliate | co-marketing | integration/tech | reseller | influencer
Step 2  Collect inputs         ──► 5 questions if not already provided
Step 3  Build the MAPS checklist
Step 4  Generate the kickoff brief + launch gate
Step 5  Delegate collateral/campaign tasks to siblings
Step 6  Save artifact
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. Wait for the returned digest before producing any partner-facing content. Obey voice + banned-words; use only real proof (mark anything unconfirmed `[verify]`).

**Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP description, core offer/pricing, and 3 voice adjectives before proceeding.**

### Step 1 — Classify the partner

Identify the partner type from the request or ask. The type governs which MAPS lanes are critical and what "first value" actually means for this partner:

| Type | Critical lanes | First-value event (TTFV target) | Indicative TTV `[verify]` |
|---|---|---|---|
| **Affiliate** | Assets (links, creatives) + Systems (tracking, payout) | First tracked, attributed sale | 7–14 days |
| **Co-marketing** | Milestones (joint campaign) + People (counterpart contacts) | Joint campaign live + first shared lead | 14–21 days |
| **Integration / Tech** | Systems (API keys, sandbox, webhook) + Assets (listing copy) | Listing published + first install/active connection | 21–30 days |
| **Reseller / VAR** | People (rep training) + Assets (sales kit) + Systems (portal access) | First deal registered | 14–21 days |
| **Influencer / Creator** | Assets (brief, links, codes) + Milestones (content deadlines) | First sponsored post live + first tracked click | 7–10 days |

The "Indicative TTV" column is a **planning estimate, not a measured benchmark** — every value is `[verify]` against the brand's own historical activation data before it goes in a plan or a partner promise. As a directional sanity check, structured channel-partner onboarding programs commonly target a 60–90 day window to first productivity and under 30 days to first deal registration `[verify]` ([introw, "Partner Lifecycle Management"](https://www.introw.io/blog/partner-lifecycle-management)); the per-type estimates above are tighter because these are lighter-weight partner types, not enterprise channel resellers. If you have no historical data, present the estimate explicitly as a target to validate, not a commitment.

### Step 2 — Collect inputs (if not supplied)

Ask only what's missing. Standard inputs:

1. **Partner name + type** — e.g. "Acme Widgets, affiliate"
2. **Deal terms summary** — commission rate, rev-share, joint offer, or integration scope
3. **Assigned internal owner** — the single accountable person on your side
4. **Partner's primary contact** — name, role, preferred channel
5. **Target activation date** — when the partner should be live and driving revenue

If the user provides a deal brief or contract summary, extract these fields from it rather than asking.

---

## MAPS — the four lanes (a house checklist scheme)

MAPS is the mnemonic we use to make sure no onboarding silently drops a dimension that gates first value — it is not a named industry framework, and you should never present it as one. Every checklist is organized across the four lanes. Populate each based on the partner type, then assign owners and set gate criteria. A lane is "done" only when the partner could not be blocked from first value by anything in it.

### M — Milestones

The sequenced events that define "activated." Each milestone must be binary (done / not done) and assigned to a single owner. Standard milestones by type:

- **All types:** Agreement signed → Kickoff call → System access granted → First live asset → First tracked conversion/action
- **Co-marketing add:** Joint campaign brief approved → Co-branded asset signed off → Launch date confirmed
- **Integration add:** Sandbox credentials provisioned → Integration test passed → Production credentials issued → Listing published
- **Reseller add:** Rep training completed → Deal registration system activated → First opportunity logged

### A — Assets

The collateral, links, and creative the partner needs to activate. Gap here is the most common reason a partner goes dark.

- **Affiliate:** Tracking links, creatives (banner sizes, email swipes), offer landing page URL, discount/coupon code, editorial guidelines. **Anti-gaming terms in writing before any link goes live:** no bidding on your branded keywords (PPC poaching), no coupon-extension/toolbar attribution, no self-referral, a defined cookie window, and a stated last-click-vs-first-click attribution rule. These are the disputes that kill affiliate programs in month two; settle them in the agreement, not after a clawback.
- **Co-marketing:** Co-branded one-pager, joint landing page copy, email copy, social assets, partner logo + brand guide
- **Integration:** API documentation link, sandbox credentials, webhook setup guide, marketplace listing copy + screenshots, support escalation path. **Spell out the sandbox contract:** which test data the partner gets, the sandbox-reset cadence (so they don't build against state that vanishes), published rate limits + the 429/backoff expectation, and the breaking-change notice period. "It worked in sandbox" failing in production is the classic integration-partner stall.
- **Reseller:** Sales deck (partner-branded), battlecard, pricing/discount matrix, demo environment access, customer-facing one-pager
- **Influencer:** Creator brief (UGC Brief Writer handles this), unique tracking link, product samples/access, posting schedule, FTC disclosure language

**Flag to call `sales-partner-collateral-creator`** for talk tracks + one-pager if not already built. Flag to call `co-marketing-sponsorship-campaign-planner` for joint campaign planning.

### P — People

The human connections that make a partnership work. Missing an escalation contact or skipping rep training is a quiet killer.

- Internal owner (single accountable; not a committee)
- Partner primary contact + technical contact (if integration type)
- Your support/success contact for the partner
- Executive sponsor (for strategic/enterprise partners)
- Training schedule and completion confirmation for reseller reps

### S — Systems

Every platform, credential, and integration the partner needs provisioned before Day 1.

- **All types:** Partner portal or shared workspace access (Notion page, Slack channel, shared Drive)
- **Affiliate:** Affiliate platform provisioned (Impact, ShareASale, PartnerStack, or custom), payout method configured, tracking test completed. **Set the payout-fraud guardrails before the first commission accrues:** a holdback/clearance period covering your return-and-refund window (so refunded orders don't pay out), a self-referral block keyed to the partner's own email/IP/payment instrument, a first-payout manual review, and a deduplication rule against your other paid channels so you don't pay an affiliate for a sale paid ads already bought. Decide whether commission is on net (post-refund, post-discount) or gross — in writing.
- **Integration:** Sandbox API keys, production API keys (issued only after the integration test passes — never reuse sandbox keys in prod), webhook endpoint registered with signature verification, documented rate limits + 429/backoff handling confirmed, and the sandbox-data-reset schedule shared so the partner builds against stable fixtures
- **Reseller:** CRM deal-registration portal, partner discount tier configured, demo environment
- **Co-marketing:** Shared campaign tracking UTMs built, shared content calendar, joint Slack channel

---

## Output format

### Checklist artifact

```markdown
# Partner Onboarding Checklist — [Partner Name] ([Type])
Brand: [slug] | Owner: [name] | Target activation: [date]

## Phase 1 — Agreement & Kickoff (Days 1–3)
- [ ] [Task] — Owner: [name] — Due: Day [N]
- [ ] ...
**Gate:** Kickoff call complete + deal terms confirmed in writing

## Phase 2 — Assets & Access (Days 3–10)
- [ ] [Task] — Owner: [name] — Due: Day [N]
- [ ] Call `sales-partner-collateral-creator` → one-pager + talk track for [partner reps / partner's audience]
- [ ] ...
**Gate:** All assets delivered + system access confirmed by partner

## Phase 3 — Training & Test (Days 10–20)
- [ ] ...
**Gate:** [type-specific: tracking test passed | rep certification passed at the team's defined threshold | integration test passed]

## Phase 4 — Launch & First Conversion (Days 20–30)
- [ ] ...
**Gate:** First tracked conversion / first attributed action / first content post live

## Ongoing Cadence
- [ ] [30/60/90-day check-in cadence] — Owner: [name]
- [ ] Refresh assets each quarter — Owner: [name]
```

### Kickoff brief (inline, ~150 words)

Agenda for the kickoff call: intro, deal terms review, asset handoff plan, system access timeline, first milestone target, escalation contacts, cadence agreement.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No partner-facing copy, messaging, or asset before `brand-brain` returns. Its voice + banned-words override everything.
- **One owner per task.** Shared ownership is no ownership. If the user hasn't named an owner, call it out — don't silently leave it blank.
- **Gates are binary.** A gate is a hard stop (pass/fail), not a progress percentage. If a gate can't be tested, rewrite it until it can.
- **Collateral is delegated, not duplicated.** Call the right sibling skill for talk tracks, co-marketing plans, and launch briefs. Don't rebuild that work here.
- **TTFV is the north-star, and completion is not value.** Every task either pulls Time-to-First-Value forward or it gets cut. "Training done," "integration configured," or "portal granted" is an internal milestone — the gate that matters is the partner's first real, attributed outcome.
- **MAPS is ours, not an authority.** Present MAPS as our house lane scheme, never as a named or established framework. TTFV is the one thing that *is* external — the metric is real and recognized — but the per-type day ranges and the MAPS lane scheme are ours.
- **Unconfirmed numbers are `[verify]`.** Activation windows differ wildly by program; every day-range is a planning estimate to validate against the brand's own data, never a fabricated benchmark and never a promise to the partner. The same rule binds this skill's own tables — no exceptions for our defaults.

## What Not to Do

- Don't write talk tracks, email sequences, or co-marketing plans here — call `sales-partner-collateral-creator` and `co-marketing-sponsorship-campaign-planner`.
- Don't produce any partner-facing copy before `brand-brain` returns.
- Don't leave any task without an owner assignment (even if the user must fill it in).
- Don't conflate onboarding with ongoing partner management — this skill covers Day 0 to activation gate; flag cadence items but don't build a full partner management program.
- Don't generate a generic checklist ignoring partner type — the MAPS lanes that matter vary significantly; adapt every output.
- Don't skip the launch gate — a checklist without a binary pass/fail condition is a wishlist.
- Don't call MAPS a framework, a methodology, or an industry standard — it is our house lane scheme. And don't invent a benchmark to fill the TTV column; if there's no historical data, label the number a target to validate.
- Don't let a gate fire on "completion" (training finished, integration configured) when no first-value event has occurred — that is the exact trap TTFV exists to catch.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or fallback applied) before any output?
- Partner type classified and the right MAPS lanes prioritized?
- All five inputs collected (partner name/type, deal terms, internal owner, partner contact, activation date)?
- Every task in the checklist has a single named owner and a due day?
- Every phase ends with a binary gate criterion, and the final gate is a real first-value event (not "completion")?
- The partner type's first-value event named, and every task traced to pulling TTFV toward it?
- `sales-partner-collateral-creator` flagged for talk tracks/one-pager where relevant?
- `co-marketing-sponsorship-campaign-planner` flagged for co-marketing partner types?
- Type-specific guardrails included (affiliate anti-gaming + payout-fraud terms; integration sandbox/rate-limit/key-handling) where relevant?
- Checklist saved to `./partners/[partner-slug]-onboarding.md`?
- MAPS presented as our house scheme (not a named framework); every TTV day-range marked `[verify]` as a planning estimate, not a measured benchmark or a promise?
