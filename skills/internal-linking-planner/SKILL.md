---
name: internal-linking-planner
description: >
  Turns every new publish or page refresh into a deliberate PageRank distribution event.
  Takes a target URL (new post, refreshed article, landing page, or any page you care about)
  plus a sitemap, crawl export, or pasted URL list and returns a complete bidirectional internal
  linking plan: inbound links TO the target (from the highest-authority, most-topically-adjacent
  pages already on the site), outbound links FROM the target (to the pages it should be passing
  authority and relevance to), anchor-text recommendations for each, and a prioritized edit queue
  so the SEO value lands in the right order. Built on Google's Reasonable Surfer Model and
  Topic Authority Flow — a junior can execute it and get senior-level results. Also runs in
  audit mode: hand it any existing page and it surfaces wasted links, over-optimized anchors,
  and orphaned supporting pages.
  Trigger phrases: "internal links," "link this post," "anchor text plan," "internal linking
  audit," "orphaned page," "PageRank distribution," "link equity," "which pages should I link
  to," "silo this article," "link from my best pages."
---

# Internal Linking Planner

Internal links are PageRank distribution decisions. Every publish without a linking plan is a missed compounding event — authority that should flow to your target page disperses at random instead. This skill turns that into a deliberate, repeatable system.

Framework: **Reasonable Surfer × Topic Authority Flow**. Google weights internal links by the probability a real user would click them (Reasonable Surfer) and by whether the linking page is topically adjacent (Topic Authority Flow). This skill operationalizes both: it scores every candidate link by click-probability + topical proximity, then routes equity toward your target or away from it deliberately.

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads the active brand's site authority context, ICP, and positioning so anchor text stays on-voice and recommendations reflect the brand's actual content strategy.
- **`topic-cluster-pillar-architect`** (optional, call when available) — if the target page's cluster membership is unclear, delegate cluster mapping here rather than inferring it manually.
- **`content-gap-finder`** (optional) — when inbound-link candidates are thin, call this to surface supporting pages that should be created to strengthen the cluster before linking.
- **`technical-seo-audit-fix-prioritizer`** (optional) — if crawl anomalies (noindex, redirect chains, orphans) are surfaced during the audit, hand off to this skill for the fix queue.
- **`gsc-monitoring-suite`** (optional) — use to pull actual GSC impressions/clicks for candidate pages to weight scoring by real traffic potential rather than estimates.

---

## How a run works

```
Step 0  Load brand context ──► brand-brain (always first)
Step 1  Identify the mode  ──► Plan (new/refreshed page) | Audit (existing page)
Step 2  Build the candidate inventory
Step 3  Score and rank candidates (RSM × TAF matrix)
Step 4  Select anchors (anchor text protocol)
Step 5  Output the edit queue + bidirectional plan
Step 6  Self-review before presenting
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Use the returned digest for:
- Voice + banned words → anchor text must obey these; no keyword stuffing in anchors if the brand voice is natural/editorial
- ICP + positioning → link editorial direction; commercial pages should pull equity toward high-intent hubs
- Any companion `style-guide.md` → anchor formatting rules (title case? exact match? partial match?)

**Fallback if brand-brain is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`; if none exists ask the user for: site type, commercial priority pages, and whether they prefer partial-match or editorial anchors.

### Step 1 — Pick the mode

| Signal in request | Mode |
|---|---|
| New post, refreshed article, "link this," "I just published" | **Plan mode** — build the full bidirectional plan |
| "Audit my links," "check this page," "find orphans," "wasted links" | **Audit mode** — diagnose the existing link profile |

Default to Plan mode when ambiguous.

---

## The Framework: Reasonable Surfer × Topic Authority Flow

### Reasonable Surfer Model (RSM) — click-probability scoring

Google's Reasonable Surfer patent weights internal links by the probability a human would click them in context. High-probability links transfer more equity. Score each candidate link 1–5:

| RSM Score | What it means | Signals |
|---|---|---|
| 5 — High | User would almost certainly click | Relevant anchor in body prose, above-the-fold, early in content, contextual |
| 4 — Likely | User would probably click | Body prose, mid-page, clearly relevant anchor |
| 3 — Moderate | User might click | Late in content, general related topic, sidebar, related-posts widget |
| 2 — Low | User unlikely to click | Footer nav, boilerplate sidebar, generic "read more" |
| 1 — Negligible | Near-zero click probability | Site-wide nav links, breadcrumbs, pagination |

**Only recommend links scoring RSM 4 or 5.** RSM 3 is acceptable for internal hubs. RSM 1–2 are noted in audit mode as low-equity links not worth relying on.

### Topic Authority Flow (TAF) — topical proximity scoring

Links between topically adjacent pages amplify authority transfer for that topic cluster; links across unrelated clusters bleed relevance. Score topical proximity 1–5:

| TAF Score | Relationship |
|---|---|
| 5 — Same cluster | Pillar → supporting post, supporting → pillar, or supporting → supporting within same cluster |
| 4 — Adjacent cluster | Related clusters sharing a parent topic |
| 3 — Broad thematic overlap | Same general category, different theme |
| 2 — Weak connection | Different category, forced stretch |
| 1 — Unrelated | No topical overlap |

**Only recommend links scoring TAF 3 or above.** Flag TAF 1–2 as cross-cluster dilution if they appear in audit mode.

### Combined Link Value (CLV) score

```
CLV = RSM × TAF   (range: 1–25)
```

Prioritize the edit queue in descending CLV order. Minimum threshold for inclusion: CLV ≥ 12 (RSM ≥ 3 × TAF ≥ 4, or RSM ≥ 4 × TAF ≥ 3).

---

## Plan mode — building the bidirectional plan

### A. Inventory the candidates

Ask the user to provide (or infer from what's given):
- **Target URL + its primary keyword / topic** — the page you want to boost
- **Sitemap, crawl export, or URL paste** — the link universe to pull candidates from
- **Cluster the target belongs to** (or call `topic-cluster-pillar-architect` to determine it)
- **PageRank proxy** — any available signal: GSC clicks/impressions, Ahrefs/Semrush UR/DR, word count, number of inbound links already. If none, use: pillar pages > category hubs > high-traffic posts (stated by user) > other.

### B. Score every candidate page for inbound links TO the target

For each candidate on the site, evaluate:
1. **RSM** — if a link to the target were placed in this page's body prose, how likely would a reader click it given the page's topic and the target's topic?
2. **TAF** — how topically close is this candidate to the target?
3. **CLV** = RSM × TAF

Select the top 5–8 inbound candidates (CLV ≥ 12). More than 8 starts to look like a link scheme; fewer than 3 is likely under-linking for a new publish.

For each selected inbound link:
- Name the candidate page (URL + title)
- Recommended anchor text (see Anchor Text Protocol below)
- Exact placement guidance: "in the H2 section on [topic X], in the 3rd paragraph" or "after the comparison table, before the CTA"
- CLV score

### C. Score every candidate page for outbound links FROM the target

For each link the target page should send out:
- Same RSM × TAF scoring
- Bias toward: the cluster's pillar page (if the target is a supporting post), supporting posts that deepen a subtopic the target mentions, and commercial/money pages the target's ICP would naturally click next
- Avoid: linking away from the target to higher-authority pages in the same cluster unless the relationship is genuinely editorial (you'd want a reader to go there)

Select 3–6 outbound links. Too many outbound links dilutes the page's own equity; too few is a missed topical-authority signal.

### D. Output format

```
## Internal Linking Plan — [Target page title] ([URL])
Target cluster: [cluster name]  |  Target keyword: [primary keyword]

### Inbound links TO this page (add to existing pages)
| Priority | From page | Section / placement | Anchor text | CLV |
|---|---|---|---|---|

### Outbound links FROM this page (add to target)
| Priority | To page | Context in target | Anchor text | CLV |
|---|---|---|---|---|

### Edit queue (ordered by CLV, highest first)
1. [Page title] — [action: add inbound / add outbound] — [anchor] — CLV: [n]
   Placement: [specific guidance]
...

### Flags
- [Any orphan risk, over-linking, anchor over-optimization]
```

Save to `./seo/internal-links/[slug]-linking-plan.md` if the user asks to persist; otherwise deliver inline.

---

## Audit mode — diagnosing an existing page

Run when the user asks to audit a live page's current internal links.

**Five-point audit:**

1. **Inbound link count + CLV distribution** — how many inbound links does the page have? What is the average CLV? Flag if fewer than 3 high-CLV inbound links exist (under-linked).
2. **Orphan risk** — if the page has 0–1 inbound links from non-navigational pages, flag as orphaned or near-orphaned. Orphaned pages accumulate no equity regardless of content quality.
3. **Anchor text diversity** — count exact-match vs. partial-match vs. generic anchors. Over-optimization flag: >50% exact-match anchors on a single target keyword is a Penguin-era signal; recommend diversifying.
4. **Outbound link CLV audit** — are the outbound links going to topically adjacent, high-CLV destinations? Flag cross-cluster dilution (TAF ≤ 2) links as low-value.
5. **Broken / redirect chain links** — note any links that land on 301s or 404s (if crawl data is provided); hand off the fix list to `technical-seo-audit-fix-prioritizer`.

**Audit output:**

```
## Internal Link Audit — [Page title] ([URL])

### Health summary
Inbound links:   [n] total | Avg CLV: [n] | Under-linked: [yes/no]
Orphan risk:     [yes / borderline / no]
Anchor diversity: [over-optimized / healthy / too generic]
Outbound quality: [n] high-CLV | [n] cross-cluster dilution links

### Top 3 quick wins (ordered by impact)
1. ...
2. ...
3. ...

### Full findings
[Per-link breakdown with CLV scores and flags]
```

---

## Anchor Text Protocol

**The four anchor types (use all four across a page's inbound link set):**

| Type | Example | When to use |
|---|---|---|
| Exact match | "web push notifications" | 1–2 times max across all inbound anchors for a target keyword; use in highest-CLV link |
| Partial match | "best push notification software" | 2–3 times; broadens topical signal without over-optimization |
| Branded | "PushEngage's guide to push timing" | Natural in editorial; good for navigational intent pages |
| Generic / contextual | "this retention playbook," "see the full comparison" | Use for RSM 3 links and in-content transitions |

**Rules:**
- Never keyword-stuff the anchor; if the brand voice is editorial/conversational, match that register
- Anchor text must accurately describe the destination — do not use misleading anchors for RSM boost
- Match case to the surrounding prose; don't force title case in body text
- Banned words from brand-brain apply here — never use them as anchor text
- Vary anchors across the inbound set; identical anchor text on every inbound link is a red flag

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No anchor text written before brand context loads; banned words are hard overrides.
- **CLV drives the queue.** Only recommend links scoring CLV ≥ 12; lower-scoring links are noted as low-equity options, not primary recommendations.
- **Bidirectional by default.** Every plan includes both inbound (who should link to the target) and outbound (where the target should link to); one-directional plans are incomplete.
- **Placement specificity.** "Add a link somewhere on this page" is not useful. Every recommendation names the section and approximate paragraph.
- **Honest about data gaps.** When GSC/Ahrefs data is absent, score on proxy signals and flag the confidence level as `[estimated]`.
- **Cluster coherence over volume.** Fewer high-CLV links outperform many low-CLV links; never pad the list to look thorough.
- **Anchor diversity is required.** A natural inbound anchor profile uses all four anchor types; flag any over-optimized sets in audit mode.

---

## What Not to Do

- Don't recommend links below CLV 12 as primary recommendations; include them only as a flagged low-priority appendix.
- Don't reimplement brand context resolution — call `brand-brain`.
- Don't recommend more than 8 inbound links to a single target page in one plan — beyond that, prioritize the highest-CLV ones and note the rest as future opportunities.
- Don't write anchors that violate brand voice or use banned words.
- Don't conflate navigation links (header/footer/breadcrumb) with editorial body links — RSM 1–2 links provide negligible equity and should not be counted in the inbound link audit total.
- Don't generate a plan if the sitemap or URL list is missing — ask for it, or offer to proceed with a limited 5-URL set the user pastes.
- Don't recommend outbound links from commercial/money pages to blog posts unless the CLV is clearly ≥ 16; equity flows down the funnel, not up.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and voice + banned words loaded?
- Every recommended link scored with explicit RSM + TAF = CLV; no link below CLV 12 in the primary plan?
- Both directions covered: inbound list (5–8 pages) AND outbound list (3–6 destinations)?
- Every recommendation includes exact placement guidance (section + paragraph), not just the page?
- Anchor text set uses at least 3 of the 4 anchor types; no anchor uses a brand-banned word?
- Audit mode: orphan risk, anchor diversity, and cross-cluster dilution all evaluated?
- Any data gaps flagged as `[estimated]` rather than stated as fact?
- Edit queue is sorted by CLV descending (highest-impact edits first)?
