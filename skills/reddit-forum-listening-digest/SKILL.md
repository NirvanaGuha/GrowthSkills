---
name: reddit-forum-listening-digest
description: >
  Turns raw Reddit threads and forum posts into a structured weekly audience-intelligence digest
  for growth and content teams. Takes subreddit names, search queries, or pasted thread URLs and
  returns: (1) top threads ranked by signal value, (2) a recurring pain vocabulary table — the
  exact words and phrases buyers use before they know your product exists, (3) emerging objections
  not yet in your sales deck, and (4) a ready-to-use "voice of the customer" snippet bank for ads,
  emails, and landing pages. Uses the Pain Vocabulary Mining framework (a structured extraction
  method built on Schwartz awareness stages + Jobs-to-Be-Done language tagging) so a junior analyst
  produces insight a senior copywriter or strategist would trust. Brand context is loaded via
  brand-brain so all output is filtered against the brand's ICP and existing messaging. Saves the
  digest to ./research/reddit-digest-[YYYY-WW].md for weekly accumulation. Use when the user says
  "reddit listening," "what are people saying about [topic]," "pain vocabulary," "forum research,"
  "what objections are we missing," "mine reddit for customer language," "voice of customer from
  forums," or pastes subreddit names / thread links and asks for insight.
---

# Reddit & Forum Listening Digest

Raw forum posts contain the exact words buyers use *before* they've heard of your product — the uncoached vocabulary that converts when you echo it back. This skill mines that language systematically, not randomly. Every run produces a ranked thread digest, a pain vocabulary table tagged by awareness stage, emerging objections for the battlecard, and a VoC snippet bank ready to paste into ads or landing pages.

The framework is **Pain Vocabulary Mining**: extract, tag, deduplicate, and rank customer language by frequency and awareness stage so you can use it in copy immediately. The digest accumulates weekly — compare runs to spot vocabulary drift before your competitors do.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's ICP, positioning, offer, and banned words. All pain vocabulary is filtered against the ICP before presenting; snippets that match a banned-word pattern are flagged, not silently dropped.
- **`voice-of-customer-mining-pipeline`** (optional) — if installed, delegates the raw extraction pass for large thread batches; this skill handles tagging, ranking, and digest formatting.
- **`objection-library-builder`** (optional) — if installed, forwards newly discovered objections from the digest for permanent capture; synthesizes inline if absent.
- **`icp-persona-builder`** (optional) — if installed, cross-checks pain themes against the persona's known JTBD map; notes gaps.
- **`competitive-intelligence-dossier`** (optional) — if a competitor is mentioned in threads, passes the mention cluster to this skill for context enrichment.

---

## How a run works

```
Step 0  Load the brand          ──► call brand-brain; get ICP + positioning + banned words
Step 1  Scope the sources       ──► subreddits / queries / URLs; confirm scope with user if vague
Step 2  Collect threads         ──► fetch/receive posts; filter by date window and minimum engagement
Step 3  Extract pain vocabulary ──► Pain Vocabulary Mining pass (see framework below)
Step 4  Tag awareness stages    ──► apply Schwartz ladder to each pain phrase cluster
Step 5  Surface emerging obj.   ──► compare to known brand objections; flag deltas
Step 6  Build snippet bank      ──► curate top 10–15 VoC quotes for copy use
Step 7  Assemble digest         ──► structured output; save to ./research/reddit-digest-[YYYY-WW].md
Step 8  Delta note (if prior)   ──► if a prior digest exists, surface vocabulary drift in 3 bullets
```

**Do not produce output before `brand-brain` returns.** The ICP filter is what separates signal from noise.

---

## Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns the active brand's digest: ICP profile, awareness tendency, positioning line, offer mechanics, real proof points, and banned words.

Use the ICP to filter thread relevance throughout (if a thread's poster profile clearly falls outside the ICP, note it but don't weight it heavily). Use banned words as a flag trigger on VoC snippets — you surface them with a warning rather than silently using them in copy.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user to install `brand-brain` (preferred) or answer a 4-question mini-setup (what it is · ICP + awareness · offer mechanics + 3 voice adjectives + banned words), then proceed.

---

## Step 1 — Scope the sources

Accept any of:
- **Subreddit list** — e.g. `r/ecommerce, r/shopify, r/smallbusiness`
- **Search queries** — e.g. "push notifications annoying site:reddit.com"
- **Thread URLs** — pasted directly
- **Topic only** — if the user gives only a topic, suggest 3–5 high-signal subreddits for confirmation before proceeding

Default date window: last 30 days. Offer to widen to 90 days on request. Minimum engagement filter: ≥5 comments (configurable).

If the scope is genuinely ambiguous, ask one clarifying question: "Which audience problem are we listening for?" — then proceed.

---

## Pain Vocabulary Mining Framework

The core intellectual framework. Four passes on the collected thread text:

### Pass 1 — Raw extraction
Pull every complaint, frustration, switch trigger, workaround, feature request, and question that maps to the brand's problem space. Do NOT filter at this stage — capture the raw language as written, including slang, portmanteaus, and colloquialisms. Preserve the original phrasing; do not rephrase into "clean" language — that destroys the copy value.

### Pass 2 — Clustering
Group extracted phrases by semantic theme (not by keyword). One theme can have 5–15 surface-level phrasings — keep all variants in the cluster because A/B tests often hinge on the exact word. Name each cluster with a plain-English label (e.g., "notification fatigue," "setup complexity," "trust / spam fear").

### Pass 3 — Awareness tagging (Schwartz ladder)
Tag each cluster with its predominant awareness stage — this tells you where to use the language in the funnel:

| Stage | What the poster knows | Where to use this language |
|---|---|---|
| Unaware | Has the pain, hasn't named it | Top-of-funnel ads, blog hooks |
| Problem-aware | Names the pain, no solution in mind | TOFU content, SEO, cold outreach subject lines |
| Solution-aware | Researching categories | Comparison pages, mid-funnel email |
| Product-aware | Has heard of you or competitors | Trial emails, demo CTAs, sales objections |
| Most-aware | Is evaluating / nearly converted | Pricing objections, risk-reverser copy |

A cluster often spans two stages — tag both and mark the dominant one.

### Pass 4 — Signal scoring
Score each cluster on three axes (1–3 each):
- **Frequency** — how many unique posters expressed this theme?
- **Intensity** — does the language carry emotional charge (frustration, relief, urgency)?
- **ICP match** — does the poster profile match the brand's ICP?

Sum the three scores (max 9) to get the cluster's signal score. Surface clusters with score ≥6 as primary findings; 4–5 as secondary; ≤3 as watch list.

---

## Digest output format

```markdown
## Reddit & Forum Listening Digest — [Brand] — Week [YYYY-WW]
**Sources:** [subreddits / queries]   **Window:** [date range]   **Threads reviewed:** [N]
**Brand / ICP filter:** [slug, via brand-brain]

---

### Top Threads (by signal value)
| # | Title | Subreddit | Score | Comments | Signal note |
|---|---|---|---|---|---|

---

### Pain Vocabulary Table
| Cluster | Raw phrases (top 3–5 variants) | Stage | Signal score | Copy-ready? |
|---|---|---|---|---|

*Copy-ready = safe to use in brand copy with no modification; [verify] if sourced from a low-ICP-match poster.*

---

### Emerging Objections
Objections NOT currently in the brand's known objection set. Format:
- **[Objection label]** — verbatim quote. Frequency: N. Awareness stage: X. Suggested counter-message: [one sentence or [verify]].

---

### VoC Snippet Bank (top 10–15)
Curated verbatim quotes, each with: source thread URL (or [paraphrased] if aggregated), poster ICP match (High / Medium / Low), awareness stage, and suggested placement (ad hook / email subject / landing page headline / testimonial pull-quote).

---

### Delta vs. Prior Digest
*(Omit on first run)*
- New themes that appeared this period: …
- Themes that spiked in intensity: …
- Themes that faded: …
```

Save to `./research/reddit-digest-[YYYY-WW].md`. On subsequent runs, load the prior digest to produce the delta note.

---

## Handling missing or paywalled data

Reddit's native feed is the primary source. If the user can paste thread text, use it directly. If the user has a third-party tool export (Brandwatch, Mention, Sprout), accept CSV or paste. Never fabricate threads or quotes — if data is thin, say so and suggest widening the window or adding subreddits. Mark any aggregated/paraphrased quote `[paraphrased]` in the snippet bank so no one pastes fake verbatims into copy.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No output before the ICP filter is loaded. Unfiltered forum data is noise.
- **Preserve original language.** The copy value is in the exact words — never rephrase into "cleaner" marketing language during extraction.
- **Real quotes or [paraphrased].** Never fabricate verbatims. Unconfirmed attribution is marked `[verify]`.
- **Stage tagging is not optional.** Unlabeled pain language gets used at the wrong funnel stage; that's why the framework tags everything.
- **Accumulate, don't reset.** Save every run to `./research/` and surface the delta — vocabulary drift is a signal, not a maintenance task.
- **ICP filter, not ICP gate.** Out-of-ICP posts can still surface a theme worth watching; keep them in the watch list, not the primary table.

## What Not to Do

- Don't fabricate threads, quotes, or engagement numbers — mark data gaps honestly.
- Don't rephrase raw quotes into "cleaner" language before presenting to the user; cleaned quotes lose their conversion power.
- Don't skip the awareness-stage tag to save time — it's what separates a word list from a usable content brief.
- Don't forward emerging objections to the battlecard without noting that they're unconfirmed forum claims, not validated sales objections.
- Don't reimplement brand ICP resolution here — that lives in `brand-brain`.
- Don't produce a generic "people are frustrated" summary — every finding must be backed by a cluster with at least one real phrase variant.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand + ICP loaded before any extraction?
- All four Pain Vocabulary Mining passes completed (extract → cluster → tag → score)?
- Every cluster has ≥1 verbatim phrase variant, an awareness stage, and a signal score?
- Emerging objections section lists only objections NOT already in the brand's known set?
- VoC snippet bank contains only real / paraphrased-and-labeled quotes, each with ICP match and suggested placement?
- Delta note produced if a prior digest exists in `./research/`?
- Digest saved to `./research/reddit-digest-[YYYY-WW].md`?
- Banned words from `brand-brain` flagged (not silently removed) where they appear in snippets?
