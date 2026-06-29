---
name: sales-partner-collateral-creator
description: >
  Turns a solution brief, target persona, proof points, and an account dossier into a
  full suite of sales-ready collateral — all in one pass. Outputs a print-ready one-pager
  (problem/solution/proof/CTA), a set of personalized deck slide swaps keyed to each
  account's stated priorities, a tight Loom pitch script with a hook-body-CTA structure,
  and partner co-sell talk tracks the channel team can deliver without a prep call. Every
  piece is written in the active brand's voice (via brand-brain) and anchored to real
  proof only — no invented differentiators, no generic "we help you succeed" filler.
  Composes account-dossier-builder for deep account intelligence, proof-vault for
  substantiated claims, battlecard-objection-handler for objection coverage, and
  advertising-claims-ftc-disclosure-reviewer for any regulated claims. Saves all outputs
  to ./collateral/<account-or-partner-slug>/. Use when the user says "build a one-pager,"
  "create sales collateral," "personalize my deck," "write a Loom pitch script," "build
  partner talk tracks," "co-sell enablement," "deck slide swaps," or hands you an account
  and a solution and asks for sales-ready materials.
---

# Sales & Partner Collateral Creator

Give it a solution, a persona, real proof, and an account or partner name — get back a full collateral kit a rep can use today: a one-pager, personalized deck swaps, a Loom script, and partner co-sell talk tracks. No generic assets. Every piece is anchored to the account's actual priorities and the brand's real proof.

This skill produces sales assets. It does not generate leads, manage CRM records, or run outreach sequences — hand it materials you're ready to customize, and it returns copy that closes. If the proof is thin or the persona is vague, it says so rather than inventing substance.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads active brand voice, offer mechanics, positioning, ICP, and real proof. Does not reimplement brand resolution itself. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand voice adjectives, banned words, the primary offer/CTA, and three real proof points before proceeding.
- **`account-dossier-builder`** — deep-researches the account (industry, pain signals, tech stack, key stakeholders, recent news) to personalize slide swaps and talk tracks. Run when an account name is provided and no dossier already exists in the session.
- **`proof-vault`** — surfaces and substantiates claims. Marks any claim the vault cannot confirm as `[verify]`. Never invent numbers.
- **`battlecard-objection-handler`** — pulls objection map for the persona/solution pairing; inlines the top 3 objections into the one-pager and talk tracks. Synthesize inline if not installed.
- **`advertising-claims-ftc-disclosure-reviewer`** — gates any regulated claims (#1, "best," guarantee language, testimonial attributions) before finalizing. Flag and hold — don't publish unreviewed superlatives.
- **`loom-async-video-script-writer`** — drafts the Loom pitch script structure (hook, body, CTA, on-screen cue notes); this skill adapts it to the account and brand voice.

---

## How a run works

```
Step 0  Load brand context  ──► brand-brain (always first; fallback above)
Step 1  Build the intelligence layer  ──► account-dossier-builder + proof-vault + battlecard-objection-handler
Step 2  Choose the collateral scope  ──► Full Kit (default) | Single Asset (on request)
Step 3  Produce assets (framework: our 5-step arc adapted from Challenger Sale)
Step 4  Gate regulated claims  ──► advertising-claims-ftc-disclosure-reviewer
Step 5  Self-review, then save + present
```

---

## Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns the active brand's voice adjectives, banned words, offer mechanics, real proof points, positioning line, ICP + awareness tendency, and the path to `brand.md`. Do not write a single line of collateral until it returns.

Obey voice and banned-words as hard overrides. Use only real proof from the returned digest; mark anything else `[verify]`. If the session already has a fresh brand digest from this run, re-use it without re-calling.

**Fallback if brand-brain is absent or returns no brand:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand voice adjectives, banned words, the primary offer/CTA, and three real proof points before proceeding.

---

## Step 1 — Intelligence layer

**Account dossier.** Call `account-dossier-builder` when an account (company name) is provided. Extract from the returned dossier: the account's confirmed pain in the persona's lane, their current solution/vendor (displacement angle vs. greenfield), any recent trigger events (funding, reorg, public pain), and the buying-committee stakeholders who matter for this asset.

If the user pastes a dossier or context block directly, parse it rather than rebuilding.

**Proof.** Call `proof-vault`. Extract the strongest 2–3 proof points mapped to the persona's priority metric (time-to-value, cost reduction, revenue lift, compliance, retention). Mark unconfirmed numbers `[verify]`.

**Objections.** Call `battlecard-objection-handler` for the solution × persona pairing. Pull the top 3 objections most likely to surface in this account context. Inline the handling language in the one-pager and talk tracks.

---

## Step 2 — Collateral scope

| Scope | Trigger |
|---|---|
| **Full Kit** (default) | "collateral," "kit," "everything," or account + solution provided without a specific format |
| **Single asset** | User names a specific format: "just the one-pager," "only the Loom script," "partner talk tracks" |

Full Kit produces all four assets in a single pass. Single-asset mode runs only the requested piece but still calls `brand-brain` and does the proof gate.

---

## The message structure (governs all four assets)

Every asset follows this arc — our working adaptation of the Challenger Sale's Commercial Teaching choreography (Dixon & Adamson), condensed into five steps for collateral. Don't flatten it into a feature list.

1. **Reframe (Teach)** — Open with an insight or tension the buyer hasn't articulated yet. Not a product pitch; a perspective shift. Draws from the account dossier's pain signals + the brand's positioning.
2. **Tailor** — Connect the reframe explicitly to this persona's role, metric, and business context. Uses dossier-derived specifics.
3. **Displace / Contrast** — Show why the current state (their existing approach or vendor) leaves the reframe unresolved. Honest, not aggressive.
4. **Solution + Proof** — Introduce the brand's solution as the resolution. Anchor to 2–3 real proof points (proof-vault). No invented claims.
5. **Nail the CTA** — One clear, low-friction next step sized to the awareness level of this asset type.

---

## Asset 1 — The One-Pager

A single-page sell sheet. Dense but readable; built for a 90-second read.

**Structure:**
- **Header:** Company logo placeholder + brand-aligned tagline for this solution.
- **The Problem (Reframe):** 2–3 sentences — the insight or tension. No bullet soup.
- **The Solution:** 2–3 short bullets — what it does (not how it works). Each bullet maps directly to a stated persona pain.
- **Why Now / Why Us:** 1–2 lines of displacement rationale + the single strongest differentiator.
- **Proof:** 2–3 substantiated proof points (metric + context + source or `[verify]`). No orphan statistics.
- **Objection absorber:** 1 short block handling the single most common objection for this persona. Drawn from `battlecard-objection-handler`.
- **CTA:** One action. Include destination URL if known from brand.md; mark as `[verify URL]` if not.
- **Footer:** Brand contact + legal/compliance line if regulated claims present.

Tone: senior-operator peer-to-peer. No exclamation marks unless brand allows. No "we're excited to." Length: fits one page (approx. 300–400 words body).

Save to `./collateral/<slug>/one-pager.md`.

---

## Asset 2 — Personalized Deck Slide Swaps

Not a full deck. A set of targeted slide replacements — the slides where personalization actually moves the needle. Reps drop these into their standard deck.

**Deliver:**
- **Title slide swap** — company name + the reframe headline, account-specific.
- **"Your world" slide** — 3–4 bullets mapping the account's confirmed pain signals (from dossier) to the challenge the solution resolves. Source each signal to a dossier finding.
- **Proof slide swap** — 2–3 proof points most relevant to this persona's priority metric. Formatted as: "[Metric] → [Result] ([Customer type / source or `[verify]`])".
- **"Why now" slide** — trigger event or market timing insight from the dossier or brand positioning.
- **Recommended next-step slide** — single CTA + logistics (demo booking link, pilot offer, etc. from brand.md).

For each slide: write the headline copy, the 3–4 body bullets or the stat block, and one presenter note (what to say aloud that isn't on the slide — the insight or anecdote that sharpens it).

Tone: boardroom-confident; every claim needs a source or `[verify]`. No aspirational hand-waving.

Save to `./collateral/<slug>/deck-slide-swaps.md`.

---

## Asset 3 — Loom Pitch Script

A 3–5 minute async video script. Built for a rep who knows the product but needs a structured, account-specific narrative they can record in one take.

**Structure (Challenger arc in video form):**

```
[Hook — 20–30s]
Address the viewer by name/role. Open with the reframe insight — the tension
they feel but haven't named. Do NOT start with "Hi, I'm [name] from [company]."
Cut to the value in sentence one.

[Body — 2:30–3:30]
Segment 1 (Tailor): connect the insight to their specific world — use 1–2
dossier-sourced specifics. "I noticed [trigger / pain signal]."
Segment 2 (Displace): describe the cost of the current state — what they're
leaving on the table. One stat or example (proof-vault).
Segment 3 (Solution + Proof): introduce the solution. Show — don't tell.
Walk one proof point end-to-end. Keep product features subordinate to outcomes.

[CTA — 30–45s]
One ask. Low friction for the awareness level (discovery call, quick demo,
a specific question to reply with). Give them the exact next step.
"If [X] resonates, reply with Y — or book 20 minutes at [link from brand.md]."
```

**Delivery notes** (for the rep):
- Screen-share the account's own website or report during "Tailor" segment to anchor the reframe in their reality.
- Pause 1–2 seconds before the CTA. Don't rush it.
- Keep total runtime under 5 minutes; tighter is better.

Mark any claim requiring verification as `[verify before recording]`.

Save to `./collateral/<slug>/loom-pitch-script.md`.

---

## Asset 4 — Partner Co-Sell Talk Tracks

Talk tracks a channel partner can pick up and run without a prep call. The partner is NOT the brand — they're a warm third-party voice. Tone shifts accordingly: peer recommendation, not vendor pitch.

**Deliver two tracks:**

**Track A — Cold/warm intro (partner to prospect):** Partner has a relationship; prospect doesn't know the brand yet. Goal: create curiosity and a warm introduction.
- Opening line that references the partner's context ("We've been solving [X] for accounts like yours...").
- 2–3 sentences of the reframe insight — positioned as a joint perspective, not a product pitch.
- Hand-off line that passes cleanly to the brand seller. ("I've cc'd [Name] who can show you exactly what this looks like in practice — worth a 20-minute call?")

**Track B — Joint-call support (partner + brand AE on the call):** Partner is on a call that surfaces the problem the brand solves. Goal: introduce the brand's solution as a natural extension of the partner's recommendation.
- "Bridging" language the partner uses to introduce the brand without breaking rapport.
- 2–3 talking points that reinforce the brand's positioning in the partner's voice — not marketing copy, real-language talking points.
- Objection handling for the 1–2 most common co-sell objections (drawn from `battlecard-objection-handler`).

Include a one-paragraph "Partner context brief" at the top of each track: what the partner needs to know about the brand's ICP, positioning, and offer to deliver this credibly.

Save to `./collateral/<slug>/partner-talk-tracks.md`.

---

## Principles (Non-Negotiable)

- **Brand-brain first, always.** No collateral before the brand context loads.
- **Challenger arc, every time.** Reframe → Tailor → Displace → Prove → CTA. Don't flatten to a feature list.
- **Real proof or `[verify]`.** No invented metrics, no orphan statistics. If proof-vault can't confirm it, flag it.
- **Gate regulated claims.** Any "#1," "best," "guarantee," or testimonial passes through `advertising-claims-ftc-disclosure-reviewer` before finalizing.
- **Account specificity is the job.** Generic collateral is worthless. If you don't have dossier data, say so and prompt the user before fabricating account details.
- **One CTA per asset.** Each piece ends with exactly one next action — no menu of options.
- **Partner voice ≠ brand voice.** Track A and B are written as a trusted peer, not a vendor. Keep them warm and non-promotional.

---

## What Not to Do

- Don't write any collateral before `brand-brain` returns — not even a headline.
- Don't invent company-specific pain signals, trigger events, or stakeholder names. If the dossier is empty, say so.
- Don't produce superlatives (#1, best, fastest) without the `advertising-claims-ftc-disclosure-reviewer` gate.
- Don't stack competing CTAs. One ask per asset, sized to the asset's awareness level.
- Don't write partner talk tracks in brand-marketing copy register — they'll read as planted and destroy rep credibility.
- Don't rebuild brand resolution, proof verification, or objection mapping inline — call the relevant sibling skills.

---

## Quality Checklist (self-review before saving)

- `brand-brain` called and active brand loaded (or fallback executed) before any copy was written?
- Account dossier sourced — no fabricated company details; dossier-specific signals appear in swaps and talk tracks?
- Proof-vault confirmed all claims; unconfirmed stats marked `[verify]`?
- Regulated claims reviewed by `advertising-claims-ftc-disclosure-reviewer`?
- Challenger arc present in each asset: reframe opens, proof anchors the middle, single CTA closes?
- One-pager: fits one page, objection absorber present, real destination URL or `[verify URL]`?
- Deck swaps: each slide has a presenter note; "your world" slide cites dossier source?
- Loom script: under 5 minutes at natural pace; hook opens in ≤30s without a "Hi I'm from..."; delivery notes included?
- Partner tracks: two distinct tracks (cold intro + joint call), partner context brief at top of each, partner voice not brand voice?
- All four assets saved to `./collateral/<account-or-partner-slug>/`?
