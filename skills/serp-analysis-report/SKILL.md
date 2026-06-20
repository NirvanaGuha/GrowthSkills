---
name: serp-analysis-report
description: >
  Turns a target keyword into a decision-ready SERP Analysis Report — a snapshot of who owns the
  top 10, what SERP features are live (AI Overview, People Also Ask, featured snippet, video, local
  pack, shopping), the dominant content format and structure every ranker shares, the topics they
  ALL leave on the table, and the specific angle this brand can win on. It reads the live results,
  classifies search intent, and reverse-engineers the "table stakes vs. differentiator" split so a
  junior operator knows exactly what to match and where to swing. It does NOT manage brand context
  itself — it calls the `brand-brain` skill to load the active brand's voice, ICP, positioning, and
  proof so the recommended wedge is real and on-strategy (and multi-brand aware). Use whenever the
  user says "analyze the SERP," "SERP analysis," "what's ranking for X," "look at the top 10 for X,"
  "what do I need to beat for this keyword," "SERP snapshot/report," "content gap for X," or hands
  over a keyword and asks how to win the page. It produces the analysis and the angle — it does not
  write the brief or the article (it hands off to those).
---

# SERP Analysis Report

Hand it a keyword, get back the page you have to beat — decoded. This skill reads the live top 10, names the search intent, maps the SERP features competing for the click, finds the format and structure every ranker shares, surfaces what they all miss, and recommends the one angle this brand can actually win on. It turns "go research the SERP" into a one-page report a writer or strategist can act on the same hour.

It analyzes and recommends a wedge. It does **not** write the content brief, the outline, or the article — those are downstream skills it hands off to. If the keyword's intent doesn't fit the brand or the offer, it says so instead of forcing a doomed page.

---

## Skills this calls

- **`brand-brain`** (required) — resolves and loads the active brand: positioning, ICP + awareness stage, real proof, voice, banned words, and offer/destinations. The wedge recommendation is grounded in *this brand's* real differentiators and proof, never invented. This skill does not implement brand resolution/scanning/interviewing — that lives in `brand-brain`.
- *(optional, when installed)* `competitive-intelligence-dossier` — when a ranking domain is a tracked competitor, pull its battlecard so the "why we win" angle is sharp; `proof-vault` — to attach a real stat/quote to the recommended differentiator. Synthesize inline (and mark `[verify]`) when absent.
- **Hands off to** (it does not do their job): a content-brief builder (the report is the input to the brief), a topic-cluster/pillar architect (related-keyword spillover), and an AEO/GEO optimizer (when AI Overview / PAA dominate the SERP).

---

## How a run works

```
Step 0  Load the brand   ──► call the `brand-brain` skill (it bootstraps on first use)
Step 1  Gather the SERP  ──► read the live top 10 + features for the keyword (locale-aware)
Step 2  Run the SERP-10 Teardown (Intent → Format → Feature → Gap → Wedge)
Step 3  Assemble the report, self-review, then present / offer to persist + hand off
```

### Step 0 — Load the brand (always first)
**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the keyword and any named brand. It returns the active brand's digest — positioning line, ICP + awareness tendency, real proof, voice + banned words, offer mechanics + destinations — and the path to `brand.md`. If the brand is new, `brand-brain` bootstraps it first. **Do not recommend a wedge until it returns** — the differentiator must be the brand's real one, not a guess.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if none, ask the user for positioning + ICP + one real differentiator, then proceed. Always prefer the call.

### Step 1 — Gather the SERP
Pull the **live** top 10 organic results plus the SERP features for the exact keyword. Confirm **locale + device** first (US/desktop default; ask if the brand serves a specific market) — the SERP changes by both. Capture for each result: rank, title, URL, domain, format (see below), and visible structure (headers, schema cues, word-count band, recency). Capture which **SERP features** are present and where they sit relative to organic #1. If you cannot fetch live results, say so plainly and ask the user to paste the SERP — **never fabricate rankings**.

---

## The SERP-10 Teardown (the framework)

Five passes. Each answers one question, and each feeds the next. Skipping a pass is how juniors produce confident-but-wrong reports.

### Pass 1 — Intent: what does this query actually want?
Classify the **dominant** intent from what's *winning*, not from the keyword's grammar. The ranking pages are Google's answer to "what satisfies this searcher."

| Intent | Tell (what ranks) | Right response |
|---|---|---|
| Informational | guides, how-tos, definitions, listicles | teach better/faster; earn the snippet |
| Commercial investigation | "best/top," comparisons, "X vs Y," reviews | a stronger comparison/buying frame |
| Transactional | product/pricing/category pages, "buy/pricing" | a conversion page, not a blog post |
| Navigational | one brand owns it | usually don't chase — note and move on |

**Mixed SERP rule:** if the top 10 splits (e.g., 6 guides + 3 product pages), the intent is **mixed** — note the split and the format that owns the *top* slots, because that's the table-stakes format. **Intent mismatch is a kill signal:** if winning intent doesn't match a page the brand can credibly publish, say so in one line and stop — don't recommend a doomed page.

### Pass 2 — Format & structure: what shape is required to compete?
Find the **dominant content format** (the one most of the top 10 share) — listicle, ultimate guide, comparison/vs, tool/calculator, video, forum/UGC, product page. Note the **structure pattern** they share: typical word-count band, header depth, presence of a comparison table, FAQ block, original data, images/video, schema. This is **table stakes** — what you must match to be *eligible*. Flag outliers (a thin page punching above weight, a page ranking on backlinks alone) — they hint at exploitable weakness.

### Pass 3 — Features: what's competing for the click before organic #1?
Map every live SERP feature and its impact:

| Feature | What it means for you |
|---|---|
| AI Overview / SGE | clicks compressed — win by being *citable* (clear, extractable passages); flag for the AEO/GEO skill |
| Featured snippet | a winnable position-zero — note its current format (paragraph/list/table) and target that exact shape |
| People Also Ask | a free outline of must-answer sub-questions — harvest them as required H2s |
| Video / image pack | format signal (searcher wants to *watch*) — plan or embed accordingly |
| Local pack / shopping | commercial/local intent confirmed — an article may be the wrong asset |
| Sitelinks / brand dominance | an entrenched incumbent — note the difficulty honestly |

The feature layout tells you both the **difficulty** (how far organic #1 sits below the fold) and the **opening** (an unowned snippet, weak PAA answers).

### Pass 4 — Gap: what do they ALL leave out?
The money pass. Read for what's **missing across the whole top 10**, not what one page does well. Look for: an unanswered PAA question, a missing comparison/table, no original data or proof, no recency (all stale), no expert/practitioner POV, a thin section everyone glosses, a persona the SERP ignores, a missing format (no one made the calculator/template/video). List the 3–5 strongest gaps. A gap only counts if it's both **real** (genuinely absent) and **wanted** (the intent implies the searcher cares).

### Pass 5 — Wedge: which gap can THIS brand win on?
Intersect the gaps with the brand (from `brand-brain`): the wedge is a gap the brand can fill **more credibly than any incumbent** because of its real positioning, ICP knowledge, or proof. Name **one primary wedge** + one backup. Anchor it to a real differentiator or proof point (pull from `proof-vault` / `competitive-intelligence-dossier` when present; mark `[verify]` otherwise — never invent a stat to fill a gap). State the recommended **angle in one sentence** a writer can build from.

---

## The report (output)

```
# SERP Analysis — "[keyword]"
Locale/device: [US · desktop] · Pulled: [date] · Brand: [slug, via brand-brain]

## Verdict (3 lines)
Intent: [type + mixed split] · Table-stakes format: [format + structure] · Recommended wedge: [one sentence]

## Top 10 snapshot
| # | Domain | Title | Format | Structure cues (len/headers/table/FAQ/data/recency) | Notable strength/weakness |

## SERP features
[feature → presence → implication; flag AI Overview / snippet / PAA explicitly]

## What every ranker shares (table stakes — match these to be eligible)
- format · depth · required sections · schema/proof present

## Content gaps (what they all miss — ranked)
1–5, each: the gap · evidence it's absent · why the searcher wants it

## People Also Ask → required sub-questions
[harvested PAA as the must-answer H2 list]

## The wedge (recommended angle)
Primary: [gap × brand differentiator] — angle in one sentence · proof to attach [or [verify]]
Backup: […]
Difficulty: [low/med/high + why] · Snippet opportunity: [yes/no + target format]

## Hand-off
→ Content Brief Builder (this report is the input) · → AEO/GEO (if AI Overview/PAA dominate) · → Topic-Cluster (related-keyword spillover)
```

Keep the table-stakes vs. gap distinction explicit — that split *is* the value. Offer to save the report to `./serp/[slug]-[keyword-slug].md` (never inside the skill folder, never touch `brand.md`), and offer the hand-off to the brief builder.

---

## Principles

- **Brand-brain first.** No wedge before `brand-brain` returns. The differentiator is the brand's real one.
- **Read what's winning, not the keyword's grammar.** Intent comes from the ranking pages.
- **Table stakes ≠ differentiator.** Separate "must match to be eligible" from "where you win." Both are required; they are not the same list.
- **The gap is the product.** The most valuable pass is what they *all* miss — and only gaps that are both real and wanted count.
- **Features change the game.** An AI Overview or owned snippet changes both the difficulty and the play; never report a SERP as if it's ten blue links.
- **Honest difficulty.** Call an entrenched SERP entrenched; flag intent mismatch and recommend not chasing it.
- **Truth only.** Real rankings (or ask for a paste), real proof, or `[verify]`. Never fabricate the SERP or a stat to fill a gap.

## What not to do

- Don't recommend a wedge before `brand-brain` returns the active brand.
- Don't invent rankings, SERP features, competitor content, or proof — if you can't fetch live, ask for a paste.
- Don't write the brief, outline, or article — analyze and hand off.
- Don't classify intent from the keyword alone, or report a single "best" page instead of the shared pattern.
- Don't recommend a generic gap any competitor could also fill — the wedge must be *this brand's*.
- Don't ignore the AI Overview / snippet / PAA layer; don't force a blog post onto a transactional/local SERP.

## Quality checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or bootstrapped) before the wedge?
- Locale/device confirmed; top 10 pulled live (or pasted) — not invented?
- Intent classified from what ranks, with the mixed-SERP split noted if present?
- Table-stakes format + structure stated separately from the gaps?
- SERP features mapped, with AI Overview / featured snippet / PAA called out and their implications named?
- 3–5 gaps that are both real and wanted; PAA harvested into required sub-questions?
- One primary wedge tied to a real brand differentiator + proof (or `[verify]`), plus a backup; honest difficulty rating?
- Persistence + hand-off to the brief builder offered?
