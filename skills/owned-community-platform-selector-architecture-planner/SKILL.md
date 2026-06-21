---
name: owned-community-platform-selector-architecture-planner
description: >
  Takes community goals, audience profile, and budget → produces a scored platform comparison
  (Slack / Discord / Circle / Khoros / Bettermode / Mighty Networks / self-hosted options),
  a recommended channel/space structure, a role-and-permission hierarchy, and a practical
  automation + moderation setup guide. Anchors every decision in the Jobs-to-Be-Done each
  platform serves, not vendor marketing. Prevents the most common L34 mistake: choosing a
  platform for its brand fit rather than its member behavior model. Use whenever someone asks
  "which community platform should we use," "set up our community," "Slack vs Discord vs
  Circle," "design our community structure," "community architecture," "owned community
  strategy," or hands over a community brief and asks where to build it.
---

# Owned Community Platform Selector & Architecture Planner

The platform choice is irreversible at scale. The channel structure drives every engagement habit your members form in week one. Get both wrong and no amount of content or moderation recovers it — you just build a ghost town with better branding.

This skill makes the platform selection and architecture decisions rigorous: a scored comparison tied to your actual community Jobs-to-Be-Done, a channel map that doesn't balloon into 47 unused spaces, a role ladder with real incentives, and an automation spec you can hand directly to an ops person.

It does not produce community content, run acquisition campaigns, or write member comms — those live in `community-led-acquisition-advocacy-amplifier` and lifecycle skills.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's voice, ICP, positioning, and proof. Platform and architecture decisions must reflect the brand's member profile, not a generic SaaS community default.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no brand.md exists, ask the user for [community purpose statement, target member persona (role/company type/awareness stage), monthly budget for the platform, and whether this is a product/support community, a peer-learning community, or a brand/advocacy community].
- **`icp-persona-builder`** *(optional)* — if no member persona exists yet, call this to produce one before scoring platforms; the behavioral dimensions it outputs (async vs sync preference, technical fluency, prior community memberships) directly affect the platform score.
- **`channel-strategy-selector`** *(optional)* — useful upstream when the broader marketing channel mix is undecided; community may not be the right investment yet.
- **`martech-stack-auditor-mapper`** *(optional)* — call when the user has an existing stack and needs to confirm the chosen platform integrates cleanly (Zapier/Make touchpoints, CRM sync, SSO).
- **`go-no-go-gate-evaluator`** *(optional)* — use at the end to run a formal gate check before committing the platform and build plan.

---

## How a run works

```
Step 0  Load the brand           ──► call brand-brain; extract member profile + voice
Step 1  Clarify community intent ──► 5 diagnostic questions (batched, one ask)
Step 2  Score platforms          ──► weighted rubric against the 6 JTBD dimensions
Step 3  Recommend + justify      ──► pick, explain, surface the biggest risks
Step 4  Design the architecture  ──► channel map + role hierarchy + automation spec
Step 5  Produce the deliverable  ──► save to ./community/[brand-slug]-community-plan.md
```

---

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill. Extract from the returned digest: ICP role/company profile, awareness tendency, brand voice adjectives, and any existing community signals (e.g., active Slack, Facebook Group, Discord mentioned in proof points). These feed directly into the member behavior model in Step 2.

Obey voice and banned-words in all naming, copy, and role label suggestions.

---

### Step 1 — Clarify community intent (one batched ask)

If any of these five are not already clear from the brief or brand-brain, ask them together — never as a drip:

1. **Primary JTBD:** networking/peer-learning, product support/success, brand advocacy/superfans, or all three? (Rank them if multiple.)
2. **Member profile:** who is the ideal active member — role, company size, technical level, and is this members-only or open?
3. **Budget:** monthly platform budget (zero / <$100 / $100–500 / $500+ / enterprise — ballpark is enough).
4. **Stack context:** are there existing tools the community must integrate with (CRM, helpdesk, SSO, Zapier/Make, analytics)?
5. **Timeline + team:** hard launch date and how many people will manage the community (headcount and hours/week)?

Never proceed past Step 1 without knowing the primary JTBD and approximate budget.

---

### Step 2 — Score platforms on the JTBD × Fit rubric

Use the **Community Platform JTBD Matrix** — a weighted 6-dimension scoring rubric. Score each shortlisted platform 1–5 per dimension; multiply by weight; sum.

#### The 6 dimensions and their weights

| Dimension | Weight | What it tests |
|---|---|---|
| **Member behavior model fit** | 30% | Does the platform's async/sync/threaded/channel paradigm match how your members naturally communicate? (Slack = async chat; Discord = real-time + gaming DNA; Circle/Bettermode = async structured discussion; Mighty Networks = course-first) |
| **JTBD alignment** | 25% | Does the platform's feature set natively serve the primary JTBD? (Support communities need ticketing hooks; peer-learning needs course/resource rooms; advocacy needs member profiles + rep systems) |
| **Growth ceiling & moderation overhead** | 20% | Can it handle 10× current size without breaking UX or requiring a full-time mod army? |
| **Integration depth** | 15% | Native or low-friction connectors to the required stack (CRM, helpdesk, SSO, event tools) |
| **Cost at target scale** | 10% | All-in monthly cost at realistic MAU target vs budget |

Score shortlist of 3–4 platforms maximum. Don't score 8 options — that's a vendor RFP, not a community decision.

**Platform baseline behaviors to use in scoring** (facts as of early 2026 — mark anything uncertain `[verify]`):

- **Slack** — async messaging-first; channels not spaces; free tier caps 90-day history [verify current]; best fit for B2B professional peer networks where members already live in Slack; scales poorly past ~2k active members without fragmentation
- **Discord** — real-time + async hybrid; server/channel/thread model; voice channels; originally gaming, now creator/crypto/developer-heavy; free with Nitro revenue model; weak native member directory; moderation burden high at scale
- **Circle** — purpose-built community SaaS; spaces + courses + events + member profiles; no persistent chat (async threads); $89–$399/mo [verify]; strong for creator/education communities; native Zapier integration; no SSO on lower tiers [verify]
- **Bettermode** — white-label community platform; strong SSO + API; post/topic/Q&A/ideation spaces; better product-community and B2B SaaS fit than Circle for companies needing deep CRM integration; pricing varies [verify]
- **Mighty Networks** — course + community hybrid; best for knowledge-creator businesses; weaker CRM integration; mobile-first; member-pays-to-join model built in
- **Self-hosted (open source: Discourse, Forem, Vanilla)** — maximum control, no vendor lock-in; highest ops cost and setup time; Discourse is the gold standard for developer/technical communities; zero ongoing platform cost but infra + dev cost

---

### Step 3 — Recommend and surface risks

State the winner clearly. Don't hedge with "it depends on your goals" — you just scored against their goals.

Format:

```
Recommendation: [Platform]
Score: [X/100]  Runner-up: [Platform, Y/100]

Why it wins: [2–3 sentences tied to JTBD + member model]
Biggest risk: [the one thing most likely to bite them]
Condition that flips the choice: [e.g., "If you add SSO as a hard requirement, Bettermode wins"]
```

If the score gap between #1 and #2 is under 8 points, say so and note what would tip it.

---

### Step 4 — Design the community architecture

#### Channel / Space structure

Apply the **Minimum Viable Channel Rule**: launch with the fewest spaces that cover all member JTBDs. Resist the instinct to pre-create every topic. Under-used channels signal a dead community faster than anything.

Standard starter architecture (adapt to JTBD and platform):

```
Orientation layer (1–2 spaces)
  → #start-here / Welcome Room    (rules, how-to-use, member intro prompt)
  → #announcements                (admin-only post, all-member read)

Core JTBD layer (2–4 spaces — platform-specific)
  → [Primary topic or product area]
  → [Secondary topic or use-case]
  → [Ask/Help / Q&A]

Social layer (1 space — unlock after launch, not before)
  → #off-topic / Watercooler      (only open this when >100 active members; don't pre-create)

Resource layer (1 space)
  → #resources / Library          (pinned, curated links — not a dumping ground)
```

Name every channel/space using the brand's voice adjectives from brand-brain. Note character limits where they apply (Slack channel names: ≤80 chars, lowercase-hyphenated).

#### Role and permission hierarchy

Three-tier ladder is the minimum; four-tier is the practical ceiling for most communities under 5k members.

| Tier | Label (customize to brand voice) | How earned | What they can do beyond members |
|---|---|---|---|
| Guest / Lurker | varies | signup | read-only or limited post |
| Member | varies | intro post OR email verify | post, react, DM |
| Contributor / Regular | varies | activity threshold (e.g., 10 posts + 30 days) | post in contributor-only space, badge, mention in digest |
| Moderator / Champion | varies | admin invite | pin, warn, approve posts, host events |

Name tiers to reflect the brand's community identity — not generic "Bronze/Silver/Gold." Use brand voice adjectives as a naming seed.

#### Automation setup (Zapier / Make / native)

Specify the 4–5 automations that do the most work. Don't list 20 — list the ones that prevent member drop-off in the first 30 days.

Mandatory automations for most communities:

1. **Welcome DM** — trigger: new member joins → send personalized welcome DM within 60 seconds with one action ("Post your intro in #start-here")
2. **Intro acknowledgment** — trigger: post in #introductions → react + thread reply from admin/bot with a relevant space recommendation
3. **Inactivity nudge** — trigger: member active in first 7 days, then no activity for 14 days → single re-engagement DM (not email — keep it in-platform)
4. **Weekly digest** — trigger: weekly cron → compile top posts + upcoming events → post to #announcements
5. **Escalation flag** — trigger: keyword list (help / broken / urgent / refund) in non-support channels → notify a moderator

For each automation: specify the trigger, the action, the tool (native / Zapier / Make), and the data fields required.

---

### Step 5 — Save the deliverable

Save the full output to `./community/[brand-slug]-community-plan.md`. Confirm the path to the user.

---

## Principles (Non-Negotiable)

- **JTBD before features.** Score on what members need to do — not on which platform has the best marketing site or the prettiest UI.
- **Brand-brain first.** No architecture decision before the brand's member profile is loaded. Voice, ICP role, and community purpose all feed the naming and structure.
- **Minimum viable channels.** Over-channeling kills communities. Launch sparse; expand on evidence.
- **Score, don't hedge.** Give a recommendation. "It depends" is not an architecture deliverable.
- **Real data or `[verify]`.** Platform pricing and feature limits change; mark anything unconfirmed and tell the user to confirm before signing a contract.
- **Three-tier roles minimum.** Flat permission structures remove the progression incentive. Design the ladder before launch.
- **Automate the 30-day drop-off window.** The first 30 days determine lifetime retention. The five automations above are not nice-to-haves.

---

## What Not to Do

- Don't recommend a platform because the user said "we like Slack's vibe" — run the rubric.
- Don't create more than 4–5 channels at launch; tell the user to hold the rest for Phase 2.
- Don't name roles "Bronze/Silver/Gold" — those are loyalty-program conventions, not community identity.
- Don't skip the integration audit if the user has an existing martech stack — a platform that doesn't connect to the CRM creates a data silo that kills attribution.
- Don't confuse owned community with social media community (Facebook Groups, LinkedIn Groups) — they have different retention and data-ownership dynamics; flag the distinction if the user conflates them.
- Don't produce the plan without knowing the primary JTBD — community purpose drives every structural decision.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called; member profile + voice loaded (or fallback path taken)?
- Primary JTBD and budget confirmed before scoring?
- Exactly 3–4 platforms scored using the 6-dimension rubric; weights applied correctly?
- Recommendation stated clearly (no hedging); biggest risk named; flip condition noted?
- Channel map launches with ≤6 spaces; social layer held back for Phase 2?
- Role hierarchy has ≥3 tiers; tiers named in brand voice?
- Exactly 4–5 automations specified with trigger + action + tool + data fields?
- All platform pricing and limits either verified or marked `[verify]`?
- Deliverable saved to `./community/[brand-slug]-community-plan.md`?
