---
name: partnership-outreach-personalization-writer
description: >
  Turns a prospect list (LinkedIn profiles, company sites, funding data, press mentions, mutual
  connections) plus a base template into individually personalized outreach — biz-dev pitches,
  guest-post proposals, integration-partner overtures, and warm networking notes — plus a
  follow-up sequence for each thread. Powered by ROPE (Research → Offer → Proof →
  Exchange), our working checklist for these messages: every message opens on a researched, specific observation the recipient will
  immediately recognize as real, pivots to a clear, bounded offer that costs them nothing yet,
  pairs it with a proof point that reduces risk, and closes with one low-friction ask. Does NOT
  write cold-broadcast spam — every output is personalized to a specific company and person.
  Saves dossiers and finalized sequences to ./outreach/ so the full pipeline survives between
  sessions. Use when the user says "write partnership outreach," "personalize this cold email,"
  "pitch a guest post," "reach out to integration partners," "biz-dev email," "warm intro note,"
  "follow up on partnership," or hands over a prospect list and asks for outreach copy.
---

# Partnership Outreach & Personalization Writer

Every partnership email lives or dies on one thing: does the recipient believe you read their stuff, or did you mail-merge them? This skill writes outreach that passes the "did they actually look?" test — because it starts with real research, builds a genuinely bounded offer, backs it with real proof, and asks for one small thing.

Our working checklist: **ROPE** — Research → Offer → Proof → Exchange (a house mnemonic, not an established outreach model).

---

## Skills this calls

- **`brand-brain`** (required, first) — resolves the active brand's voice, banned words, ICP, real proof, offer mechanics, and positioning. Every pitch is written from these brand facts, never from made-up differentiators.
- **`account-dossier-builder`** — when a prospect URL is available but research is thin, delegates account profiling (business model, tech stack, recent news, pain hypotheses) rather than guessing.
- **`proof-vault`** — when specific proof points are needed (customer logos, metrics, case-study pull-quotes) to match a prospect's vertical; call instead of fabricating.
- **`objection-library-builder`** — surfaces the top objections a partner type raises so the pitch pre-empts rather than discovers them.
- **`cta-variant-generator`** — drafts and pressure-tests the single-ask close at the end of each message.
- **`de-slop-humanize-pass`** — final pass on any draft that reads like a template. Especially critical on follow-ups.

---

## How a run works

```
Step 0  Load brand context       ──► call brand-brain; block until digest returns
Step 1  Profile each prospect    ──► research gate; call account-dossier-builder or parse pasted data
Step 2  Score and tier           ──► Tier 1 (hand-craft each touch) / Tier 2 (template + real hook)
Step 3  Write with ROPE          ──► one message per prospect; follow-up scaffold
Step 4  Final humanize pass      ──► call de-slop-humanize-pass on every draft
Step 5  Save to ./outreach/      ──► one file per prospect; index file updated
```

---

## Step 0 — Load the brand (always first)

**Invoke `brand-brain`** (Skill tool, `skill: brand-brain`). It returns voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning, ICP. No outreach is written before this returns.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`. If none, ask the user to install `brand-brain` or answer a 5-field inline setup (brand name · ICP · value prop · 3 real proof points · voice adjectives + banned words). Prefer the call.

---

## Step 1 — Profile each prospect (research gate)

**Do not skip this for any Tier 1 prospect.** A pitch is only as personalized as its research.

For each prospect on the list, collect (from pasted data, LinkedIn URL, company URL, or by calling `account-dossier-builder`):

| Field | Why it matters in ROPE |
|---|---|
| Company's current growth motion or announced initiative | The R hook — what they're visibly working on |
| A specific piece of content they published (title + date) | Credibility in the opener — shows real attention |
| Recent funding, hire, product launch, or press mention | Timely, non-generic hook |
| Tech stack or integration ecosystem | Validates the integration offer |
| Relevant mutual connection or warm path | Reduces stranger-tax |
| The person's title and apparent ownership area | Targets the right pain, avoids wrong-door sends |

Mark any field you cannot confirm as `[verify]` — never invent a "recent blog post" or a funding round.

---

## Step 2 — Tier and scope

| Tier | Criteria | Treatment |
|---|---|---|
| **Tier 1** | Strategic fit; brand-relevant audience or tech overlap; ≥1 warm signal | Hand-craft R hook; every sentence earned |
| **Tier 2** | Good fit; less research data available | Semi-custom: shared template with a mandatory real hook replacing the placeholder |
| **Disqualify** | No clear value exchange; off-ICP audience; purely extractive ask | Flag and skip — don't send, explain why |

Ask the user to confirm the tier assignments before writing at volume.

---

## Step 3 — Write with ROPE

### R — Research hook (sentences 1–2)
Open on a **specific, verifiable observation** the recipient will immediately recognize as about them: a concrete piece of content, a stated initiative, a recent hire, a product decision. Not "I've been following your work" — that's a lie disguised as a compliment. One sentence max for the observation; one sentence connecting it to why you reached out.

> "Your post on using push notifications for cart recovery (May 14) called out the same retention problem we've been solving at [brand] — the 'notify-and-forget' trap."

### O — Offer (1–2 sentences)
State what you're proposing **as a bounded, low-cost-to-them thing**: a guest post, an integration co-announcement, a co-marketing asset, a 20-minute exploratory call, a shared audience introduction. Be specific about scope. Never ask for a revenue share, a co-investment, or a contract in the first message.

> "We'd love to co-write a guide on triggered-notification strategies for your blog — we'd draft it, you own it, and both teams get byline exposure to a shared audience."

### P — Proof (1 sentence)
One real, specific proof point that lowers the risk of responding. Pull from `brand-brain` or `proof-vault`. Mark anything unconfirmed `[verify]`.

> "We've published similar co-authored pieces with [Partner A] and [Partner B] that drove [X visits / Y signups] [verify] — they took us about two hours of their time."

### E — Exchange / ask (1 sentence)
One explicit, low-friction next step. Never two asks. Delegate to `cta-variant-generator` for the close; the default for partnership outreach is a 20-minute call or a "yes/no reply."

> "Would a 20-minute call next week make sense to see if there's a fit?"

---

## Step 3b — Follow-up scaffold

Every outreach gets a 3-touch scaffold saved alongside the initial message:

| Touch | Timing | Purpose |
|---|---|---|
| **F1** | +5 business days if no reply | Value-add bump — share a relevant resource, not "just following up" |
| **F2** | +7 business days after F1 | Reframe the offer; offer an alternative ask (intro vs. call) |
| **F3** | +10 business days after F2 | Clean close — "closing the loop; happy to revisit later" |

F1 must add something real (a stat, a resource, a relevant news hook). F2 may reduce the ask. F3 closes cleanly without guilt. No touch beyond F3 without a new signal.

---

## Step 4 — Humanize pass

Run every draft through `de-slop-humanize-pass` before finalizing. Specific slop patterns to flag for partnership outreach:

- "I wanted to reach out" / "hope this finds you well" / "I came across your work"
- Compliments that could apply to anyone ("your company is doing amazing things")
- Vague offers ("explore synergies," "mutually beneficial partnership")
- Passive asks ("feel free to reach out if interested")
- "Leverage" as a verb

---

## Step 5 — Save to ./outreach/

```
./outreach/
  index.md                      ← running list: prospect · tier · status · last touch
  [prospect-slug]/
    dossier.md                  ← research notes (populated by account-dossier-builder)
    outreach-sequence.md        ← initial pitch + F1/F2/F3 with send dates
```

Never save to the skill folder. Never write to `brand.md`.

---

## Outreach type quick-reference

| Type | ROPE emphasis | Key difference |
|---|---|---|
| **Biz-dev / distribution** | R: their audience signal; O: co-promotion scope | Proof = audience overlap or shared customer story |
| **Guest post / content swap** | R: a specific content gap you spotted; O: exact draft offer | Proof = sample post or published collab |
| **Integration / tech partner** | R: their tech stack or a user complaint you spotted; O: integration scope | Proof = existing integration or a demo |
| **Warm networking** | R: a mutual contact or shared context; O: an insight trade, not an ask | No proof required; exchange = intro or coffee |
| **Affiliate / referral** | R: their audience's pain match; O: commission structure upfront | Proof = existing affiliate earnings or conversion rate `[verify]` |

---

## Principles

- **Research before writing, always.** A fake hook is worse than no hook — it signals you automated the personalization.
- **One ask per message.** Two asks = zero replies.
- **Offer before you extract.** The first message should cost the recipient nothing except a reply.
- **Real proof or `[verify]`.** Never invent a case study, a customer name, or a metric to make the pitch stronger.
- **Tier honestly.** If you don't have enough research for a Tier 1, either do the research or send a Tier 2 — don't pretend.
- **Follow-ups add value.** "Just checking in" is the second-most-deleted phrase in a busy inbox.
- **Brand-brain first.** Voice, banned words, and offer mechanics come from the brand — not from the default pitch template.

---

## What not to do

- Don't open with a compliment that could apply to any company ("love what you're building").
- Don't pitch a revenue share, a contract, or a co-investment in a cold first touch.
- Don't use `[verify]` items in the Proof section without flagging them — silence on unconfirmed proof is dishonesty.
- Don't write follow-up F1 as "just following up" — if there's nothing to add, wait for a real signal.
- Don't call `account-dossier-builder` and then ignore what it returns — the dossier is the research gate.
- Don't treat Tier 2 as permission to skip the R hook — a real observation is mandatory even on a template.
- Don't save dossiers or sequences inside the skill folder.

---

## Quality checklist

- `brand-brain` called and active brand digest received before any copy was written?
- Every R hook references a specific, verifiable, date-stamped observation (no "I admire your work")?
- Offer is bounded (specific scope, low recipient cost) and written in the brand's voice?
- Proof is sourced from `brand-brain` or `proof-vault`; every unconfirmed number marked `[verify]`?
- Single ask at the close; `cta-variant-generator` called or inline close reviewed?
- `de-slop-humanize-pass` applied; slop patterns above removed?
- Follow-up scaffold (F1/F2/F3) written and each touch adds something?
- Tier assignments confirmed with the user before writing at volume?
- All outputs saved to `./outreach/[prospect-slug]/`; index.md updated?
- No pitch sent to a disqualified prospect; disqualification reason documented?
