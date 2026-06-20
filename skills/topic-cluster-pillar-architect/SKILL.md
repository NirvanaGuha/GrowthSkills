---
name: topic-cluster-pillar-architect
description: >
  Turns a broad topic into a complete topical-authority blueprint — one pillar page plus a mapped
  set of supporting spoke (cluster) articles, each with a proposed URL, working title, target keyword
  and intent, and the bidirectional internal-linking logic that ties the cluster together. It builds
  the architecture an SEO team executes against: what to write, in what order, and how every page links
  to reinforce the pillar's authority. It does NOT manage brand context itself — it calls the
  `brand-brain` skill to load the active brand's positioning, ICP, and offer so the cluster is
  differentiated and conversion-aware (not a generic keyword list), and it hands each spoke off to the
  content-brief / SERP-research stage rather than writing the articles. Use whenever the user says
  "topic cluster," "pillar page," "hub and spoke," "content cluster," "build a cluster around X,"
  "topical authority for X," "pillar and supporting articles," "internal linking plan," "content map
  for [topic]," or hands over a broad theme and asks how to structure content around it.
---

# Topic Cluster & Pillar Architect

Give it a broad topic; get the map a team can execute for months. One **pillar page** that owns the head term, a ring of **spoke articles** that each own a long-tail sub-topic, and the **internal-linking contract** that makes Google read the whole set as one authoritative entity. Every node is shaped by the brand's real positioning, ICP, and offer — because brand context comes from the shared `brand-brain` skill, not from guessing.

This skill architects. It does not research individual SERPs in depth, write briefs, or draft articles — it produces the structure those stages execute against, and it names exactly which downstream skill picks up each spoke.

---

## Skills this calls

- **`brand-brain`** (required) — resolves and loads the active brand's positioning, ICP + awareness tendency, offer mechanics + destinations, and voice. The cluster's angle, prioritization, and conversion mapping all key off this. Do not reimplement brand resolution/scanning here.
- **`competitive-intelligence-dossier`** (optional) — when a competitor owns the topic, pull their cluster coverage to find the white-space spokes worth owning instead of duplicating.
- **`positioning-messaging-architect`** / **`icp-persona-builder`** — already folded into `brand.md` via `brand-brain`; read those sections to ensure the pillar frames the topic in the brand's category and language, not a generic one.
- **`offer-pricing-brain`** — read offer/destinations (in `brand.md`) so the conversion-intent spokes and pillar CTA point at real destinations.
- **Downstream handoff (name, don't run):** each spoke is handed to **`serp-research`** then **`brief-builder`** (or the library's *SERP Analysis* / *Content Brief Builder*); the pillar's CTA can be sharpened with **`cta-variant-generator`**. This skill stops at the map.

---

## How a run works

```
Step 0  Load the brand   ──► call the `brand-brain` skill (it bootstraps on first use)
Step 1  Frame the topic  ──► is this a pillar-worthy hub? scope it
Step 2  Build the map    ──► pillar + spokes via the HUB framework
Step 3  Wire the links   ──► bidirectional internal-link contract
Step 4  Sequence + present (offer to persist the blueprint)
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the topic and any named brand. It returns the active brand's digest — positioning line + market category, ICP + awareness tendency, offer mechanics + destination URLs, voice adjectives + banned words — and the path to `brand.md`. If the brand is new, `brand-brain` bootstraps it first. **Do not architect anything until it returns.**

Use the positioning to decide *the angle the pillar owns*, the ICP to decide *which sub-topics matter*, and the offer to decide *which spokes carry conversion intent*. Obey voice + banned words in every proposed title.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask for the brand's category/positioning, ICP, and primary offer destination, then proceed. Always prefer the call.

### Step 1 — Frame the topic (pillar-worthiness gate)

Not every input is a pillar. Before mapping, classify the topic so the architecture is right-sized:

- **Pillar-worthy?** The head term must be broad enough to fan out into 6+ distinct sub-topics, with real search demand and clear relevance to the brand's ICP and offer. Too narrow ("best send time for push") → it's a *spoke*, not a pillar; offer to architect the parent cluster instead.
- **One pillar or a family?** If the topic spans two genuinely different head terms (e.g. "web push" vs "email automation"), say so and propose two clusters rather than one bloated hub.
- **Pillar type.** Pick the right pillar format — it determines spoke shape:

| Pillar type | When | Spokes are… |
|---|---|---|
| **Guide / "what is"** | Educational head term, problem-aware ICP | definitional, how-to, sub-concept explainers |
| **Resource hub** | Many parallel sub-topics, broad category | one spoke per sub-topic, lightly opinionated |
| **Comparison / "vs"** | Solution-aware ICP evaluating options | alternatives, vs-pages, use-case fits |
| **Product/use-case** | Bottom-funnel, tied to the offer | feature, integration, industry, JTBD pages |

---

## The HUB framework (the named method)

Build every cluster with **HUB — Head, Universe, Bridges**. It is the discipline that turns a keyword dump into a defensible topical-authority structure.

### H — Head: define the pillar

The pillar page targets the **head term** and exists to comprehensively cover the topic at breadth, not depth — it links *out* to spokes for depth. Specify:

- **Head keyword + intent** (usually informational/broad; the term the whole cluster competes for).
- **Pillar angle** — the one sentence that makes this the brand's pillar, not a generic encyclopedia entry. Pull it from positioning (e.g. frame "customer retention" through the brand's category and value lens). This is the load-bearing differentiation decision.
- **Proposed URL** — short, keyword-led, folder-rooted so the cluster nests under it (`/topic/` as the hub). See URL rules below.
- **Working title + H2 skeleton** — each H2 previews a spoke and links to it.
- **Pillar CTA** — the single conversion action, pointed at a real offer destination from `brand.md`.

### U — Universe: map the spokes

Enumerate the sub-topics that *together* cover the head term's universe. Drive coverage from intent, not from a keyword tool's autocomplete — group candidate sub-topics across the **search-intent spread** so the cluster isn't all one type:

| Intent band | Role in the cluster | Awareness |
|---|---|---|
| **Informational — definitional** | "what is / why" spokes that feed the pillar's authority | unaware → problem-aware |
| **Informational — how-to** | tactical spokes; highest volume, easiest wins | problem-aware |
| **Commercial — comparison** | "vs / alternatives / best" spokes near the decision | solution-aware |
| **Transactional — use-case/product** | spokes that map to the offer; carry the strongest CTA | product-aware |

For each spoke, specify: **target keyword + intent**, **working title** (on-voice, no banned words), **proposed URL**, **why it earns a place** (one line — distinct intent, not a near-duplicate of another spoke), and **priority** (see sequencing). Aim for **6–12 spokes** for a first build; flag obvious duplicates and merge them rather than padding the count. Mark any volume/demand claim `[verify]` unless it came from real data the user supplied.

### B — Bridges: the internal-linking contract

The architecture *is* the links. State the contract explicitly so executors can't get it wrong:

- **Pillar ⇄ every spoke (bidirectional).** Pillar links down to each spoke from the relevant H2; every spoke links back up to the pillar with a consistent, descriptive anchor. This bidirectional ring is what makes Google read the set as one entity.
- **Spoke → spoke (selective).** Link spokes to each other *only* where the topics genuinely relate (e.g. a "how-to" links to the "vs" page a reader logically needs next). Don't cross-link everything — over-linking dilutes signal.
- **Anchor discipline.** Descriptive, varied, keyword-aware anchors — never "click here," never the exact same anchor every time (avoid over-optimization).
- **Funnel routing.** Informational spokes route readers *toward* commercial/transactional spokes and the pillar CTA, so the cluster moves people down the funnel, not just around it.
- **Orphan check.** Every spoke must have ≥1 inbound and ≥1 outbound internal link. No orphans.

Express the contract as a small table or adjacency list — concrete enough to hand to a writer.

---

## Sequencing (what to build first)

Don't ship 12 articles in random order. Recommend a build order that compounds:

1. **Pillar first** — it's the destination every spoke links to; publish it (even thin, then expand) so spokes have an anchor.
2. **Quick-win spokes next** — the how-to / informational spokes with attainable difficulty, to start earning the cluster traffic and links.
3. **Conversion spokes** — comparison + use-case pages that monetize the audience the early spokes attract.
4. **Long-tail fill** — remaining definitional/edge spokes that round out coverage.

Tie each spoke to a priority (P1/P2/P3) on a simple value × attainability read; if the user supplied real difficulty/volume data, use it, otherwise reason from SERP intent and mark estimates `[verify]`.

---

## URL & title rules

- **URLs:** lowercase, hyphenated, keyword-led, no stop-word noise, no dates. Nest spokes under the pillar folder where it reflects true hierarchy (`/web-push-notifications/` pillar → `/web-push-notifications/best-practices/`), or keep a flat blog structure if that's the brand's existing convention — match what `brand.md` / the live site already does rather than inventing a new scheme. Flag any change to existing URL structure as a migration decision, not a silent default.
- **Titles:** on-voice, intent-matched, no banned words, no clickbait the page can't pay off. The pillar title signals breadth ("The complete guide to…"); spoke titles signal the specific job.

---

## Output format

```
## Topic cluster — [head topic]
Brand: [slug, via brand-brain] · Pillar type: [type] · Angle: [one-line differentiated frame]

### Pillar
| Field | Value |
|---|---|
| Head keyword / intent | … |
| Proposed URL | … |
| Working title | … |
| H2 skeleton (→ spokes) | … |
| CTA → destination | … (real offer URL) |

### Spokes
| # | Spoke (working title) | Target keyword | Intent | Proposed URL | Why it earns a place | Priority |
|---|---|---|---|---|---|---|

### Internal-linking contract
- Pillar ⇄ spokes: [the rule + anchor guidance]
- Spoke ↔ spoke: [the specific cross-links that make sense]
- Funnel routing: [how informational routes to conversion]

### Build sequence
P1 → P2 → P3, with the one-line rationale per wave.

### Handoff
Next step per spoke: → serp-research → brief-builder. Pillar CTA → cta-variant-generator.
```

Offer to **save the blueprint** to `./content-clusters/[slug]-[topic]-cluster.md` (or the project's content folder). Never write inside the skill folder, never touch `brand.md`.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No architecture before `brand-brain` returns. Positioning decides the angle; ICP decides the spokes; the offer decides the conversion routes.
- **Architecture, not a keyword list.** Every spoke must earn its place with a *distinct* intent — if two spokes would target the same intent, merge them.
- **The pillar covers breadth; spokes cover depth.** Don't let the pillar try to rank for everything, and don't let a spoke balloon into a second pillar.
- **The links are the cluster.** A map without an explicit bidirectional linking contract is just a list. Always specify the bridges.
- **Match the existing URL structure.** Respect the brand's live conventions; flag, don't silently impose, a new scheme.
- **Truth discipline.** Real volume/difficulty data or `[verify]`. Never invent search volume, rankings, or competitor coverage.

## What Not to Do

- Don't architect before `brand-brain` returns the active brand.
- Don't reimplement SERP research, briefing, or drafting — hand each spoke to the downstream skill by name.
- Don't pad the spoke count with near-duplicate intents; quality of coverage beats quantity.
- Don't propose orphan pages, "click here" anchors, or every-page-links-to-every-page over-linking.
- Don't invent search volume, keyword difficulty, or competitor cluster coverage — mark estimates `[verify]`.
- Don't build one bloated hub for two genuinely different head terms — propose separate clusters.

## Quality checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded (or bootstrapped) before any mapping?
- Topic passed the pillar-worthiness gate (broad enough, ICP-relevant, one head term)?
- Pillar has a *differentiated* angle from positioning — not a generic encyclopedia frame?
- 6–12 spokes, each with a distinct intent across the informational→transactional spread, on-voice titles, no banned words?
- Internal-linking contract is explicit (pillar ⇄ spokes bidirectional, selective spoke↔spoke, funnel routing, no orphans)?
- Proposed URLs match the brand's existing structure (or migration is flagged)?
- Build sequence given (pillar → quick wins → conversion → fill) with priorities?
- Every volume/difficulty estimate marked `[verify]`; each spoke names its downstream handoff?
