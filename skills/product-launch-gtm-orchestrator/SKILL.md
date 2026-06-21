---
name: product-launch-gtm-orchestrator
description: >
  Product spec + target segment + launch date → a complete, ready-to-ship GTM kit in ONE run:
  positioning + messaging matrix, launch + nurture emails, press release with social/blog
  amplification, landing/product page copy, social posts, a launch deck, a UTM-tagged channel
  plan, a launch timeline with owners, and a go/no-go checklist. This is a flagship ORCHESTRATOR —
  a growth team in a box — that CHAINS the library's specialist skills end-to-end instead of
  reimplementing any of them. It calls `brand-brain` first for voice/ICP/offer/proof, runs
  `gtm-launch-planner` to size and scaffold the launch, runs `positioning-messaging-architect`
  for the message spine every asset inherits, fans out to the asset writers (press, landing page,
  email, social, deck), then runs `campaign-qa-launch-checklist-generator` as the final gate.
  Use when the user says "launch this product/feature," "build our GTM kit," "we ship on [date] —
  give me everything," "full go-to-market package," "product launch plan AND the assets," or hands
  over a product spec + a launch date and wants the whole campaign out the door, not one piece.
  Not for a single asset (call that specialist directly) and not for paid-only campaigns (use
  full-campaign-launch-orchestrator).
---

# Product-Launch GTM Orchestrator

You ship on a date. This produces the whole go-to-market kit to hit it. Give it a product spec, the segment you're launching to, and the launch date — it returns a positioning spine, a messaging matrix, the launch email sequence, a press release with its social + blog amplification, the landing/product page copy, launch-day social posts, a launch deck, a UTM-tagged channel plan, a launch timeline with owners, and a go/no-go checklist. One run. A buyer points at this and says *this replaces a junior marketer's week.*

This skill **orchestrates** — it does not re-write the press release or re-derive the brand voice itself. Every stage is a sibling skill that already exists; this skill decides the order, carries the handoff between stages, enforces the quality gates, and compiles the output into one bundled kit. If the product isn't ready to launch, it says so before producing assets — it does not paper over a missing offer with a press release.

---

## Skills this calls

The pipeline, in invocation order. Each is a real skill — **invoke it via the Skill tool; never reimplement its work.**

- **`brand-brain`** (required, Stage 0) — resolves the active brand and returns voice, banned words, ICP, offer mechanics + destination URLs, real proof, positioning. The spine all downstream copy inherits.
- **`gtm-launch-planner`** (Stage 1) — sizes the launch S/M/L and returns the launch plan: audience, channel mix, timeline, RACI, go/no-go gates. This is the *scaffold* every later stage fills.
- **`positioning-messaging-architect`** (Stage 2) — turns the spec + plan into the launch message hierarchy: one-liner, value props per persona, the messaging matrix. The single source of message truth for all assets.
- **`press-release-social-blog-amplification-pack`** (Stage 3, assets) — the announcement: press release + exec LinkedIn post + X thread + owned-media blog post.
- **`landing-product-page-copy-writer`** (Stage 3, assets) — the launch landing/product page copy, on the new positioning.
- **`welcome-onboarding-email-sequence-builder`** + **`full-email-push-asset-builder-copy-responsive-html`** (Stage 3, assets) — the launch announcement email + the post-signup sequence, rendered to responsive copy/HTML.
- **`social-content-calendar-builder`** (+ `linkedin-post-writer`, `x-thread-writer`, `short-form-video-script-writer`) (Stage 3, assets) — the launch-window social schedule and the posts that fill it.
- **`deck-presentation-writer`** (Stage 3, assets) — the internal/sales launch deck off the messaging matrix.
- **`utm-parameter-bulk-builder`** (Stage 4) — UTM-tags every destination link across channels so attribution works on day one.
- **`campaign-qa-launch-checklist-generator`** (Stage 5, gate) — the final pre-launch QA: copy, creative, UTMs, targeting, tracking, legal — pass/fail per item.

*Optional, pulled in when the plan calls for them:* `cta-variant-generator` (per-asset CTAs), `icp-persona-builder` (new/narrow segment), `proof-vault` (proof microcopy), `headline-hook-generator`, `ai-image-generator-on-brand-assets` (hero/OG assets), `content-qa-reviewer` (per-asset copy review), `de-slop-humanize-pass` (final humanization), `pre-mortem-post-mortem-generator` (L-tier risk pass), `media-podcast-pitch-crafter` (PR-heavy launches), `push-notification-copy-generator` (if the brand ships push). Compose inline only if a named skill is genuinely unavailable.

---

## How a run works

```
Stage 0  Brand context   ──► call brand-brain (always first; blocks everything)
Stage 1  Plan + size     ──► call gtm-launch-planner → S/M/L plan, timeline, RACI, go/no-go
Stage 2  Message spine   ──► call positioning-messaging-architect → matrix every asset inherits
Stage 3  Asset wave      ──► fan out the writers (press · landing · email · social · deck)
         └─ QA loop       per asset: content-qa-reviewer → PASS keep / REVISE loop (max 2)
Stage 4  Tracking        ──► call utm-parameter-bulk-builder → tag every link
Stage 5  Launch gate     ──► call campaign-qa-launch-checklist-generator → go / no-go
Stage 6  Compile + HUMAN ──► bundle the kit to ./gtm-kits/[slug]-[date]/ ; present go/no-go
```

### Stage 0 — Load brand context (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. Wait for its digest — voice adjectives, banned words, ICP + awareness tendency, offer mechanics + destination URLs, real proof, positioning — and the path to `brand.md`. **Produce nothing downstream until it returns.** Its voice and banned words are hard overrides for every later stage; pass the digest into each child skill so none of them re-derives or re-asks for it.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly. If none exists, ask the user to install `brand-brain` (preferred) or answer a 4-question mini-setup (what it is · ICP + awareness · offer mechanics + destination URLs · 3 voice adjectives + banned words), then proceed. Always prefer the call.

### Stage 1 — Plan and size the launch

Gather the three inputs the kit needs: **product/feature spec**, **target segment**, **launch date**. Ask only for what's missing (batch the questions). Then **call `gtm-launch-planner`**, passing the brand digest + spec + segment + date. It returns the launch tier (S/M/L), audience, channel mix, timeline, RACI, and go/no-go gates.

The tier sets the asset scope so you don't over- or under-build:

| Tier | Launch shape | Assets the kit produces |
|---|---|---|
| **S** | single-channel drop, < 2 weeks | landing page section + 1 announcement email + 2–3 social posts + UTMs + checklist |
| **M** | coordinated multi-channel, 2–6 weeks | + press/amplification pack, full email sequence, social calendar, launch deck |
| **L** | full-market program, 6+ weeks | + pre-mortem, media pitches, on-demand/event tie-ins, staged-rollout timeline |

If the planner's go/no-go surfaces that the product isn't launch-ready (no offer, no proof, no clear audience), **stop and report that** with what must be true first — don't generate assets for a launch that can't happen.

### Stage 2 — Lock the message spine

**Call `positioning-messaging-architect`**, passing the brand digest + the spec + the launch plan's locked audience. It returns the launch one-liner, value props per persona, proof mapping, and the **messaging matrix**. This is the contract for Stage 3: **every asset writer receives this matrix and must inherit it** — same one-liner, same value props, same proof. This is what keeps a press release, a landing page, and an email all saying the same thing in the same voice. Do not let any downstream stage invent a new positioning.

### Stage 3 — Asset wave (fan out, then QA each)

Run the asset writers the tier calls for. Independent assets can be generated in the same wave; each gets **the brand digest + the messaging matrix + the relevant slice of the plan** as input so it never re-derives context:

- **Announcement** → `press-release-social-blog-amplification-pack` (press release + LinkedIn + X thread + blog).
- **Page** → `landing-product-page-copy-writer` (hero, sections, CTAs via `cta-variant-generator`).
- **Email** → `welcome-onboarding-email-sequence-builder` for the sequence, rendered via `full-email-push-asset-builder-copy-responsive-html`.
- **Social** → `social-content-calendar-builder` to schedule, then `linkedin-post-writer` / `x-thread-writer` / `short-form-video-script-writer` to fill it.
- **Deck** → `deck-presentation-writer` off the messaging matrix.

**Per-asset QA loop (the gate that makes this trustworthy).** After each writer returns, run **`content-qa-reviewer`** on its output. If it returns **PASS**, keep the asset. If it returns **REVISE**, send the specific notes back to the *same* writer skill and regenerate — **max 2 revise loops per asset**. If an asset still fails after two loops, keep the best version, flag it `⚠ needs human edit` in the kit, and move on (never block the whole launch on one stubborn asset). Optionally run `de-slop-humanize-pass` on long-form copy before final QA.

### Stage 4 — Tag every link

**Call `utm-parameter-bulk-builder`** with the channel plan + every destination URL the assets point to. Replace bare links in the assets with their UTM-tagged versions so attribution works from the first click. A launch with untagged links is a launch you can't measure.

### Stage 5 — The launch gate (go / no-go)

**Call `campaign-qa-launch-checklist-generator`**, passing the assembled kit + the channel plan. It returns a pass/fail checklist across copy, creative, UTMs, targeting, tracking, and legal. Roll its result up into a single **GO / NO-GO** verdict: GO only if every blocking item passes and no asset is still `⚠ needs human edit` on a critical channel. List any non-blocking flags as "ship-with-caveats."

### Stage 6 — Compile and hand to the human

Compile everything into one bundled kit under a **project-relative path** (leading `./` = the user's project, NOT the skill folder):

```
./gtm-kits/[brand-slug]-[launch-date]/
  00-launch-plan.md          (from gtm-launch-planner: tier, timeline, RACI, go/no-go)
  01-messaging-matrix.md     (from positioning-messaging-architect — the spine)
  02-press-release.md        + 02-amplification-linkedin-x-blog.md
  03-landing-page-copy.md
  04-email-sequence/         (announcement + nurture, copy + responsive HTML)
  05-social-calendar.md      + 05-posts/ (linkedin, x, short-form scripts)
  06-launch-deck-outline.md
  07-utm-link-sheet.csv
  08-launch-checklist.md     (the go/no-go gate result)
  README.md                  (the run summary + GO/NO-GO + what needs a human)
```

Then **present to the human**: the GO/NO-GO verdict, the timeline with the next 3 dated actions and their owners, anything flagged `⚠ needs human edit`, and any `[verify]` claims. **The human approves the launch — this skill never declares a launch live.** Offer to regenerate any single asset or to escalate the tier.

---

## Principles

- **Brand-brain first, always.** No plan, no message, no asset before `brand-brain` returns. Its voice + banned words override every child skill.
- **One message spine, inherited by all.** The Stage-2 messaging matrix is the contract. Every asset says the same thing in the same voice — that consistency is the whole point of orchestrating instead of writing piecemeal.
- **Orchestrate, don't reimplement.** Each stage is a real sibling skill invoked via the Skill tool. This skill owns order, handoff, gates, and compilation — nothing else.
- **Gate every asset; never self-approve.** Authoring and review are separate passes: a writer produces, `content-qa-reviewer` judges, the loop closes. Don't approve your own copy.
- **Fail loudly on a non-launch.** If the planner says the product isn't ready, stop and say what must be true first. Don't manufacture assets for a launch that can't happen.
- **Truth discipline.** Use only real proof from `brand-brain`/`proof-vault`; every unconfirmed number, date, or claim is marked `[verify]`. Never invent customers, metrics, quotes, or press coverage.
- **The human ships.** This produces a GO/NO-GO recommendation and a kit. A person approves and launches.

## What not to do

- Don't write any asset before `brand-brain` and the Stage-2 messaging matrix exist.
- Don't reimplement a stage's work (press release, landing copy, UTMs, QA) — call the skill that owns it.
- Don't let two assets drift onto different positioning; if a writer deviates from the matrix, send it back, don't reconcile by hand.
- Don't loop a failing asset forever — max 2 revise passes, then flag `⚠ needs human edit` and move on.
- Don't ship untagged links, fabricated proof, or a launch the QA gate failed on a blocking item.
- Don't save the kit inside the skill folder — always to the project-relative `./gtm-kits/...` path.
- Don't declare the launch live or schedule sends yourself — recommend GO/NO-GO and hand to the human.
- Don't over-build an S-tier launch with L-tier assets, or under-build the reverse — match the tier.

## Quality checklist (self-review before presenting)

- `brand-brain` called first; digest passed into every child skill (no stage re-asked for brand context)?
- Three inputs resolved (spec · segment · launch date); launch sized S/M/L by `gtm-launch-planner`?
- Stage-2 messaging matrix produced, and every asset visibly inherits it (same one-liner, value props, proof)?
- Every tier-required asset generated by its owning skill — none hand-written here?
- Each asset run through `content-qa-reviewer`; REVISE looped (≤2) or flagged `⚠ needs human edit`?
- All destination links UTM-tagged via `utm-parameter-bulk-builder`?
- `campaign-qa-launch-checklist-generator` run; a single GO/NO-GO verdict rolled up with blocking vs. caveat flags?
- Kit compiled to `./gtm-kits/[slug]-[date]/` with the README run summary; only real proof used, rest `[verify]`?
- Human handed the verdict, the next 3 dated owned actions, and everything needing their edit — and asked to approve?
