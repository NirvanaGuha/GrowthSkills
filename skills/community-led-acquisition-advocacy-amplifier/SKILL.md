---
name: community-led-acquisition-advocacy-amplifier
description: >
  Turns an owned community — Slack workspace, Discord server, Circle space, Facebook group,
  forum, or in-product community — into a measurable low-CAC acquisition channel. Takes
  community size, member persona, top member posts, and existing engagement data and returns:
  (1) a referral incentive structure calibrated to the community's motivation profile,
  (2) shareable on-brand assets members will actually use, and (3) a UGC repurposing plan
  that converts member success stories, testimonials, and milestone posts into social proof
  the brand can publish. Built on the SPACES framework (Success/Product/Acquisition/Community/
  Expansion/Service) to distinguish which community activities drive acquisition versus retention.
  Calls brand-brain for voice and proof, icp-persona-builder if persona is thin, proof-vault
  for validated social proof, and ugc-creator-brief-writer for creator-brief output. Use when
  the user says "turn my community into a growth channel," "get members to refer," "leverage
  UGC from community," "community-led growth," "advocacy program," "amplify member stories,"
  "referral incentive for community," "repurpose member posts," or hands over community data
  and asks how to grow from it.
---

# Community-Led Acquisition & Advocacy Amplifier

An engaged community is the lowest-CAC acquisition channel most brands under-exploit. This skill systematizes the move from passive community to active growth engine — with a referral incentive structure members will act on, share-ready assets they'll actually forward, and a UGC repurposing plan that extracts durable social proof from posts that would otherwise disappear into a feed.

It does not build the community or platform from scratch. For platform selection and architecture, call `owned-community-platform-selector-architecture-planner`. This skill assumes the community exists and focuses on turning its latent advocacy into attributed acquisition.

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads voice, banned words, offer mechanics, real proof, ICP, and positioning. Do not write any asset or incentive copy before it returns.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no brand.md exists, ask the user for [brand voice adjectives, ICP role/pain, offer mechanics and referral-eligible plan(s), and any existing social proof].
- **`icp-persona-builder`** — call if the member persona is thin (fewer than 3 dimensions: role, pain, motivation). Returns persona spec this skill uses to calibrate incentive type.
- **`proof-vault`** — supplies verified testimonials, case stats, and G2/Capterra quotes to embed in shareable assets. Synthesize inline if absent.
- **`ugc-creator-brief-writer`** — when the output calls for a formal UGC creator brief (ambassador program, structured testimonial campaign), delegate; do not rebuild here.
- *(optional)* `referral-program-brief-builder` — if the user wants a complete program spec rather than the incentive structure this skill produces.
- *(optional)* `content-repurposer-atomizer` — for heavy-lift repurposing of a post or transcript into multiple formats.
- *(optional)* `social-proof-screenshot-styler` — to style raw testimonial screenshots into publishable brand cards.

---

## How a run works

```
Step 0  Load the brand          ──► call brand-brain (mandatory)
Step 1  Profile the community   ──► inputs → SPACES classification + motivation scan
Step 2  Build the incentive     ──► referral structure calibrated to motivation profile
Step 3  Create the assets       ──► share-ready copy + asset briefs
Step 4  UGC repurposing plan    ──► identify, source, adapt, publish pipeline
Step 5  Activation roadmap      ──► 4-week launch sequence
Step 6  Measurement frame       ──► 3 leading + 1 north-star metric
```

---

## Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns voice adjectives, banned words, offer mechanics, real proof, ICP, and positioning. Every asset this skill writes uses that voice and only confirmed proof (everything else gets `[verify]`). Do not proceed until brand-brain returns.

---

## Step 1 — Profile the community

Before designing incentives, classify what the community actually does and why members stay. Use the **SPACES framework** (Jonathon Colman / CMX Hub):

| SPACES bucket | What it means | Acquisition relevance |
|---|---|---|
| **S**upport | Members help each other troubleshoot | Low direct; high trust-building |
| **P**roduct | Feedback, feature requests, beta testing | Low direct; strong proof pipeline |
| **A**cquisition | Members recruit prospects | Primary target for this skill |
| **C**ontent | Members create/amplify brand content | Direct amplification lever |
| **E**ngagement | Ongoing relationship, NPS, retention | Multiplier — keeps advocates active |
| **S** ervice / Success | Customer success, onboarding | Converts to case studies |

Identify which 1–2 buckets dominate the community today, then identify which are under-leveraged. The gap between current state and **Acquisition** + **Content** is the program's scope.

**Inputs to collect** (ask if not provided):

- Community platform and approximate member count
- Member persona (role, company size, pain — call `icp-persona-builder` if thin)
- Top 3–5 posts by engagement (paste text or summarize)
- Existing referral or affiliate program (Y/N; if Y, current incentive)
- Product's referral-eligible tiers or plans
- Any existing UGC policy or brand ambassador activity

**Motivation scan** — from the posts and persona, identify the dominant motivation type: **status/recognition** (most common in professional communities), **financial** (affiliate/discount), **altruistic/mission**, or **reciprocity** (I help because I was helped). This gates incentive type selection in Step 2.

---

## Step 2 — Referral Incentive Structure

Apply **incentive-motivation fit** — the wrong incentive type tanks referral conversion even at high values:

| Dominant motivation | Best incentive type | Avoid |
|---|---|---|
| Status / recognition | Public leaderboard, badge, feature spotlight, "Charter Member" title | Cash (feels transactional) |
| Financial | Two-sided cash/credit, affiliate commission, discount stacking | Badge-only (feels patronizing) |
| Altruistic / mission | Donation match, community contribution, featured impact story | Anything that feels like marketing |
| Reciprocity | Give-one-get-one (referrer + referee both win), team/cohort credits | One-sided rewards |

**Incentive structure output** (deliver for each program tier):

```
Program name: [brand-voice name, not "Referral Program"]
Trigger:       [what action starts the clock — signed up via link, paid, activated feature X]
Referrer gets: [specific value + when]
Referee gets:  [specific value + when]
Qualification: [what the referee must do to unlock the reward]
Cap:           [max reward per member per period, if any]
Expiry:        [if credits/coupons, when they expire]
Incentive type: [status/financial/altruistic/reciprocity]
Motivation fit rationale: [1 sentence]
```

If the brand has no referral-eligible offer at all, flag it — don't invent a mechanism that doesn't exist. Recommend `referral-program-brief-builder` for a full program spec.

---

## Step 3 — Shareable Assets

Members share when the asset (a) makes them look good, (b) is trivially easy to forward, and (c) feels genuine — not like a flyer they've been handed. Produce:

**A. Referral invite messages** (3 variants per channel the community uses):
- Direct DM / 1:1 ("I've been using X for Y…")
- Community announcement post (channel #general or equivalent)
- External social (LinkedIn/Twitter — platform-appropriate, no markdown on LinkedIn)

Each variant: on-brand voice, specific to a real use case from the top member posts, max 150 words, one referral link placeholder `[REF_LINK]`.

**B. Social proof share cards** (copy brief, not design):
- Format: outcome stat + quote + logo watermark (brief for `social-proof-screenshot-styler`)
- 3 card concepts using only confirmed proof from `proof-vault` or brand-brain digest; unconfirmed tagged `[verify]`

**C. Milestone celebration templates** (2 variants):
- Triggered when a member hits a product milestone worth sharing (e.g., first campaign live, 10k subscribers reached, first conversion)
- Copy: community congratulations post + optional social share prompt, written to feel earned not marketed

---

## Step 4 — UGC Repurposing Plan

Community posts are ephemeral by default. This plan makes them durable:

**Identify tier:**
- **Tier 1 (publish-ready):** Testimonials with a specific outcome stat, before/after, or named use case → route to `social-proof-screenshot-styler` + `proof-vault`
- **Tier 2 (requires adaptation):** Strong story but informal → light edit pass (preserve voice, add permission tag), repurpose to blog quote, LinkedIn post, or case study
- **Tier 3 (signal only):** Pattern-level insights (recurring pain, objection) → feed into `voice-of-customer-mining-pipeline` or `reddit-forum-listening-digest`

**Permission protocol** (required — do not skip):
> Every piece of member content that leaves the community requires explicit permission. For Tier 1/2: direct message the member, name the specific post, name the channels it will appear in, confirm before publishing. Do not assume community TOS grants reuse rights for brand marketing. This is both legal hygiene and trust protection.

**Repurposing cadence output:**

```
Tier 1 pipeline: [where to publish, what format, with which brand wrapper]
Tier 2 pipeline: [editing pass brief + destination]
Permission DM template: [ready to send, editable]
Monthly target: [how many Tier 1/2 pieces is realistic given community activity]
```

---

## Step 5 — 4-Week Activation Roadmap

```
Week 1 — Seed: identify top 5 advocates (high engagement, success stories), brief them 1:1,
         give early access / status recognition, collect first testimonials
Week 2 — Launch: announce program in community (use referral invite template A),
         post first social proof card, activate referral links for seed advocates
Week 3 — Amplify: first milestone celebration posts live, first Tier 2 repurposed content
         published, leaderboard (if status incentive) visible to community
Week 4 — Measure + iterate: referral link click-through, new sign-ups attributed,
         UGC pieces collected; diagnose incentive fit; adjust copy if conversion < 2%
```

Save to `./community/[brand-slug]-activation-roadmap.md` on request.

---

## Step 6 — Measurement Frame

Three leading indicators (weekly), one north-star (monthly):

| Metric | Leading / North-star | Why it matters |
|---|---|---|
| Referral link click-through rate | Leading | Measures message + incentive resonance before conversion |
| Advocate activation rate (% members who shared ≥1 link) | Leading | Measures program reach vs. passive awareness |
| UGC pieces collected (Tier 1 + 2) | Leading | Measures pipeline health for social proof |
| Community-attributed new signups / MRR | North-star | The CAC story for leadership |

All four tracked in a simple sheet — template in `./community/[brand-slug]-community-metrics.csv` on request.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No asset ships before brand-brain returns the active brand. Voice and banned words are hard overrides.
- **Incentive-motivation fit before incentive size.** A $50 cash reward to a status-motivated community member will underperform a leaderboard badge. Match the type first.
- **Permission before publish.** Every piece of member content requires explicit opt-in before brand use. No exceptions.
- **Compose, don't duplicate.** `ugc-creator-brief-writer`, `social-proof-screenshot-styler`, `proof-vault`, `referral-program-brief-builder` each own their domain — delegate, don't rebuild.
- **Real proof or `[verify]`.** Never invent outcome stats, member counts, or conversion benchmarks.
- **SPACES gates scope.** Acquisition and Content buckets are the target; if the community is pure Support, say so — don't pretend it's a referral engine yet.

---

## What Not to Do

- Don't write referral assets before `brand-brain` returns the brand digest.
- Don't design a referral incentive without knowing which motivation profile dominates — wrong type kills conversion regardless of value.
- Don't repurpose member content without a permission protocol step. Skipping this damages community trust and can create legal exposure.
- Don't generate a referral mechanism when no referral-eligible product tier exists — flag the gap and route to `referral-program-brief-builder`.
- Don't conflate community size with advocacy potential. Lurker-heavy communities need an activation step before any referral program lands.
- Don't produce a bloated asset library — three focused, highly usable variants beat ten generic ones.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or fallback path followed) before any copy?
- Voice + banned words honored; all proof confirmed or marked `[verify]`?
- SPACES classification completed; dominant bucket identified; acquisition/content gap named?
- Incentive type matched to motivation profile with one-line rationale?
- Referral structure output includes trigger, both-sides reward, qualification, cap, expiry?
- Three share-ready asset variants per channel, grounded in real member posts (not generic)?
- UGC tiers defined; permission DM template included?
- 4-week roadmap present with concrete Week 1 seed action?
- Four metrics defined (3 leading, 1 north-star)?
- Sibling skills called (not rebuilt) for creator briefs, proof styling, persona gaps?
