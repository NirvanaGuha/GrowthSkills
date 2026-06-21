---
name: content-gap-finder
description: >
  Identifies keywords and topics that competitors rank for in organic search that your domain does
  not — surfacing the highest-opportunity white space for content investment. Takes your domain plus
  up to three competitor domains (or a seed keyword set + competitor list) and returns a prioritized
  gap table scored by traffic opportunity, difficulty, and strategic fit to the brand's ICP. Three
  modes: Quick Gap (fastest path from competitor URLs to a ranked gap list), Deep Gap (full
  keyword-level analysis across all competitors with clustering), and Topic-Cluster Gap (maps gaps
  onto an existing or recommended topic-cluster architecture). Every output is ICP-filtered through
  brand-brain so you never chase traffic that misses your buyer. Use whenever the user says
  "content gap," "what keywords are we missing," "what should we write next for SEO," "competitor
  keyword gaps," "find topics we don't cover," "white-space analysis," "gap audit," or hands over
  competitor domains and asks where the opportunity is.
---

# Content Gap Finder

Your competitors are ranking for keywords your buyers type every day — and you have no content there. This skill finds those gaps, scores them by real opportunity, and tells you exactly which ones to build first based on your ICP, your cluster architecture, and the effort-to-impact ratio. The output is not a raw keyword dump. It is a ranked, actionable build list.

This skill surfaces gaps. It does not write the content. For writing, pass the gap table to `blog-post-drafting-engine`, `content-brief-builder`, or `topic-cluster-pillar-architect`.

---

## Skills this calls

- **`brand-brain`** (required) — resolves the active brand's ICP, positioning, and banned topics before any gap is scored; gaps irrelevant to the ICP are deprioritized, not discarded (they may be TOFU).
- **`competitive-intelligence-dossier`** (optional) — when a richer competitor profile is needed (product lines, audience, known strategy) to inform whether a gap is worth entering. Call when the user hasn't provided competitor context and the gaps look surprising.
- **`topic-cluster-pillar-architect`** (optional) — when the user wants gaps mapped onto a full cluster architecture rather than a flat ranked list. Call at the end of a Deep Gap or when requested.
- **`keyword-research-clustering-suite`** (optional) — when seed terms need expanding before gap analysis, or when the user wants clusters built from the gap list.
- **`seo-content-health-decay-audit`** (optional) — when you need to cross-reference existing content before labeling something a gap (avoid recommending content you already have but haven't checked).
- **`content-brief-builder`** (downstream) — pass the prioritized gap table here to turn individual gaps into full content briefs.

---

## How a run works

```
Step 0  Load brand context  ──► call brand-brain (ICP, voice, banned topics, positioning)
Step 1  Collect inputs       ──► your domain + competitors + any seed keywords / GSC data
Step 2  Pick the mode        ──► Quick | Deep | Topic-Cluster Gap
Step 3  Run the framework    ──► gap discovery → ICP filter → scoring → ranking
Step 4  Deliver the output   ──► gap table + build recommendations + next-step calls
```

### Step 0 — Load brand context (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). Use the returned ICP, positioning line, offer mechanics, and banned-word list to set two filters before scoring:

- **ICP relevance score** — a gap keyword's audience fit to the brand's ICP; penalizes high-volume terms that attract the wrong buyer.
- **Banned-topic exclusion** — suppress gaps the brand explicitly does not want to create content around (competitor features, regulated claims, off-positioning topics).

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If none exists, ask the user for: ICP job title + company type, positioning in one sentence, and any topics explicitly off-limits. Proceed with those as proxies.

### Step 1 — Collect inputs

Required:
- **Your domain** (e.g. `pushengage.com`)
- **Up to 3 competitor domains** (e.g. `onesignal.com`, `sendbird.com`)

Optional but improves scoring:
- GSC export (queries + impressions + clicks + position, last 90 days) — tells you which gaps you already have ranking signals for
- Ahrefs / Semrush keyword gap CSV export — use directly; skip the manual extraction if provided
- Seed keyword list — primes the competitor crawl scope

If only a domain is given (no competitor names), ask for competitors or offer to call `competitive-intelligence-dossier` to identify the top 3.

---

## The Framework: ICP-Filtered Opportunity Scoring (IFOS)

Standard keyword gap tools give you a raw list sorted by volume. IFOS re-ranks by **buying-audience density** — the share of searchers who match your ICP — multiplied by attainability. A 5,000-volume keyword your ideal buyer types is worth more than a 50,000-volume term your buyer never searches.

### The four scoring dimensions

| Dimension | What it measures | Data source |
|---|---|---|
| **Traffic ceiling** | Maximum monthly clicks the #1 position earns (estimated from volume × expected CTR at rank 1) | Tool export or estimate from volume |
| **Attainability** | 1 – (KD / 100); how likely you can rank without DA-equivalent authority. Penalize KD > 70 unless you already have topical authority. | KD from tool or estimated |
| **ICP fit** | 0.0–1.0 score: 1.0 = keyword maps to core ICP job title + problem + stage; 0.5 = adjacent buyer; 0.2 = general audience | Assessed against brand-brain ICP digest |
| **Strategic angle** | Bonus score (0–0.3): +0.1 if keyword maps to existing pillar/cluster; +0.1 if competitor ranks but has thin/outdated content (beatable); +0.1 if keyword maps to a brand proof point | Manual assessment |

**IFOS Score = Traffic Ceiling × Attainability × ICP Fit + Strategic Angle Bonus**

Normalize to 0–100 after computing the batch.

### Awareness-stage tagging

Every gap keyword gets tagged:
- **TOFU** — informational, brand-unaware (ICP fit 0.2–0.5 typically); high volume, lower conversion
- **MOFU** — solution-aware; comparison, "vs", "alternative", "best [category]"
- **BOFU** — product-aware or most-aware; brand + feature, pricing, reviews, direct sign-up intent

Prioritize BOFU and MOFU gaps for immediate content investment; queue TOFU by volume × brand-building value.

---

## Modes

### Quick Gap (default)

Best for: fast priority list, weekly content planning, first pass at a new competitor.

1. Load brand context (Step 0).
2. Ask for domain + up to 3 competitors (or accept what's provided).
3. If a tool export (Ahrefs/Semrush gap CSV) is provided, parse it directly. If not, reason from any seed keywords and known competitor content (use web search if available; otherwise note `[tool export recommended]` and work with what exists).
4. Apply ICP filter and IFOS scoring.
5. Return the **Top 15 gaps** table (keyword, monthly volume estimate, KD estimate, IFOS score, awareness stage, why it's a gap, recommended content type).
6. Recommend the top 3 to build first with one-line rationale each.

Output is inline. Save only if asked.

### Deep Gap

Best for: quarterly content audits, new market entry, establishing a topic cluster from scratch.

1. Load brand context + optionally call `seo-content-health-decay-audit` to pre-exclude already-covered URLs.
2. Collect full competitor keyword sets (tool export preferred; manual scope if not available).
3. Cluster gaps by topic (call `keyword-research-clustering-suite` if available, else group manually by head term + modifiers).
4. Score every cluster as a unit: aggregate IFOS across keywords in the cluster, weight by cluster head-term volume.
5. Return:
   - **Cluster-level gap table** (cluster name, estimated total cluster traffic, attainability, ICP fit, IFOS score, awareness stage, pillar or standalone)
   - **Top 5 clusters to prioritize** with build order rationale
   - **Individual keyword table** for the top 3 clusters (the exact terms to target)
   - **3 quick-win singles** — high-IFOS, low-KD keywords outside the top clusters that are fast to publish

Save to `./content-gaps/[brand-slug]-deep-gap-[YYYY-MM].md`.

### Topic-Cluster Gap

Best for: when the brand already has a cluster architecture or wants to build one from gaps.

1. Run Deep Gap first (or accept existing Deep Gap output).
2. Call `topic-cluster-pillar-architect` passing the gap cluster table and brand context.
3. Map each gap cluster to: existing pillar (extend), orphan cluster (new pillar needed), or supporting article (slot into existing cluster).
4. Return a visual map as a Markdown table: Pillar → Gap Cluster → Recommended Article Titles (3–5 per cluster).
5. Flag clusters with no existing pillar as new pillar candidates — include estimated effort (article count to establish authority).

Save to `./content-gaps/[brand-slug]-cluster-gap-[YYYY-MM].md`.

---

## Output format (all modes)

### Quick Gap table
```
## Content Gaps — [your-domain] vs [competitor-domains]
Brand: [slug] | ICP: [one-line from brand-brain] | Run date: [YYYY-MM-DD]

| # | Keyword | Vol/mo | KD | IFOS | Stage | Gap reason | Recommended content type |
|---|---|---|---|---|---|---|---|

### Top 3 to build first
1. [keyword] — [one-line rationale]
2. …
3. …

### Next step
- Pass to `content-brief-builder` for full briefs on the top picks.
- Pass to `topic-cluster-pillar-architect` if you want these mapped to a cluster plan.
```

Volume and KD values from tool exports are used as-is. Estimated values are marked `[est]`. Never present estimates as confirmed data.

---

## Principles

- **ICP filter before volume sort.** A 500-volume keyword your ideal buyer searches beats a 50,000-volume term they don't. Never present a raw volume-sorted list without ICP filtering.
- **Gaps, not ideas.** Every item must be a keyword a competitor demonstrably ranks for (or has measurable ranking signals toward) that you do not. If you can't verify the competitive ranking signal, mark it `[unconfirmed gap — verify in tool]`.
- **Awareness-stage discipline.** BOFU and MOFU gaps convert; TOFU gaps build audience. Tell the user which is which and what the appropriate expectation is for each.
- **Honesty on data quality.** Tool exports give real data. Without exports, estimates are directional only. Label them. Never present estimates as confirmed numbers.
- **Compose, don't duplicate.** When cluster mapping is needed, call `topic-cluster-pillar-architect`. When deeper competitive context is needed, call `competitive-intelligence-dossier`. Don't rebuild those skills here.
- **Brand-brain always first.** ICP filters, banned topics, and voice all come from brand-brain. Do not score gaps or produce output before it returns.

## What Not to Do

- Don't sort by volume alone and call it a gap analysis. Volume without ICP fit and attainability is a distraction list.
- Don't label a keyword a gap without confirming a competitor ranks for it (or flagging it as unconfirmed).
- Don't recommend content for keywords the brand explicitly excludes (check banned topics from brand-brain).
- Don't present estimated volume/KD as confirmed data. Label estimates.
- Don't produce output before brand-brain returns — gaps scored without ICP context are often wrong.
- Don't rebuild competitive profiling or cluster architecture — call `competitive-intelligence-dossier` and `topic-cluster-pillar-architect`.
- Don't conflate "topics we haven't written about" with "gaps" — a gap requires a competitor ranking signal, not just absence from your blog.

## Quality checklist (self-review before presenting)

- `brand-brain` called and ICP + banned-topic filter applied before any gap was scored?
- Every gap item has a named competitor that ranks for it (or is flagged `[unconfirmed]`)?
- IFOS score computed on all four dimensions (not just volume)?
- Awareness stage (TOFU/MOFU/BOFU) assigned to every keyword?
- Volume and KD labeled `[est]` if not from a tool export?
- Quick: top 15 returned with top 3 build-first recs? Deep: cluster table + top 5 + individual terms for top 3 + quick wins?
- Next-step skill calls suggested at the end of output?
- Saved to `./content-gaps/` if mode is Deep or Topic-Cluster?
