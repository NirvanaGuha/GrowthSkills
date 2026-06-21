---
name: knowledge-base-internal-faq-manager
description: >
  Turns raw Q&A threads, support tickets, swipe files, and existing KB articles into a polished,
  searchable knowledge base with canonical FAQ entries, replication SOPs, and a structured audit
  that flags stale, duplicate, or conflicting content. Works in three modes: Create (net-new KB/FAQ
  articles from messy source material), Refresh (update and gap-fill an existing KB), and Audit
  (surface stale/duplicate/contradictory entries across a live KB). Every article inherits the brand's
  voice, terminology, and banned-word list via brand-brain — no corporate jargon inserted, no
  inconsistency with live pricing or feature names. Saves output to a project-relative ./kb/ folder
  so it survives any skill update. Use when the user says "write an FAQ," "clean up our KB,"
  "turn this Slack thread into a help article," "audit our knowledge base," "document this process,"
  "we keep answering the same questions," "our onboarding docs are out of date," or hands over
  support-ticket exports, Notion dumps, or a pile of Slack threads and asks for something reusable.
---

# Knowledge Base & Internal FAQ Manager

Raw Q&A, ticket exports, and brain-dump notes are not a knowledge base. This skill turns them into one: clean canonical articles, a tight FAQ layer, replication SOPs, and an audit that finds what's stale, duplicated, or silently wrong before it costs another support hour.

Three modes, one output standard. A single operator can run all three in a session or call any one independently.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads voice, terminology, banned words, and live offer/pricing context. KB articles must use the brand's real feature names and current pricing — brand-brain is the source of truth. Do not re-derive brand context here.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no brand.md exists, ask the user for [the brand's product name, canonical feature names, current pricing tiers, and any banned/off-brand terms].
- **`product-feature-knowledge-base-curator`** — when source material contains changelog entries or release notes, delegate feature-library curation to it rather than rebuilding inline.
- **`sop-builder-reviewer`** — when the output includes a replication SOP (how to maintain the KB going forward), call this to draft the SOP section; review it inline if absent.
- **`doc-note-summarizer`** — for condensing long transcripts, Loom summaries, or dense Notion exports before structuring them into KB articles; synthesize inline when not installed.
- **`content-qa-reviewer`** — final pass on any written article before it is marked Ready; run inline if absent.
- **`voice-of-customer-mining-pipeline`** — optionally, when the source is a large support-ticket export, call it first to extract the top recurring question clusters before writing articles.

---

## How a run works

```
Step 0  Load the brand       ──► brand-brain (voice, terms, banned words, live pricing)
Step 1  Classify the mode    ──► Create | Refresh | Audit (or all three)
Step 2  Triage source material
Step 3  Do the work (mode-specific)
Step 4  Write the SOP        ──► sop-builder-reviewer (or inline)
Step 5  QA pass              ──► content-qa-reviewer (or inline)
Step 6  Save artifacts to ./kb/
```

---

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's voice adjectives, banned words, offer/pricing mechanics, canonical feature names, and ICP. Every article uses the brand's exact feature names and current pricing — never invent names or state prices not confirmed by brand-brain.

### Step 1 — Classify the mode

| User gives you | Mode |
|---|---|
| Slack threads, ticket exports, swipe files, brain dumps | **Create** — net-new articles |
| Existing KB export + new info to fold in | **Refresh** — update + gap-fill |
| Existing KB export, no new additions | **Audit** — flag stale/duplicate/wrong |
| Mix | Run all three in sequence |

When unclear, ask: "Is there an existing KB I should audit first, or should I build from scratch?"

---

## Mode A — Create (net-new articles)

Uses the **Question Cluster → Canonical Answer → Article** framework (adapted from Basecamp's internal help-article methodology [verify exact origin]).

**1. Triage and cluster the source material.**
Read all inputs. Group by question intent, not by source. A single cluster may contain a Slack message, two ticket subjects, and a line from a sales FAQ — all answering the same underlying question. Name each cluster with a user-voiced question ("How do I add a second domain?"), not an internal label ("Domain setup").

**2. Rank clusters by ticket/thread frequency.**
More appearances = higher publish priority. State the count per cluster. If you can't count from the source material, mark priority as `[estimate — verify against ticket data]`.

**3. Draft each article.**
For each cluster, produce one canonical KB article using the structure below. Call `product-feature-knowledge-base-curator` for any cluster centered on a product feature; handle others inline.

**Article structure (Divio Documentation System — Reference layer):**
```
## [User-voiced question as H2 title]
**Answer** (1–3 sentences: direct, scannable, no preamble)
**Details** (when the short answer needs context — steps, exceptions, limits)
**Related articles** (2–3 cross-links by slug; use [link] placeholder if slug unknown)
**Last verified:** [date or YYYY-MM]
```

**4. Voice + accuracy rules.**
- Use the brand's canonical feature/tier names from brand-brain, every time.
- Mark any claim you can't confirm from source material or brand-brain as `[verify]`.
- No banned words; no corporate hedges ("please note that," "it is important to," "kindly").
- Prices and limits only from brand-brain or source material confirmed as current.

**5. Write the replication SOP.**
After articles are drafted, call `sop-builder-reviewer` for a short SOP on how to maintain this KB: where articles live, who owns updates, how often to audit, the trigger for an immediate update (pricing change, feature deprecation). Synthesize inline if absent.

**6. Save.**
Write articles to `./kb/articles/[slug].md` (slug = kebab-case of the article title). Write a `./kb/index.md` listing all articles with priority rank, last-verified date, and owner field. Write the SOP to `./kb/SOP-kb-maintenance.md`.

---

## Mode B — Refresh (update + gap-fill an existing KB)

**1. Diff against the brand-brain.**
Load the existing KB articles. For each article, check: do the feature names, pricing, and limits match the current brand-brain? Flag every mismatch as a `STALE` item.

**2. Apply new source material.**
Treat new tickets/threads/notes as in Create mode — cluster them, check if a cluster maps to an existing article (→ update it) or is net-new (→ write a new article).

**3. Produce a diff report.**
```
UPDATED: [article slug] — [what changed, one line]
NEW: [article slug] — [what it covers]
FLAGGED: [article slug] — [stale claim, needs owner review]
NO CHANGE: [count] articles
```

**4. Apply updates in place.**
Write the updated article files. Bump `Last verified` on every touched article. Do not silently overwrite user-confirmed content — flag conflicts for review.

---

## Mode C — Audit (no new additions)

Uses the **Dead Reckoning Audit** framework: assume everything drifted, prove what's still accurate, flag the rest.

**Audit dimensions:**

| Flag | Trigger |
|---|---|
| `STALE` | Feature name, price, or limit contradicts brand-brain |
| `DUPLICATE` | Two articles answer the same user question (cite both slugs) |
| `ORPHAN` | Article has no internal cross-links and low/zero search volume signal (mark for retire) |
| `CONTRADICTION` | Two articles give conflicting answers (must resolve before next update) |
| `GAP` | High-frequency ticket cluster has no article |

**Deliver:**
1. Audit table: slug | flag | one-line reason | recommended action (update / merge / retire / new article needed) | suggested owner.
2. Priority queue: top 5 items to fix in the next sprint, ranked by user impact.
3. Health score: X of Y articles pass audit (no flags). Track over time.

Save to `./kb/audit-[YYYY-MM-DD].md`.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No article published until brand-brain returns. Feature names and pricing from brand-brain only.
- **User-voiced questions, not internal labels.** Titles are the question the user actually types, not the team's internal category name.
- **One answer per cluster.** Duplication is a support cost. One canonical article, cross-linked — not two half-right ones.
- **Last-verified dating on every article.** An undated KB is an untrustworthy KB.
- **Mark what you can't confirm.** Unverified claims get `[verify]`; never invent limits or pricing.
- **Replication SOP is a deliverable, not a footnote.** A KB with no maintenance plan decays immediately.

---

## What Not to Do

- Don't write articles before brand-brain returns current feature names and pricing.
- Don't use internal team labels as article titles — always reframe as the user's question.
- Don't merge brand-brain data across brands; if the user runs a multi-brand operation, scope every article to one brand slug.
- Don't leave `Last verified` empty — if you can't date it, write `[verify — set date on review]`.
- Don't produce articles longer than ~400 words unless the topic genuinely requires it; if an article is long, it probably covers two questions (split it).
- Don't skip the audit dimension when doing a Refresh — missed contradictions cost support hours.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called; active brand loaded; all feature names and pricing confirmed against brand-brain?
- Source material triaged into named question clusters with frequency/priority noted?
- Every article uses user-voiced H2 title, direct answer first, `Last verified` date, and cross-links?
- No banned words, no invented prices or limits; all unconfirmed claims marked `[verify]`?
- Replication SOP written and saved?
- Audit (Modes B + C): diff report or audit table complete; stale/duplicate/contradiction flags all addressed or escalated?
- All artifacts saved to `./kb/` with slug-named files and an updated `./kb/index.md`?
