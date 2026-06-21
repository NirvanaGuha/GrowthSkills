---
name: positioning-reviewer
description: >
  Audits a draft positioning statement (or full messaging platform) against competitive context and
  brand reality — producing a structured critique that scores clarity, differentiation, and
  believability, then prescribes exactly what to rewrite and why. Operates as a senior reviewer
  of the output from `positioning-messaging-architect`: it does NOT rebuild the positioning from
  scratch, it stress-tests what already exists. Uses the April Dunford "Obviously Awesome"
  framework — best-fit customer, category, unique value, alternative comparators, proof — as the
  scoring spine. Call this before any positioning statement graduates to homepage copy, an ad
  campaign, a board deck, or a sales one-pager. Trigger phrases: "review my positioning,"
  "critique my positioning statement," "is my positioning differentiated," "positioning gut-check,"
  "does my positioning hold up," "positioning vs competitors," "make my positioning stronger,"
  "pressure-test the messaging," "audit my value prop," or whenever a positioning doc arrives and
  needs a senior pass before it ships.
---

# Positioning Reviewer

A positioning statement that clears the internal review rarely clears the market. This skill runs the external stress-test — scoring what the draft says against what the customer hears, what competitors claim, and what the brand can actually prove. It does not rewrite positioning from scratch; it identifies exactly which part broke and how to fix it, so the person with context can make the right call.

The scoring spine is April Dunford's **Obviously Awesome** framework: positioning is only sound when it nails the best-fit customer, the competitive alternative, the unique capability, the value that capability delivers to that customer, and the market category that frames the comparison. A statement can sound confident and still fail every one of those five dimensions. This skill exposes the gap.

---

## Skills this calls

- **`brand-brain`** (required first) — resolves voice, ICP, offer mechanics, real proof, and the brand's existing positioning line. The reviewer evaluates the *draft* against what the brand can actually defend.
- **`positioning-messaging-architect`** (upstream producer) — if no draft exists, defer to it. This skill is the reviewer, not the author.
- **`competitive-intelligence-dossier`** (optional) — call it when competitive context is thin or stale. If already provided by the user, use that directly.
- **`proof-vault`** (optional) — call it to validate proof claims in the draft. If absent, mark unverified claims `[verify]`.
- **`icp-persona-builder`** (optional) — call it when the ICP is underspecified in `brand-brain`'s digest. Differentiation only holds relative to a defined buyer.

---

## How a run works

```
Step 0  Load brand context   ──► call brand-brain (voice, ICP, proof, existing positioning)
Step 1  Ingest the draft     ──► accept: statement, full platform, or a live URL/doc
Step 2  Load competitive alt ──► use user-supplied context or call competitive-intelligence-dossier
Step 3  Score on 5 dimensions ──► Dunford rubric, 1–5 per dimension with evidence
Step 4  Identify failure mode ──► one of seven named failure patterns (see below)
Step 5  Prescribe fixes      ──► rewrite directives per failing dimension, not generic advice
Step 6  Optional output      ──► save to ./positioning/[slug]-positioning-review.md if asked
```

Never skip Step 0. A positioning review without ICP and proof is editorial opinion — it has no leverage.

---

## Step 0 — Load brand context (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns the active brand's digest: voice adjectives, banned words, ICP + awareness tendency, offer mechanics, real proof points, and the existing positioning line if one exists.

Compare the draft positioning against the brand's real proof before scoring believability. If a claim in the draft can't be supported by the returned proof, mark it `[verify]` — do not silently accept it or silently delete it.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly. If none exists, ask the user for the five Dunford inputs (best-fit customer, named alternatives, unique capability, value delivered, category frame) and the top three real proof points before proceeding.

---

## The Obviously Awesome scoring rubric (Dunford, 5 dimensions)

Score each dimension 1–5. Show the score, the evidence (quote the draft), and the verdict in one tight table row. Then expand only the failing dimensions.

| # | Dimension | The question it answers | Fail signal |
|---|---|---|---|
| 1 | **Best-fit customer** | Who is this *exactly* for? Not a persona — a firmographic + behavioral wedge. | "companies," "marketers," "anyone who…" — too broad to be a differentiator |
| 2 | **Named alternative** | What would they do if you didn't exist? Not a competitor — a *behavior* (spreadsheet, do-nothing, incumbent). | No alternative named; positions against an imaginary competitor |
| 3 | **Unique capability** | What can you do that the named alternative structurally can't? (Feature ≠ capability; capability is a durable, hard-to-copy mechanism.) | Feature list instead of mechanism; "best-in-class" language |
| 4 | **Value delivered** | What does that capability unlock for the best-fit customer? Stated in their language, not the product's. | Benefits written in product language; no customer outcome |
| 5 | **Category frame** | What game are you playing, and does that frame favor you? Creating a new category, sub-categorizing, or repositioning an existing one? | Default category chosen by habit, not by what makes the unique capability obvious |

**Scoring scale:** 5 = sharp, defensible, distinct; 3 = present but diluted; 1 = missing or actively harmful.

A score of ≤3 on any dimension triggers a required Fix Directive (below). Do not let a mid-score dimension pass without at least a brief note.

---

## The seven failure patterns

After scoring, name the dominant failure pattern — the one framing move that would fix the most dimensions at once. Every weak positioning falls into one:

1. **Feature list masquerading as positioning** — lists what the product does, not what uniquely shifts the buyer's situation.
2. **Competing against the wrong alternative** — positioned against a named rival when the real alternative is a spreadsheet, a manual process, or the status quo.
3. **Claimed differentiation, no mechanism** — states "the only," "the best," "the fastest" without a durable structural reason it's true.
4. **ICP too broad to be useful** — the buyer is described at a level where positioning claims can't be made concrete; everyone is the customer, so no one is.
5. **Value stated in product language** — benefits written as features ("real-time sync," "AI-powered") instead of outcomes the buyer cares about ("closes the books in hours, not days").
6. **Category frame works against you** — entered the default category for the space even though the unique capability would be more obvious, and more defensible, in a different frame.
7. **Proof gap** — positioning makes claims the brand can't currently support; believability collapses under customer scrutiny.

---

## Fix Directives (for every dimension scoring ≤3)

For each failing dimension, write one Fix Directive — not a rewrite, but a precise instruction the human can execute:

```
DIMENSION: [name]
CURRENT: "[exact quote from draft]"
PROBLEM: [one sentence — what the score reflects]
DIRECTIVE: [what to change and the specific question to answer to fix it]
OPTIONAL EXAMPLE DIRECTION: [a rough shape, clearly marked as illustrative — never as final copy]
```

The Optional Example Direction is always labeled "illustrative only" and always ends with `[verify proof before using]` if it includes a number or claim.

Fix Directives are not rewrites. The reviewer's job is to raise the right question with enough precision that the positioner can answer it. If you are also the positioner (the user runs both skills), the directives are the brief for your next `positioning-messaging-architect` run.

---

## Competitive stress-test (optional but strongly recommended)

If competitive context is available (user-supplied or via `competitive-intelligence-dossier`):

1. Pull the top 3 competitors' headline positioning claims.
2. Run the **swap test**: replace the brand name in the draft with each competitor's name. If the statement still reads true, it has not differentiated — flag it.
3. Run the **walk-away test**: would the best-fit customer walk away from a competitor and choose this brand *because of what the positioning says*? If not, the capability or value is underspecified.

Output the swap-test results inline alongside the score table. One line per competitor, verdict: PASSES (unique) or FAILS (interchangeable).

---

## Output format

```
## Positioning Review — [Brand slug], [Date]

**Draft reviewed:** "[full statement, quoted]"
**Reviewed against:** [competitive alternatives used, sources]
**Brand brain loaded:** [slug, brand.md path, confidence]

### Dunford Score
| Dimension | Score | Evidence (from draft) | Verdict |
|---|---|---|---|
| Best-fit customer | /5 | "…" | … |
| Named alternative | /5 | "…" | … |
| Unique capability | /5 | "…" | … |
| Value delivered | /5 | "…" | … |
| Category frame | /5 | "…" | … |
| **Total** | **/25** | | |

**Dominant failure pattern:** [one of the seven named patterns]

### Competitive Swap Test
| Competitor | Swap result |
|---|---|

### Fix Directives
[one per dimension scoring ≤3]

### Proof gap log
[any claim in the draft not supported by brand-brain's proof digest — marked [verify]]

### Summary verdict
[2–3 sentences: what's working, what breaks first, the one move that fixes the most]
```

Save to `./positioning/[slug]-positioning-review.md` when the user asks or the brand has a positioning folder. Inline otherwise.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No review begins without knowing the brand's real proof and ICP. Scoring believability on imagination is editorial theater.
- **Dunford rubric, every dimension.** Do not skip a dimension because the draft is silent on it — silence is a score of 1. Name it.
- **One failure pattern per review.** Resist listing every problem equally; the dominant pattern is the lever. Fix that and three other scores often rise.
- **Directives, not rewrites.** The reviewer prescribes; the positioner authors. Maintain the lane.
- **Honest proof discipline.** Claims the brand cannot currently support are marked `[verify]`, never smoothed over.
- **The swap test is mandatory when competitive data exists.** Interchangeable positioning is not positioning.

## What Not to Do

- Don't rewrite the positioning statement — issue Fix Directives and let `positioning-messaging-architect` do the authoring pass.
- Don't score without quoting the draft — every verdict needs evidence from the text.
- Don't invent competitor claims — use only what's returned by `competitive-intelligence-dossier` or supplied by the user.
- Don't accept "the only" or "the best" without a proof citation — route to `[verify]`.
- Don't skip the best-fit customer dimension because it "seems fine" — ICP drift is the most common invisible failure.
- Don't produce a review without `brand-brain` context; a generic positioning checklist is not this skill.

## Quality Checklist (self-review before presenting)

- `brand-brain` loaded; ICP, real proof, and existing positioning line in hand?
- All five Dunford dimensions scored with quoted evidence?
- Dominant failure pattern named (one, not five)?
- Swap test run for every competitor with competitive data present?
- Fix Directive written for every dimension scoring ≤3 — directive form (not a rewrite)?
- Proof gap log complete; all unverified claims marked `[verify]`?
- Summary verdict states what's working *and* what breaks first?
- Output saved to `./positioning/[slug]-positioning-review.md` if a save was requested or the folder exists?
