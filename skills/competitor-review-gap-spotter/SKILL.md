---
name: competitor-review-gap-spotter
description: >
  Turns public review data into a strategic ownership map. Give it your review themes + a
  competitor's review themes (G2, Capterra, Trustpilot, App Store, Reddit, Gartner Peer Insights,
  or any structured copy-paste) and it surfaces the pains your competitor's customers voice loudly
  but your competitor ignores in their messaging, their product positioning, and their response
  behavior — gaps you can credibly own. Outputs a ranked gap matrix, a per-gap messaging brief,
  and a content/feature priority signal. Calls brand-brain to anchor every gap against your
  actual proof and positioning; calls competitive-intelligence-dossier when a richer competitor
  dossier exists; calls voice-of-customer-mining-pipeline when raw verbatim mining is still needed.
  Use whenever the user says "review gap analysis," "what pains does my competitor miss,"
  "find gaps in competitor reviews," "mine G2/Capterra for angles," "competitor review intelligence,"
  "where does [competitor] fail their customers," or pastes a block of review text asking for
  competitive insight.
---

# Competitor Review Gap Spotter

Your competitor's 1-star reviews are their roadmap leak. Their 5-star reviews tell you which pain points they've *decided not to talk about*. This skill cross-references both sides — what their reviewers say versus what the competitor actually claims — to surface the pains that are loud, real, and unclaimed. Those are gaps you can own.

Our working model here is VOCA-Gap (a house framework, not an external standard): **V**oiced Pain Frequency × **O**wner Absence × **C**redential Match × **A**ction Readiness. A gap scores highest when: reviewers raise it repeatedly, the competitor's messaging ignores it, your brand has real proof against it, and you can move on it within a quarter.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads active brand context: positioning, real proof, banned words, ICP. Every gap is filtered through your brand's real capability; you cannot claim gaps you cannot prove.
- **`competitive-intelligence-dossier`** (call when installed) — for a richer competitor profile (positioning claims, website copy, ad angles) to sharpen the Owner Absence score. Synthesize inline from what the user provides if absent.
- **`voice-of-customer-mining-pipeline`** (call when installed) — when raw verbatim reviews need mining/tagging before comparison. If absent and the user provides raw text, do the thematic bucketing inline.

---

## How a run works

```
Step 0   Load your brand        ──► call brand-brain
Step 1   Ingest review sets      ──► your reviews + competitor's reviews (or raw paste)
Step 2   Tag both sets           ──► bucket into pain themes, note valence + frequency
Step 3   Map Owner Absence       ──► which competitor-voiced pains does their messaging not address?
Step 4   Score gaps (VOCA-Gap)   ──► rank by four-factor score
Step 5   Credential filter       ──► drop or flag gaps your brand cannot prove
Step 6   Build the gap matrix    ──► output ranked table + per-gap messaging brief
Step 7   Self-review + save
```

---

## Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`) before reading any reviews. It returns: voice adjectives, banned words, positioning line, real proof points, ICP + awareness tendency, and companion files (`competitors.md`, `proof.md` if present).

If `brand-brain` is not installed: read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask for a 4-question mini-setup (what it is · ICP · 3 real proof points · positioning line) and proceed. Flag that the brand context is unverified.

---

## Step 1 — Ingest review sets

Accept any of:
- **Pasted text** — raw review blocks; extract quotes, ratings, dates if present.
- **CSV / structured export** — G2, Capterra, Trustpilot, App Store, Gartner Peer Insights exports. Key columns: rating, review text, reviewer role/company size if available.
- **URL** — if a scrape/fetch tool is available, pull the page; otherwise ask the user to paste.
- **Asymmetric input** — user provides only competitor reviews. In this case, synthesize your brand's review themes from the brand-brain `proof.md` or proof section (use real proof, not invented praise), or ask for a sample.

Minimum viable input: competitor review text. Everything else is additive precision.

---

## Step 2 — Tag both review sets (Thematic Bucketing)

For each review set, extract recurring pain/praise themes using a consistent taxonomy. Apply these standard theme buckets (add brand-specific ones as needed):

| Bucket | What it captures |
|---|---|
| **Onboarding / Setup** | Time-to-value, complexity, documentation |
| **Support quality** | Response time, resolution, human access |
| **Reliability / Uptime** | Bugs, downtime, data loss |
| **UI / Ease of use** | Learning curve, navigation, visual clutter |
| **Pricing / Value** | Cost vs. output, hidden fees, tier limits |
| **Integrations** | Missing connectors, broken syncs, API limits |
| **Reporting / Analytics** | Depth, export formats, real-time data |
| **Feature completeness** | Missing capabilities vs. job-to-be-done |
| **Scalability** | Performance at high volume, enterprise gaps |
| **Account management** | Responsiveness, proactive guidance |

For each theme, note: **frequency** (count or %, low/med/high), **valence** (pain or praise), and **representative verbatim** (1–2 real quotes, attributed to platform if known).

---

## Step 3 — Map Owner Absence (the core intelligence step)

Owner Absence means: *this pain is voiced loudly by the competitor's customers, but the competitor's messaging does not address it credibly.*

Check three surfaces for each competitor-voiced pain:
1. **Competitor's own review responses** — do they acknowledge and fix, or deflect and ignore?
2. **Competitor's website/positioning copy** (use `competitive-intelligence-dossier` if available, or what the user provides) — does the pain appear as a feature claim?
3. **Competitor's self-described strengths in 5-star reviews** — does their best-case customer still not mention this pain as solved?

A theme scores **High Owner Absence** when all three surfaces are silent or evasive. **Medium** when mentioned but unconvincingly. **Low** (deprioritize) when the competitor clearly addresses it and reviewers confirm it.

---

## Step 4 — VOCA-Gap Scoring

Score each competitor-voiced pain gap on four factors (1–3 each):

| Factor | What it measures | Scoring guide |
|---|---|---|
| **V — Voiced Frequency** | How often does this pain appear in competitor reviews? | 1 = rare mentions; 2 = recurring theme; 3 = dominant/top complaint |
| **O — Owner Absence** | How absent is the competitor in addressing it? | 1 = they address it credibly; 2 = they mention it but weakly; 3 = silent or evasive |
| **C — Credential Match** | Does your brand have real, provable capability here? | 1 = no proof; 2 = partial/in-progress; 3 = strong documented proof |
| **A — Action Readiness** | Can you exploit this gap within one quarter (content, feature, positioning)? | 1 = needs major investment; 2 = medium lift; 3 = can move now |

**VOCA Score = V + O + C + A (max 12).** Rank descending. Gaps scoring C=1 (no credential) are flagged "CLAIM RISK" and excluded from the messaging brief unless the brand plans to build the proof.

---

## Step 5 — Credential Filter (Truth Discipline)

For each top-scoring gap, check it against the brand-brain proof:
- **Proven gap:** brand has documented proof (case study, metric, feature page, verified testimonial). Mark as `[proven]`.
- **Partial gap:** brand has capability but no published proof. Mark as `[verify]` — do not claim until proof is documented.
- **Aspirational gap:** brand does not yet have this capability. Mark as `[build first]` — do not claim.

Never generate messaging copy for `[build first]` gaps. Flag them as product roadmap signals instead.

---

## Step 6 — Gap Matrix + Messaging Brief

### Output format

```
## Competitor Review Gap Analysis — [Competitor] vs. [Your Brand]
Brand: [slug, via brand-brain] | Date: [today] | Sources: [review platforms + counts]

### Top Gaps (VOCA score ≥ 8)
| Rank | Theme | VOCA Score | Competitor Pain Evidence | Owner Absence | Your Credential | Status |
|---|---|---|---|---|---|---|
| 1 | [theme] | [score] | "[verbatim quote]" (n=[count]) | [High/Med/Low] | [proof ref] | [proven/verify/build first] |
...

### Mid-Tier Gaps (VOCA score 5–7)
[same table, collapsed]

### Excluded Gaps (C=1, no credential)
[list only — do not generate copy for these]

---

### Per-Gap Messaging Brief (Top Gaps, proven/[verify] only)

#### Gap 1: [Theme]
**The pain in their words:** "[best verbatim quote]"
**Why they ignore it:** [one line — product gap, strategic choice, blind spot]
**Your angle:** [one sentence connecting your proof to this pain — on-brand voice, no banned words]
**Proof to cite:** [exact proof point from brand.md — or [verify] if unconfirmed]
**Content / channel signal:** [where this angle fits: hero copy, comparison page, G2 review ask, ad, sales one-pager, blog post]
**Urgency:** [now / this quarter / next quarter]

---

### Product / Roadmap Signals ([build first] gaps)
[List gaps you cannot yet claim — this is intelligence for the product team, not copy]
```

Save to `./competitor-gaps/[competitor-slug]-gaps.md` when the user has multiple competitors or asks to save. Inline output by default.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No gap analysis before the brand context loads. Every gap is filtered through real proof.
- **Frequency beats emotion.** One furious 1-star review is noise. Ten mentions of the same theme across review periods is signal.
- **Owner Absence is the unlock.** A loud pain your competitor addresses well is a benchmark, not a gap. A loud pain they ignore is your opening.
- **No invented proof.** If you cannot cite a real proof point from brand.md, the gap is `[verify]` or `[build first]`, never a messaging claim.
- **VOCA-Gap is a ranking tool, not a checklist.** Use it to prioritize — not to manufacture importance for weak gaps.
- **Recency matters.** Older reviews that recent reviews contradict should be noted as potentially resolved; do not claim a gap that may be closed.

---

## What Not to Do

- Do not generate claims for `[build first]` gaps — pass them to the product team, not to copy.
- Do not report every review theme as a gap. Themes the competitor addresses well are competitive baselines, not opportunities.
- Do not invent verbatim quotes — cite real review text or mark the source as `[paraphrased from n mentions]`.
- Do not conflate a single platform's reviews with the full picture — note platform coverage and flag if only one source is available.
- Do not run competitor gap analysis without loading the brand first; voice + proof mismatches in copy are worse than silence.
- Do not claim a pain is "ignored by the competitor" without checking their review responses and website copy — both count.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded (or bootstrapped) before any analysis?
- Both review sets tagged using consistent theme buckets, with verbatim evidence and frequency noted?
- Owner Absence checked on all three surfaces (review responses, website copy, 5-star strengths)?
- Each gap scored on all four VOCA factors; ranking is descending by total score?
- Every claimed gap has a proof point from brand.md; `[verify]` and `[build first]` flagged separately?
- Messaging brief written for proven/verify gaps only, in the brand's real voice, with no banned words?
- Product/roadmap signal list included for `[build first]` gaps?
- Output saved to `./competitor-gaps/` if the user has multiple competitors or requested persistence?
