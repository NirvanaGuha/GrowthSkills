---
name: keyword-research-clustering-suite
description: >
  Turns seed topics, a domain, and an Ahrefs/Semrush/GSC keyword export into a clean keyword map —
  topic clusters scored by volume, difficulty, and search intent, plus a ranked quick-win opportunity
  list of keywords worth chasing first. Built on a pillar/cluster topic-model and an intent x effort
  prioritization matrix, so a junior operator produces a senior SEO's keyword strategy: which clusters
  to own, which page type each cluster maps to, and the 10-20 terms that move traffic this quarter.
  It does NOT manage brand context itself — it calls the `brand-brain` skill to load the active brand's
  ICP, positioning, and offer so clusters are relevant to the buyer (not just high-volume), and pulls
  persona language and the competitor keyword footprint from sibling skills when present. Use whenever
  the user says "keyword research," "cluster these keywords," "build a keyword map," "topic clusters,"
  "what should we rank for," "find quick wins," "group my Ahrefs/Semrush export," "keyword gap," or
  hands over a keyword export / list of seed topics and asks what to target. It plans what to rank for —
  it does not write the article (hand clusters to a brief/content skill for that).
---

# Keyword Research & Clustering Suite

Give it seed topics, a domain, and a keyword export — get back a keyword map a content team can build a quarter of work from: topic clusters scored by volume, difficulty, and intent, each mapped to a page type, plus a ranked quick-win list of the terms to chase first. Every cluster is filtered through the brand's real ICP and offer, so you rank for what the *buyer* searches, not just what's high-volume — because brand context comes from the shared `brand-brain` skill, not from guessing.

This skill plans what to rank for. It does not write the article, audit existing decay, or place backlinks — it hands a clean, prioritized cluster map to whatever produces content from it.

---

## Skills this calls

- **`brand-brain`** (required) — resolves and loads the active brand's ICP, positioning, offer mechanics, and banned words. Cluster relevance and intent scoring lean on this; do not re-derive brand context here.
- *(optional, when installed)* `icp-persona-builder` for persona vocabulary (the exact phrasing each persona searches with — feeds intent + modifier mining); `competitive-intelligence-dossier` for the competitor keyword footprint (what rivals already rank for → gap detection). Synthesize inline from the export + brand.md when absent.
- *(downstream)* hand approved clusters to a content-brief / `serp-research` skill to actually build pages. This skill stops at the map.

---

## How a run works

```
Step 0  Load the brand    ──► call the `brand-brain` skill (it bootstraps on first use)
Step 1  Ingest + normalize ──► seed topics + domain + export → one clean keyword table
Step 2  Cluster            ──► group into pillar/cluster topics (PILLAR — our working checklist)
Step 3  Score + label      ──► volume · difficulty · intent · brand-fit per cluster
Step 4  Prioritize         ──► the Quick-Win Quadrant → ranked opportunity list
Step 5  Present + persist   ──► cluster map + quick-win list, offer to save
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the request and any named brand. It returns the active brand's digest — ICP + awareness tendency, positioning line, offer mechanics, voice + banned words — and the path to `brand.md`. New brand → it bootstraps first. **Do not cluster or recommend before it returns.** Use the ICP and offer to judge *relevance* (Step 3) and the persona/competitor companions if present.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if none, ask a 3-question mini-setup (what it sells · who the buyer is · the money page/offer URL) then proceed. Always prefer the call.

### Step 1 — Ingest & normalize

1. **Take what you're given.** Seed topics (required), the domain (for own-vs-gap context), and an export (Ahrefs / Semrush / GSC / a raw list). If there's no export, work from seeds + brand and clearly mark all metrics `[verify — no volume/KD source]` — never invent numbers.
2. **Normalize to one table:** `keyword · volume · KD/difficulty · current rank (if domain data) · source`. Map column names across tools (Ahrefs "KD", Semrush "KD%", GSC clicks/impressions/position). De-dupe, lowercase, strip tracking junk.
3. **Drop the noise:** zero/near-zero volume with no strategic value, brand-name terms of *other* companies (note as competitor signals, not targets), and obvious irrelevance to the offer.

---

## Step 2 — Cluster: PILLAR (our working checklist)

Group keywords by the **job the searcher is doing**, not by string similarity. "best push notification software" and "push notification tools" are one cluster (same job, one page); "what is web push" is a different cluster (learn, not buy). Walk PILLAR:

| | Step | What you do |
|---|---|---|
| **P** | **Parent topic** | Name the pillar — a broad theme the brand can credibly own (tie to positioning), e.g. *"Web Push Notifications."* |
| **I** | **Intent split** | Within the pillar, split keywords by intent (see ladder below) — informational vs. commercial don't share a page. |
| **L** | **Lump by job** | Cluster keywords that one page would satisfy. Test: *could a single URL rank for all of these and fully answer them?* If no, split. |
| **L** | **Label the head** | Pick the cluster's head term (highest-volume keyword that represents the job) — that's the target/title term. |
| **A** | **Assign page type** | Map each cluster to a page type (below). One cluster = one page = one primary keyword + its variants. |
| **R** | **Relate to pillar** | Note the internal-link relationship (cluster → pillar, cluster ↔ sibling) so the content plan ships as a hub, not orphans. |

**Page-type map (intent → asset):**

| Intent | Cluster looks like | Page type |
|---|---|---|
| Informational | "what is", "how to", "guide", "examples" | Blog / guide (top-of-funnel) |
| Commercial-investigation | "best", "top", "vs", "alternatives", "review" | Comparison / listicle / alternatives page |
| Transactional | "software", "tool", "pricing", "buy", "[category] for [use-case]" | Product / pricing / category landing page |
| Navigational | "[brand] login", "[brand] + feature" | Existing product page (rarely a new target) |

Output of Step 2 is a set of named clusters, each with a head term, member keywords, intent, and page type.

---

## Step 3 — Score & label each cluster

Score the **cluster** (use the head term + aggregate members), not loose keywords:

- **Volume** — sum or head-term volume. Bucket: High / Med / Low so it reads at a glance.
- **Difficulty (KD)** — head-term KD from the export. Cross-check against the domain's authority: a KD of 40 is easy for a strong domain, a wall for a new one. Note as *Easy / Moderate / Hard for this domain* — not just the raw number.
- **Intent** — from the ladder (one label per cluster; mixed-intent = split it).
- **Brand-fit (the relevance gate, 1-3)** — does this cluster reach *our buyer* and connect to *our offer*?
  - **3** = ICP searches it AND it maps to the offer/money page (e.g. category, comparison, bottom-of-funnel how-to).
  - **2** = ICP-adjacent / top-of-funnel; builds authority, converts indirectly.
  - **1** = high-volume but off-ICP or no path to the offer → deprioritize even if huge.
  This is what separates a senior map from a volume dump. Use the brand-brain ICP + offer to set it; never chase a fat-volume term with brand-fit 1.

**Intent ladder (label each cluster):**

| Modifier signal | Intent | Funnel stage |
|---|---|---|
| what / why / how / guide / ideas / examples | Informational | Awareness |
| best / top / vs / alternative / review / comparison | Commercial | Consideration |
| software / tool / platform / pricing / cost / buy / for [use-case] | Transactional | Decision |
| [brand] / login / dashboard | Navigational | Existing customer |

---

## Step 4 — Prioritize: the Quick-Win Quadrant

A quick win = **achievable difficulty × real demand × buyer relevance**, ideally where the domain already has a foothold. Plot clusters on Difficulty (for this domain) vs. Value (volume × brand-fit):

```
            VALUE (volume x brand-fit) →
        ┌─────────────────────┬─────────────────────┐
  LOW   │  QUICK WINS          │  BIG BETS           │
  KD    │  do first — easy +   │  plan — high reward, │
   ↑    │  valuable + on-ICP   │  needs authority/time│
        ├─────────────────────┼─────────────────────┤
  HIGH  │  FILL-INS            │  MONEY PITS         │
  KD    │  batch when idle     │  skip / revisit later│
        └─────────────────────┴─────────────────────┘
```

**Quick-win signals to rank the list (strongest first):**
1. Domain already ranks #5-20 for the head term (striking distance — a refresh, not a new page). Pull from GSC/export rank if present.
2. KD ≤ domain's comfortable ceiling AND brand-fit ≥ 2.
3. Transactional/commercial intent with brand-fit 3 (closest to revenue).
4. Long-tail with clear intent and low competition the export shows rivals ignoring (gap — confirm via `competitive-intelligence-dossier` if present).

Produce a **ranked quick-win list (10-20 keywords)**: keyword · cluster · volume · KD · intent · brand-fit · why-it's-a-win (one line) · page action (new page / refresh existing URL).

---

## Step 5 — Present & persist

**Default output is the cluster map + the quick-win list.** Lead with the quick wins (that's what gets acted on), then the full cluster map, then a one-line strategy read (which pillars to own this quarter, what to defer).

```
## Keyword map — [brand · pillar(s)]
Inputs: [N keywords from <source> · domain · seeds]   Brand: [slug, via brand-brain]

### Quick wins (do first)
| # | Keyword | Cluster | Vol | KD | Intent | Fit | Why it wins | Action |

### Cluster map
| Cluster (head term) | Intent | Page type | Vol | KD (for us) | Fit | # kws | Quadrant |
> cluster → pillar / sibling link notes per cluster

### Strategy read
- Own this quarter: [pillars] · Build authority then revisit: [...] · Skip: [money pits]
```

**Persist:** offer to save the full map to `./keyword-research/[slug]-keyword-map.md` (and the normalized table as a sibling `.csv` if useful). Never save inside the skill folder; never touch `brand.md`. Note that approved clusters are the handoff into a content-brief / `serp-research` skill.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No clustering or recommendation before `brand-brain` returns. Relevance is judged against the real ICP + offer, not vibes.
- **Cluster by job, not by string.** One page per searcher-job; mixed intent = split, never merge informational with transactional.
- **Relevance beats volume.** A 200-volume term that reaches the buyer beats a 20k term that doesn't. Brand-fit 1 stays deprioritized no matter the volume.
- **Difficulty is relative to the domain.** Always score KD against *this* domain's authority, not as an absolute.
- **Quick wins are where you already have a foothold.** Striking-distance and low-KD-high-fit lead the list.
- **Truth only.** Real volume/KD from the export, or `[verify]`. Never invent metrics, rankings, or competitor data.

## What Not to Do

- Don't cluster before `brand-brain` returns the active brand.
- Don't reimplement brand/ICP/competitor resolution here — call `brand-brain` (and siblings when present).
- Don't write the article, the brief, or the meta tags — this skill stops at the map; hand off downstream.
- Don't merge clusters that need different pages, or split one job across two pages (cannibalization).
- Don't invent volume, KD, or rankings when no export is supplied — mark `[verify]` and say so.
- Don't recommend a high-volume, off-ICP term as a quick win because the number is big.

## Quality checklist (self-review before presenting)

- `brand-brain` called and ICP + offer loaded (or bootstrapped) before any clustering?
- Export normalized to one table; column mappings (Ahrefs/Semrush/GSC) reconciled; metrics real or `[verify]`?
- Every cluster has a head term, single intent, page type, and a pillar/sibling link relationship?
- Each cluster scored on volume, domain-relative KD, intent, and brand-fit (1-3)?
- Quick-win list is ranked (10-20), each with a one-line why + a page action (new vs. refresh)?
- Strategy read names what to own, defer, and skip — and off-ICP volume traps are deprioritized, not chased?
- Output offered for saving to `./keyword-research/`, never inside the skill folder?
