---
name: battlecard-objection-handler
description: >
  Competitive dossier + messaging hierarchy + known objections → sales-ready one-page battlecard
  with reframe responses and proof points. Composes competitive-intelligence-dossier and
  objection-library-builder rather than re-deriving competitor or objection data; its job is
  the battlecard artifact — the opinionated, rep-ready synthesis that turns raw intel into a
  winnable conversation. Obeys brand voice from brand-brain (tone, banned words, positioning
  line). Outputs a structured one-pager plus optional deep-dive per named competitor.
  Use when someone says "build a battlecard," "competitive one-pager," "how do I sell against X,"
  "handle this objection," "why we win vs. competitor," "objection reframe," "sales cheat sheet,"
  or pastes a competitor comparison and asks what to say.
---

# Battlecard & Objection Handler

Turn competitive intelligence and known objections into a single page a rep can open thirty seconds before a call. This skill does not research competitors or compile objections from scratch — it composes `competitive-intelligence-dossier` and `objection-library-builder` for those layers, then synthesizes their output into the *artifact*: a tightly structured, on-voice, proof-backed battlecard that tells a rep exactly what to say, when to say it, and what traps to avoid.

The output lives at `./sales/battlecard-[competitor-slug].md` (or `./sales/battlecard-master.md` for a multi-competitor card). Every reframe is tied to a real proof point or marked `[verify]`.

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads the active brand's voice, ICP, positioning, banned words, and proof. No battlecard copy is written before this returns.
- **`competitive-intelligence-dossier`** (required unless caller passes raw dossier) — fetches/structures competitor positioning, pricing tier, differentiators, and known weaknesses. Pass the active brand slug + competitor name(s); receive structured intel.
- **`objection-library-builder`** (required unless caller passes pre-built objection list) — surfaces the top recurring objections (budget, timing, competitor preference, feature gap, trust) for this ICP segment. Receive grouped objections with frequency signal.
- *(optional)* `proof-vault` — for additional proof points when the brand.md proof section is thin. Synthesize inline when absent.
- *(optional)* `positioning-messaging-architect` — if the brand's positioning is unclear or the brand is new; otherwise read it from brand.md.

---

## How a run works

```
Step 0  Load brand        ──► brand-brain (voice, ICP, positioning, proof, banned words)
Step 1  Load intel        ──► competitive-intelligence-dossier per competitor
Step 2  Load objections   ──► objection-library-builder for this ICP
Step 3  Apply the framework ► Build the battlecard (see below)
Step 4  Self-review       ──► check against quality checklist, then output
```

### Step 0 — Brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Use the returned digest: voice adjectives, banned words, offer mechanics, real proof, ICP awareness tendency, positioning line. Hard overrides: no copy violates voice or banned-words; no proof invented.

**Fallback if brand-brain is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`. If neither exists, ask the user for: brand name, ICP, positioning in one sentence, top 3 proof points, banned words. Proceed once confirmed.

### Step 1 — Competitive intel

If the user passes a raw dossier or pasted competitor page, extract intel directly. Otherwise invoke `competitive-intelligence-dossier` (Skill tool) with the competitor name(s). Parse the returned sections for: positioning claim, pricing anchor, top 3 stated strengths, documented weaknesses/gaps, and any known trap phrases reps hear on calls.

### Step 2 — Objection data

If the user provides an objection list, use it. Otherwise invoke `objection-library-builder` (Skill tool) scoped to this ICP segment. Receive top objections by category (price, competitor, timing, trust, feature). Take the top 5–7 into the card; move the rest to the appendix.

### Step 3 — Build the battlecard (Challenger-reframe model)

The structural framework is the **Challenger reframe**: don't defend, reframe the decision criteria. Every response section follows three beats — **Acknowledge → Reframe → Prove** — so the rep never sounds defensive and always moves the frame.

---

## The Battlecard artifact

Output as a single Markdown file written to `./sales/battlecard-[competitor-slug].md`. When no competitor is named, write `./sales/battlecard-master.md` covering top 2–3 competitors.

### Required sections (in order)

```markdown
# Battlecard: [Brand] vs. [Competitor]
> One-line positioning against this competitor. Brand voice. No banned words.

## Quick-reference (30-second scan)
| We win when | We lose when | Never say |
|---|---|---|
| [ICP fit signals] | [honest disqualifiers] | [trap phrases] |

## Their narrative — and how to reframe it
[One sentence: what their rep says about you. Then your reframe.]
Framework: Acknowledge → Reframe → Prove

## Head-to-head: decision criteria
| Criterion | [Competitor] | [Brand] | Talking point |
|---|---|---|---|
(4–6 rows. Honest — include one where they're stronger, with a reframe)

## Objection responses
### "[Objection verbatim or close paraphrase]"
**Reframe:** [one sentence, Acknowledge → Reframe]
**Proof:** [real metric, case quote, or `[verify]`]
**Bridge:** [one sentence moving to next step]
(Repeat for each objection, top 5–7)

## Proof points (quick-paste)
- [Proof 1] — source or `[verify]`
- [Proof 2] — source or `[verify]`
(3–5 lines max; only real proof)

## Trap phrases to avoid
[Words/claims that backfire with this ICP — brief, bullets]

## When to walk away
[One honest line about deal profiles where the competitor genuinely wins — credibility builder]
```

### Depth toggle

- **Quick (default):** one-page output as above, inline in chat, then saved to `./sales/`.
- **Deep (on request or when 2+ competitors):** one file per competitor with an additional "Competitive threat level" section (High/Medium/Low with rationale) and a "Land mines to defuse" section covering FUD the competitor plants.

---

## Framework: Acknowledge → Reframe → Prove (A-R-P)

The method behind every objection response. Do not skip Acknowledge — a rep who skips it sounds defensive and signals fear.

| Beat | Job | Example |
|---|---|---|
| **Acknowledge** | Show you heard it; reduce tension | "That's a fair question — [competitor] does lead on [feature]." |
| **Reframe** | Shift the decision criterion to where you win | "The question worth asking is [new frame]: …" |
| **Prove** | Close with a real fact, metric, or quote | "[Customer] saw [result] in [timeframe]. [verify if unconfirmed.]" |

**Key applications:**

- *Price objection:* Acknowledge cost; reframe to TCO, onboarding cost, or support overhead. Prove with retention/ROI data.
- *Feature gap:* Acknowledge the gap honestly; reframe to outcomes (what the feature is *for*) and show your alternative path. Never claim parity you don't have.
- *Competitor preference:* Acknowledge familiarity; reframe to current-need fit vs. legacy familiarity. Prove with a segment-specific win story.
- *Trust/size:* Acknowledge scale; reframe to responsiveness, roadmap influence, and support SLA. Prove with support metrics or named customer access.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No battlecard copy is written before `brand-brain` returns. Its voice, positioning, and banned-words override everything else.
- **Compose, don't re-derive.** Competitive intel comes from `competitive-intelligence-dossier`; objections from `objection-library-builder`. Do not re-research or re-compile from scratch here.
- **Acknowledge before you reframe.** Skipping Acknowledge makes the rep sound defensive. It's not optional.
- **Honest about where you lose.** Include one real disqualifier in the "We lose when" row and one "When to walk away" line. Credibility is the battlecard's most valuable property.
- **Real proof or `[verify]`.** Never invent metrics, case quotes, or differentiators. Every proof point has a source or is marked `[verify]`.
- **One page in practice.** The artifact must be scannable in 30 seconds. If it bloats, cut — move extras to an appendix.

---

## What Not to Do

- Don't write a battlecard before `brand-brain` loads. Voice and positioning errors in a battlecard reach prospects.
- Don't re-implement competitor research or objection compilation — call `competitive-intelligence-dossier` and `objection-library-builder`.
- Don't write "we're better than X at everything" — it destroys credibility with a rep who's already heard the competitor's pitch.
- Don't invent proof points. A `[verify]` marker is far better than a fabricated stat a prospect Googles and disproves on the call.
- Don't use banned words or superlatives the brand disavows.
- Don't produce a 5-page document and call it a battlecard. If deep, use the depth toggle — one page + an optional appendix.

---

## Quality Checklist (self-review before output)

- `brand-brain` called and digest loaded before any copy was written?
- `competitive-intelligence-dossier` called (or raw dossier provided) — no competitor intel invented?
- `objection-library-builder` called (or pre-built list provided) — top 5–7 objections included?
- Every objection response follows Acknowledge → Reframe → Prove?
- Head-to-head table includes at least one honest competitor advantage with a reframe?
- "When to walk away" row present — credibility signal included?
- All proof points real or marked `[verify]`; no banned words; brand positioning line used?
- Output fits one page at scan speed; saved to `./sales/battlecard-[slug].md`?
