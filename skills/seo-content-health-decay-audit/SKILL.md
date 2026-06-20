---
name: seo-content-health-decay-audit
description: >
  Audits an existing content library for decay, cannibalization, and dead weight, then returns a
  ranked triage list of what to refresh, consolidate, redirect, or prune — and why, with the data
  to back each call. Takes a GSC export (and optional Ahrefs/Semrush + a sitemap or URL list),
  scores every URL on a four-part health model (traffic trend, ranking trend, engagement, link
  equity), clusters URLs that compete for the same query, and outputs a prioritized action plan a
  junior can execute. It does NOT manage brand context itself — it calls the `brand-brain` skill to
  load the active brand's ICP, positioning, and money pages so "what matters" is scored against real
  business value, not just clicks. It diagnoses and prioritizes; it hands the actual rewrite briefs
  to `serp-research` / `brief-builder`. Use when the user says "content audit," "content decay,"
  "decaying pages," "which posts to refresh," "keyword cannibalization," "content pruning,"
  "consolidate posts," "content health," "GSC audit," or hands over a GSC/Ahrefs/Semrush export and
  asks what's slipping and what to do about it.
---

# SEO Content Health & Decay Audit

Point it at your content library and a GSC export and it tells you the truth: which URLs are quietly bleeding traffic, which are cannibalizing each other, which are dead weight dragging down site quality, and — ranked by business impact — exactly what to do about each one. The output is a triage list, not a vibe. Every "refresh this" comes with the decline number that justifies it and the money-page relevance that prioritizes it.

This skill is the diagnostician. It decides *what* to fix and *in what order*; it does not write the refresh brief or the new article — it hands the winners to `serp-research` → `brief-builder` (or the Content-Decay Refresh Sweep) with the decay evidence attached.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's ICP, positioning, and money/conversion pages so URLs are scored on *business value*, not raw clicks. A decaying bottom-funnel page beats a decaying tangential listicle. Do not re-derive brand context here.
- **`competitive-intelligence-dossier`** *(optional, when investigating SERP-loss causes)* — to check whether a drop is a competitor displacing you vs. an algorithmic / intent shift.
- **Hand-off (downstream, not called inline):** the prioritized REFRESH and CONSOLIDATE items are handed to **`serp-research`** + **`brief-builder`** (or **`Content-Decay Refresh Sweep`**) to produce the actual rewrite briefs. This skill stops at the action plan.

---

## How a run works

```
Step 0  Load the brand   ──► call `brand-brain` (it bootstraps on first use) — get ICP, positioning, money pages
Step 1  Intake the data  ──► GSC export (required) + optional Ahrefs/Semrush + sitemap/URL list; set the decay window
Step 2  Score every URL  ──► the CHASM health model → a 0–100 health score + a decay verdict per URL
Step 3  Cluster & detect ──► group URLs that compete for the same query (cannibalization)
Step 4  Triage           ──► assign each URL one of five actions, ranked by business impact
Step 5  Present / persist ──► the ranked action plan; offer to save to ./reports/ and hand winners downstream
```

### Step 0 — Load the brand (always first)
**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the request and any named brand. It returns the digest: ICP, positioning line, and — critically — the brand's **money pages / primary conversion destinations**. Use these to weight the audit: a page is worth saving in proportion to how close it sits to revenue and how on-strategy its topic is. **Fallback if absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if none, ask for the brand's money pages and core topics before scoring.

### Step 1 — Intake the data
- **Required:** a Google Search Console export (Pages or Queries, with Clicks + Impressions + Position) covering **two comparable periods** (e.g. last 90 days vs. prior 90, or this period vs. same period last year — same length, avoid seasonal mismatch). If the user has only one period, ask for the comparison period or pull the trend from GA4 (see note below).
- **Optional, strongly improves accuracy:** Ahrefs/Semrush export (organic traffic, keyword count, position history, referring domains), GA4 landing-page report (sessions, engagement, conversions), and a sitemap or URL list to catch **zombie pages** that get zero impressions and so never show up in GSC.
- **Set the decay window** with the user (default: 90d vs. prior 90d). Decay is *relative decline*, so the window choice is load-bearing — name it in the report.
- *PushEngage note (this workspace):* GSC/GA4 tooling already lives at `~/ga4-query/`; the `pushengage-analytics` skill explains pulls and known data gotchas (broken `page_view`, the Jul'25–Mar'26 GSC hole). Prefer those scripts over rebuilding queries.

---

## The CHASM health model (the framework)

Score every URL 0–100 across five weighted signals. CHASM = **C**licks trend · **H**its in SERP (ranking) · **A**ttention (engagement) · **S**tanding (link equity) · **M**oney relevance. The first four are *how healthy is this page*; the fifth (from `brand-brain`) is *how much do we care* — it's the multiplier that turns "biggest drop" into "highest-priority drop."

| Signal | What it measures | Source | Weight | Healthy → Decaying |
|---|---|---|---|---|
| **C — Clicks trend** | Organic click delta, period vs. period | GSC | 30% | flat/up → −20%+ drop |
| **H — Hits / ranking** | Avg position delta for top queries | GSC | 25% | held/improved → slipped a page or off pg.1 |
| **A — Attention** | Engagement rate, time, conversions | GA4 | 20% | engaged → high bounce / zero conversions |
| **S — Standing** | Referring domains, link trend | Ahrefs/Semrush | 15% | links growing/stable → links lost, never earned |
| **M — Money relevance** | Closeness to ICP + money pages | brand-brain | 10% (× weight) | on-strategy/BOFU → off-topic/orphan |

Compute each signal on a 0–100 sub-scale; weight and sum. Then read the **decay verdict** off Clicks + Hits together — the two signals that separate the four failure modes apart:

| Verdict | Pattern | Likely cause |
|---|---|---|
| **Healthy** | Clicks flat/up, position held | Leave it. Don't touch winners. |
| **Slow decay** | Clicks down 20–50%, position slipping | Content staleness / freshness loss → **refresh** |
| **Cliff** | Clicks down 50%+ over a short window | Algo hit, SERP feature loss, or competitor → **investigate, then refresh/rebuild** |
| **Cannibalized** | Impressions exist but split across URLs, position volatile | Two+ pages competing → **consolidate** (see below) |
| **Zombie** | Near-zero clicks AND near-zero impressions | Never ranked / thin / orphan → **prune or merge** |
| **Decayed-out** | Clicks ≈ 0 now, had meaningful traffic before, not recoverable | Topic dead / product sunset → **redirect or retire** |

**Always read the cause before prescribing.** A cliff that's a competitor displacing you (check via `competitive-intelligence-dossier`) needs a different fix than a cliff from a stale publish date or a lost featured snippet.

---

## Cannibalization detection

Two URLs cannibalize when they rank for the **same primary intent** and trade or split impressions. Detect it, don't guess at it:

1. From the GSC **Queries × Pages** view, group queries where **2+ URLs from your site appear** for the same (or near-duplicate) query.
2. Confirm it's true cannibalization, not healthy coverage: same *intent* (both want to be *the* answer), overlapping position history, and neither cleanly winning. Different intents on the same query (e.g. a guide vs. a comparison) are **not** cannibalization.
3. Pick the **canonical winner** — the URL with the better position, more links, closer money relevance, or stronger conversion path.
4. Prescribe: **consolidate** the loser into the winner (301 + merge the unique content), or **differentiate** the two by retargeting one to a distinct intent. Note expected effect: consolidation usually lifts the survivor.

---

## Step 4 — Triage: five actions, ranked

Assign every audited URL exactly one action, then **rank the whole list by business impact** = (traffic at stake) × (M — money relevance) × (recoverability). High-impact, recoverable, on-strategy URLs go to the top.

| Action | When | Hand-off |
|---|---|---|
| **REFRESH** | Slow decay / recoverable cliff on an on-strategy URL | → `serp-research` + `brief-builder` (with decay evidence) |
| **CONSOLIDATE** | Cannibalized cluster | → merge loser→winner, 301; brief the survivor |
| **REDIRECT / RETIRE** | Decayed-out, topic dead, product sunset | → 301 to nearest relevant page or remove + 410 |
| **PRUNE** | Zombie / thin / no business value | → noindex or delete to lift overall site quality |
| **LEAVE** | Healthy | → no action; protect it (note it so nobody "optimizes" a winner) |

Be willing to recommend **pruning** — cutting dead content is a real SEO lever (site-quality signal), and a junior's instinct is to refresh everything. Don't.

---

## Output format

```
# Content Health & Decay Audit — [brand] · [decay window]
Brand: [slug, via brand-brain] · Money pages weighted: [list]
Data: [GSC ✓ · Ahrefs/Semrush ? · GA4 ? · sitemap ?] · Window: [period vs. period]
[⚠ data-quality caveats, e.g. GSC gap, one-period-only]

## Triage summary
| URLs audited | Refresh | Consolidate | Redirect/Retire | Prune | Leave |

## Prioritized action plan  (ranked by business impact)
| # | URL | Health (CHASM) | Verdict | Clicks Δ | Pos Δ | Action | Why (one line) |

## Cannibalization clusters
| Query/intent | Competing URLs | Canonical winner | Action |

## Recommended hand-offs
- Top N REFRESH/CONSOLIDATE → serp-research + brief-builder (decay evidence attached)

## Method note
- Window, signal weights, and any [verify] / data gaps
```

Lead with the **top ~10 actions**, not all 300 rows — an audit nobody can act on is a vanity deliverable. Offer the full table as an appendix or a saved file.

---

## Step 5 — Persist & hand off

Offer to save the report to `./reports/[slug]-content-audit-[YYYY-MM-DD].md` (and the full URL table as CSV alongside if large). Never write inside the skill folder or touch `brand.md`. Then offer to push the top REFRESH/CONSOLIDATE items into `serp-research` → `brief-builder` so the diagnosis becomes briefs — carrying the decay evidence (clicks Δ, lost queries, canonical winner) so the briefer rebuilds *toward what was lost*.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** Money relevance comes from the brain; without it you'll prioritize clicks over revenue. No scoring before it returns.
- **Decay is relative.** It's a *trend* vs. a comparable prior period — never an absolute-traffic snapshot. Name the window.
- **Read the cause before the cure.** Stale freshness, algo hit, SERP-feature loss, and competitor displacement need different fixes. Don't prescribe blind.
- **Cannibalization is same-intent, not same-keyword.** Two pages on one query with different intents is coverage, not conflict.
- **Pruning is a tool.** Recommend cutting dead weight; don't refresh-everything by reflex.
- **Protect winners.** Flag healthy high-value pages as LEAVE so nobody "optimizes" them into a decline.
- **Rank by impact, not by drop size.** Biggest decline ≠ highest priority — weight by business value and recoverability.

## What Not to Do

- Don't score or prioritize before `brand-brain` returns the money pages and ICP.
- Don't reimplement brand/ICP/positioning here — call `brand-brain`.
- Don't write the rewrite brief or the new article — hand off to `serp-research` / `brief-builder`.
- Don't compare mismatched windows (seasonal traffic, unequal lengths) and call the gap "decay."
- Don't invent traffic, ranking, or link numbers — if a data source is missing, score what you have and mark the rest `[verify]`.
- Don't dump 300 rows with no ranking — that's data, not a decision.
- Don't recommend a 301 without naming the destination, or a prune without confirming zero business value.

## Quality checklist (self-review before presenting)

- `brand-brain` called; money pages + ICP loaded and used as the M multiplier?
- Two comparable periods used; decay window named; data-quality caveats stated?
- Every audited URL has a CHASM score, a decay verdict, and exactly one action?
- Cannibalization confirmed by same-intent (not just same-keyword), with a canonical winner per cluster?
- Action list **ranked by business impact**, not raw drop size; winners flagged LEAVE?
- Cause read before cure on every cliff (competitor check where relevant)?
- Top ~10 actions surfaced first; full table offered as save/appendix; hand-off to serp-research/brief-builder offered with evidence attached?
- All numbers real or `[verify]`; nothing invented?
