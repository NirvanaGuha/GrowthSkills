---
name: content-decay-refresh-sweep
description: >
  Flagship orchestrator — a growth-team-in-a-box that turns a Search Console + GA4 property into a
  prioritized batch of declining posts WITH the refreshed drafts already written. From GSC + GA4
  property IDs + a decay threshold, it audits the whole content library for organic erosion, ranks
  the highest-leverage decliners, and for each flagged URL chains a brief → rewrite → on-page SEO →
  QA loop end-to-end, producing a refresh batch a buyer can ship. It does NOT manage brand context
  itself — it calls the `brand-brain` skill first to load voice, ICP, offer, and proof, then chains
  sibling skills (`seo-content-health-decay-audit`, `content-refresh-briefer`,
  `blog-post-drafting-engine`, `on-page-seo-optimizer`, `content-qa-reviewer`) so every stage's work
  is done by the specialist, not re-implemented here. Use whenever the user says "refresh my decaying
  content," "which posts are losing traffic," "content decay sweep," "find and fix declining posts,"
  "recover lost organic traffic," "do a content refresh batch," "my blog traffic is sliding — fix it,"
  or hands over GSC/GA4 access (or an export) and asks what to update and in what order. Produces a
  prioritized refresh batch with executed drafts saved to a project folder — it does not publish
  (handoff to a publisher) and it is not net-new content (that's the topic-cluster / drafting path).
---

# Content-Decay Refresh Sweep

Point this at your Search Console and GA4 and it does what a junior SEO would spend a week on: find every post that's quietly bleeding organic traffic, rank them by how much you'd recover, and for the ones worth saving, write the refresh brief, draft the rewrite, optimize the on-page SEO, and run it through QA — looping until each one passes. You get back a prioritized batch where the top posts already have finished, on-voice, QA-cleared drafts attached, plus an honest cut line for the ones not worth the effort yet.

This is an **orchestrator**, not a new skill that re-does the work. Every stage is run by the sibling skill that owns it — this skill's job is sequencing, handoffs, gates, and a single compiled deliverable. It never invents traffic numbers, never writes brand context itself, and never publishes — it stops at finished drafts and hands them to a human (or a publisher skill) to ship.

---

## Skills this calls

In pipeline order. Every one already exists — invoke it via the **Skill tool**; do not re-implement any stage.

1. **`brand-brain`** (required, first) — loads the active brand's voice, banned words, ICP, offer, proof. Brand context is read here, never authored here.
2. **`seo-content-health-decay-audit`** (required) — the decay detector. Takes GSC + GA4 inputs + threshold → ranked list of declining URLs with decline magnitude and recoverability.
3. **`content-refresh-briefer`** (per flagged URL) — turns one decaying URL + its decay signal into a concrete refresh brief: what's stale, what intent shifted, what to add/cut/restructure.
4. **`blog-post-drafting-engine`** (per flagged URL, *refresh mode*) — executes the brief against the existing post, producing the rewritten draft (not a from-scratch article).
5. **`on-page-seo-optimizer`** (per flagged URL) — title/meta/H-structure/internal-link/schema/entity pass on the rewritten draft.
6. **`content-qa-reviewer`** (per flagged URL, gate) — Approve / Revise / Reject verdict; drives the per-URL loop.

**Optional, when installed and the audit warrants:** `serp-analysis-report` (when intent has clearly shifted under a query), `keyword-research-clustering-suite` (when the post is cannibalizing or missing a cluster), `aeo-geo-llm-visibility-optimizer` (when the goal is winning AI Overview / answer-engine citations on the refresh), `cta-variant-generator` (refresh the in-post conversion ask), `data-qa-measurement-gotcha-checker` (when the GA4/GSC numbers look suspicious — see Principles). Synthesize inline only if a *required* sibling is unavailable; skip optionals silently.

---

## How a run works

```
Stage 0  Brand        ──► call brand-brain → voice/ICP/offer/proof + brand.md path
Stage 1  Scope+Inputs ──► resolve GSC + GA4 property IDs, date windows, decay threshold, batch size
Stage 2  Audit        ──► seo-content-health-decay-audit → ranked decliners + recoverability
Stage 3  Triage gate  ──► HUMAN approves the cut line (which URLs enter the batch)
Stage 4  Per-URL loop ──► briefer → drafting(refresh) → on-page-seo → qa-reviewer (loop on Revise)
Stage 5  Compile      ──► one batch folder + index; offer to save; stop at drafts (no publish)
```

### Stage 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. It returns the active brand's digest — voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning, ICP + awareness tendency — and the path to `brand.md`. If the brand is new, it bootstraps first. **Do not run the audit or write anything until it returns.** Every downstream stage receives this digest so the rewrite stays on-voice and uses only real proof.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user to install `brand-brain` (preferred) or answer a 4-question mini-setup (what it is · ICP + awareness · offer + destination · 3 voice adjectives + banned words), then proceed. Always prefer the call.

### Stage 1 — Resolve scope + inputs

Confirm before spending a single audit query:
- **Properties:** GSC site (`sc-domain:…` or URL-prefix) and GA4 property ID. If the user only has an export, accept the file.
- **Windows:** the decline comparison (default: trailing 3 months vs. the prior comparable 3 months; offer 28-day for fast-moving libraries).
- **Decay threshold:** the minimum drop that flags a URL (default: ≥20% organic clicks *or* ≥3 average-position slots lost on a meaningful query). State the threshold you used.
- **Batch size:** how many URLs to take all the way to finished drafts this run (default: top 5 by recoverability; the rest are listed but not drafted).
- **Section filter (optional):** limit to `/blog/`, a topic cluster, or a date range.

Echo the resolved scope back in one line so the run is reproducible.

### Stage 2 — Audit (decay detection)

**Invoke `seo-content-health-decay-audit`**, passing the resolved properties, windows, and threshold. It returns a **ranked list of declining URLs**, each with: decline magnitude (clicks / impressions / position delta), the top queries losing ground, a recoverability read (refreshable vs. structurally dead vs. seasonal), and a one-line cause hypothesis. **Handoff out of this stage = that ranked table.** Do not re-derive decline math here; consume what the audit returns.

If the audit's own numbers look untrustworthy (impossible spikes, self-referral, hostname spam, a known broken event), **pause and call `data-qa-measurement-gotcha-checker`** before trusting the ranking — a refresh batch built on bad data wastes the whole run.

### Stage 3 — Triage gate (human approves the cut line)

Present the ranked decliners and your proposed cut: which enter the batch (refreshable + high recoverable loss), which are **deliberately excluded** and why (structurally dead → recommend consolidate/redirect/retire instead of refresh; seasonal → wait; thin/off-strategy → don't sink effort). **The human approves the batch before any drafting starts.** This is the single most important gate — it's where a week of wasted rewriting gets prevented. Never silently auto-select; show the call and let them adjust the threshold or the count.

### Stage 4 — Per-URL refresh loop (the engine)

For each approved URL, in priority order, run the chain. **Each stage's output is the next stage's input:**

1. **Brief** — invoke `content-refresh-briefer` with `{URL, decay signal from Stage 2, brand digest}`. → a refresh brief (what's stale, intent shift, add/cut/restructure, target queries, internal-link opportunities). If intent has clearly moved, first pull a `serp-analysis-report` and feed it into the brief.
2. **Draft** — invoke `blog-post-drafting-engine` in **refresh mode** with `{existing post + refresh brief + brand digest}`. → the rewritten draft. This *revises the existing post*, preserving what still ranks; it does not start a blank page.
3. **On-page SEO** — invoke `on-page-seo-optimizer` with `{rewritten draft + target queries}`. → optimized title/meta/headings/internal links/schema/entities. (Add `aeo-geo-llm-visibility-optimizer` here when the goal is AI-Overview citation.)
4. **QA gate** — invoke `content-qa-reviewer` with `{optimized draft + refresh brief + brand digest}`. → **Approve / Revise / Reject**.
   - **Approve** → the URL's draft is done; record it.
   - **Revise** → feed the reviewer's specific notes back to the **drafting** stage (step 2) and re-run 2→4. **Loop max twice.** If still not Approved after the second loop, mark it **Needs human** with the open issues — do not ship a draft the reviewer rejects, and do not loop forever.
   - **Reject** (brief was wrong / post shouldn't be refreshed after all) → drop it from the batch, note why, and surface it back to the Stage 3 list.

Run independent URLs in parallel where the tooling allows; keep each URL's loop self-contained so one failure doesn't block the batch.

### Stage 5 — Compile the bundled deliverable

Assemble **one refresh batch** and offer to save it to a **project-relative** path (leading `./` = the user's CWD/project, never the skill folder):

```
./content-refresh-sweep/<brand-slug>-<YYYY-MM-DD>/
  README.md                     # the batch index (below)
  00-decay-audit.md             # full ranked decliners + the approved cut line + exclusions/rationale
  01-<post-slug>/
     brief.md                   # content-refresh-briefer output
     draft.md                   # final QA-approved rewrite
     on-page-seo.md             # title/meta/headings/schema/internal links
     qa.md                      # final verdict + loop history
  02-<post-slug>/ …
```

`README.md` is the at-a-glance batch: a priority-ordered table (URL · est. recoverable loss · status `Done / Needs human / Excluded` · top target query · link to the folder), the threshold + windows used, the excluded-with-reasons list, and **clear next steps** (publish order, the non-refresh actions — consolidate/redirect/retire — and which URLs still need a human). State plainly: this batch stops at finished drafts; publishing is a separate, human-owned step (hand off to a publisher skill or the user's CMS flow).

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No audit, no brief, no draft before `brand-brain` returns. Its voice + banned-words override everything downstream.
- **Orchestrate, don't re-implement.** Each stage is run by its owning sibling skill via the Skill tool. This skill sequences and gates; it does not re-derive decline math, re-write briefs, or re-run QA logic itself.
- **The human owns the cut line.** Approve which URLs enter the batch before drafting (Stage 3). The expensive mistake is refreshing the wrong posts, not refreshing them slightly imperfectly.
- **Refresh ≠ rewrite-from-scratch.** Preserve the URL, the sections still ranking, and earned links. A refresh improves an existing asset; it does not replace it with a brand-new one.
- **The QA loop is real and bounded.** Revise → loop back to drafting, max twice, then escalate to a human. Never ship a Rejected draft; never loop forever.
- **Truth discipline.** Every number is `[verify]` unless it came straight from the audit's data pull; never invent traffic figures, proof, or differentiators. Mark anything unconfirmed.
- **Trust the data only after QA-ing it.** Suspicious analytics get run through `data-qa-measurement-gotcha-checker` before they drive a ranking.
- **Stop at drafts.** This orchestrator does not publish. Finished, QA-approved drafts + a publish-order list are the deliverable.

## What Not to Do

- Don't run anything before `brand-brain` returns the active brand.
- Don't write or edit `brand.md` here (that belongs to `brand-brain`).
- Don't re-implement a stage a sibling already owns — call the sibling.
- Don't auto-select the refresh batch; surface the cut line and let the human approve it.
- Don't refresh structurally dead, off-strategy, or purely seasonal URLs — recommend consolidate / redirect / retire / wait instead.
- Don't loop the QA gate more than twice per URL; escalate to a human with the open issues.
- Don't invent decline percentages, traffic, or proof; don't blow past the brand's banned words or voice.
- Don't publish or push to a CMS — hand finished drafts to a publisher / human.
- Don't save the batch inside the skill folder — always a project-relative `./content-refresh-sweep/…` path.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded (or bootstrapped) before any other stage?
- Scope echoed back: GSC + GA4 properties, comparison windows, decay threshold, batch size?
- Decay audit run via `seo-content-health-decay-audit` (not re-derived), and its numbers sanity-checked / QA'd if suspicious?
- Human approved the Stage 3 cut line, with excluded URLs and the non-refresh recommendation for each?
- For every batched URL: briefer → drafting(refresh) → on-page-seo → qa-reviewer all run, with the handoff passed forward each step?
- Every QA verdict resolved — Approved, or Needs-human after ≤2 Revise loops; no Rejected draft shipped?
- Compiled batch saved to a project-relative folder with a README index, the audit, per-URL artifacts, and a clear publish-order + next-steps list?
- Every unverified number marked `[verify]`; voice + banned-words honored; only real proof used?
- Deliverable stops at finished drafts (no publishing) and names the human-owned next step?
