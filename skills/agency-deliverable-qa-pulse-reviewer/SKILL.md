---
name: agency-deliverable-qa-pulse-reviewer
description: >
  Submitted deliverable + original brief or weekly agency update → redline pass/flag/fail
  report against requirements plus a delivery-drift pulse summary. The QA pass scores
  each requirement from the brief as Pass / Flag / Fail, surfaces every gap, brand
  violation, or missed spec, and outputs a structured redline the internal team can
  attach directly to the agency's feedback email. The pulse summary reads the delivery
  pattern over time — slipping timelines, scope creep, repeated flags — and gives the
  relationship a health score before the next invoice or renewal decision. Use when the
  user says "review what the agency sent," "QA this deliverable," "check this against
  the brief," "redline this copy/design/report," "agency check-in," "is the agency
  delivering," "should I renew this retainer," or pastes an agency update and asks
  what's missing.
---

# Agency Deliverable QA & Pulse Reviewer

Agencies and contractors rarely fail all at once — they drift. Timelines slip a day at
a time; copy misses the ICP by one persona; a required deliverable arrives as a "WIP."
By the time the relationship feels broken, months of budget have already passed through
it. This skill catches drift early: a line-by-line redline of whatever arrived against
whatever was asked for, plus a delivery-health pulse that reads the pattern before it
becomes a write-off.

Two outputs, one run:

1. **Redline Report** — every requirement from the original brief scored Pass / Flag /
   Fail with evidence, a required action, and an owner.
2. **Delivery-Drift Pulse** — the relationship's health over time: on-time rate, scope
   creep, recurring flags, and a clear Stay / Fix / Escalate verdict.

---

## Skills this calls

- **`brand-brain`** (required first) — voice, ICP, banned words, real proof. Every
  deliverable is evaluated against the brand it was built for. Never grade copy without
  this loaded.
- **`content-qa-reviewer`** — delegate deep copy quality, readability, and style-guide
  adherence checks; synthesize inline only when absent.
- **`sales-asset-reviewer`** — for any sales enablement deliverable (decks, cold emails,
  battlecards); call when the deliverable type matches.
- **`campaign-brief-builder`** — if no original brief exists, call it to reconstruct one
  from the work already done and stated goals; use the output as the QA baseline.
- **`data-qa-measurement-gotcha-checker`** — for analytics, reporting, or dashboard
  deliverables; call when the deliverable is data-heavy.
- **`landing-page-heuristic-live-cro-auditor`** — for landing page or paid-channel page
  deliverables.
- **`pre-mortem-post-mortem-generator`** — when the pulse review surfaces a serious
  delivery failure; use post-mortem format for the structured retrospective.

---

## How a run works

```
Step 0  Load the brand          ──► call brand-brain; get voice, ICP, banned words
Step 1  Resolve the inputs      ──► deliverable + brief (reconstruct if missing)
Step 2  Run the Redline Report  ──► line by line against the brief (TRACE checklist)
Step 3  Run the Pulse Summary   ──► pattern analysis over time (DRIFT score)
Step 4  Self-review + present   ──► structured report ready to attach to feedback email
```

---

## Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill before touching any deliverable copy. Use the returned
digest: voice adjectives, banned words (hard overrides), ICP, and real proof. A
deliverable that is on-spec but off-brand has still failed. Flag brand violations in
the Redline as Fail.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + that
brand's `brand.md` directly. If none exists, ask the user for voice adjectives, banned
words, and ICP before proceeding. Always prefer the Skill call.

---

## Step 1 — Resolve the inputs

Two inputs are required: **the deliverable** and **the original brief** (or equivalent
scope doc — SOW, creative brief, agency proposal, Slack confirmation thread).

If neither exists:
- Deliverable missing → tell the user; do not proceed.
- Brief missing → call `campaign-brief-builder` with the stated goals and work already
  delivered to reconstruct a working brief, then QA against that. Flag in the report
  that the QA baseline was reconstructed (not original) — this itself is a mild pulse
  flag.

When the user only pastes a weekly update (no deliverable body), run Step 3 (Pulse)
only and note that a full Redline requires the actual deliverable.

---

## Step 2 — Redline Report (TRACE checklist)

Score every requirement from the brief using the **TRACE** lens (a house checklist we
use here, not an external standard):

| Dimension | What to check |
|---|---|
| **T — Timing** | On time? Partial delivery flagged as WIP? Deadline mentioned in brief met? |
| **R — Requirements** | Every stated deliverable present? Format, length, channel specs met? |
| **A — Accuracy** | Claims, data, links, product names, pricing, dates correct? Invented proof? |
| **C — Craft** | Brand voice, ICP alignment, banned words, readability, grammar, structure? |
| **E — Extras / Scope** | Anything outside scope added without discussion? Anything in scope missing? |

**For each requirement, output one row:**

```
| # | Requirement (from brief) | TRACE dim | Status | Evidence | Required action | Owner |
```

Status values:
- **PASS** — fully met, no action needed.
- **FLAG** — present but partially met, or requires clarification; action required
  before client-facing use.
- **FAIL** — missing, incorrect, or in direct conflict with brief or brand; must be
  reworked before use.

After the table, add a **Critical Gaps** section: any Fails that block delivery or
expose legal/brand risk. These get surfaced at the top of the feedback email.

**Tone calibration for the email:** the Redline is internal-facing (honest, specific),
but the system also generates a feedback-email draft that is professional and
relationship-aware. Separate the two clearly.

---

## Step 3 — Delivery-Drift Pulse (DRIFT score, our working model)

The Pulse is a pattern read, not a one-moment grade. It requires either a log of past
deliveries (user-pasted, Notion/Asana export, or a history of prior Redline reports
saved in `./vendors/`) or the user's verbal account of the relationship.

Score five dimensions 1–5 (5 = no concern):

| Dimension | What to assess |
|---|---|
| **D — Deadline adherence** | Delivered on time historically? Consistent lateness? Pattern worsening? |
| **R — Requirement hit rate** | % of deliverables arriving complete and on-spec on first submission |
| **I — Initiative / Proactivity** | Does the agency flag issues early? Bring ideas? Or purely reactive? |
| **F — Feedback loop** | How many revision rounds per deliverable? Is it trending up? |
| **T — Trust signals** | Transparency on capacity, blockers, team changes, pricing surprises |

```
DRIFT Score: [D: x/5 | R: x/5 | I: x/5 | F: x/5 | T: x/5]
Overall: XX/25
```

**Relationship verdict:**
- 20–25 → **Stay** — high performer; prioritize.
- 13–19 → **Fix** — one structured conversation; define written SLAs.
- 8–12 → **Escalate** — formal performance review; put renewal on hold.
- ≤7 → **Exit** — begin transition planning now; document everything.

If history is sparse (first or second engagement), note that the pulse is provisional
and set a reminder to re-run at the next delivery milestone.

---

## Artifacts

Save to `./vendors/` relative to the user's working directory (create if absent):

- **`./vendors/[agency-slug]-redline-[YYYY-MM-DD].md`** — the full Redline Report +
  feedback-email draft.
- **`./vendors/[agency-slug]-pulse-[YYYY-MM-DD].md`** — the Drift score + verdict +
  recommended next conversation points.
- *(Optional)* **`./vendors/[agency-slug]-log.md`** — append a one-line entry per
  delivery event if the user wants running history for future pulse reviews.

Never overwrite a prior redline; always date-stamp. Never write inside the skill folder.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No deliverable is graded before the active brand is loaded.
  Off-brand is a Fail regardless of spec compliance.
- **Evidence, not opinion.** Every Flag or Fail cites the specific line in the brief
  and the specific line (or absence) in the deliverable. No vague "this feels off."
- **Redline is internal; feedback is diplomatic.** Separate honest internal assessment
  from what you send. The email draft lands well; the Redline tells the full truth.
- **Absence of a brief is itself a drift signal.** If no brief exists, flag it. Running
  agency relationships without written scope is where cost and quality creep begin.
- **Pattern beats moment.** One late delivery is noise. Three in a row is a pattern.
  The Pulse exists to catch the pattern before the next contract renewal decision.
- **Real proof only.** If the deliverable invents or inflates proof, that is a Fail —
  mark it, don't soften it.

---

## What Not to Do

- Don't run the Redline without `brand-brain` returning first.
- Don't produce a Pass verdict on a deliverable with brand violations, invented proof,
  or missing scope items — even if the writing is polished.
- Don't conflate the internal Redline with the feedback email; keep them as separate
  documents.
- Don't run a Pulse on a single data point and call it a verdict — note that history is
  thin and the score is provisional.
- Don't reimplement brand scanning, copy quality checks, or data validation — call
  `brand-brain`, `content-qa-reviewer`, and `data-qa-measurement-gotcha-checker`.
- Don't write inside the skill folder; artifacts go to `./vendors/` in the user's CWD.
- Don't soften a Fail to a Flag to protect the agency relationship; the Redline is for
  the internal team, and accuracy protects the buyer, not the vendor.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded before any deliverable was evaluated?
- Every TRACE dimension checked; no requirements skipped because they were awkward?
- Every Flag and Fail has specific evidence (brief line + deliverable line or absence)?
- Feedback-email draft is professional and relationship-aware, clearly separated from
  the internal Redline?
- DRIFT score covers all five dimensions; verdict matches the numeric band?
- Thin-history pulse flagged as provisional?
- Redline saved to `./vendors/[agency-slug]-redline-[YYYY-MM-DD].md`?
- Pulse saved to `./vendors/[agency-slug]-pulse-[YYYY-MM-DD].md`?
