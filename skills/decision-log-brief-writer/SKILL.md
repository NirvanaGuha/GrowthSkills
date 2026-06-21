---
name: decision-log-brief-writer
description: >
  Turns a raw decision — a Slack thread, a meeting note, a brain-dump, a bullet — into a
  structured decision log entry, a one-page decision brief for stakeholders, and a
  Notion-ready database row. Works for any marketing or growth decision: channel prioritization,
  budget reallocation, copy direction, go/no-go, vendor selection, positioning pivot, or
  experiment-to-launch call. It does NOT make the decision for you — use `go-no-go-gate-evaluator`
  or `tradeoff-memo-writer` for the deliberation pass. This skill locks in what was decided,
  why, who owned it, what the reversibility window is, and what would trigger a revisit —
  so future you (and future teammates) aren't reverse-engineering intent from a Slack thread
  at 11 PM. Grabs the active brand's voice and terminology via `brand-brain` so log entries
  stay on-taxonomy and on-tone. Use whenever the user says "log this decision," "write up
  what we decided," "decision brief," "capture the context behind this call," "make a
  Notion row for this," or hands over a thread/note and asks for a structured record.
---

# Decision Log & Brief Writer

Capture the decision while the context is hot. The brief is for stakeholders; the log entry is for posterity; the Notion row is the one-click import. All three come from one pass.

Every unmemorable decision is a future rework. This skill forces the five things a decision log must answer: what was decided, why this and not the alternatives, who owns it, how reversible it is, and what would make you revisit it.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's terminology, ICP, and voice so log entries use canonical product/channel names and the house tone. Decision Log & Brief Writer does not implement brand scanning or storage; that lives in `brand-brain`, once.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no `brand.md` exists, ask the user for the brand name, product/channel names to use, and tone (formal vs. conversational).
- **`go-no-go-gate-evaluator`** *(optional, upstream)* — if the decision hasn't been made yet, run the gate evaluator first; then bring the verdict here to log it.
- **`tradeoff-memo-writer`** *(optional, upstream)* — deliberation and options analysis lives there; bring the memo here to distill the log entry from it.
- **`board-exec-summary-writer`** *(optional, downstream)* — when the brief needs to escalate to a board or exec deck, pass it there for format conversion.
- **`stakeholder-update-status-writer`** *(optional, downstream)* — when the decision needs a broader team announcement alongside the log entry.

---

## How a run works

```
Step 0  Load the brand     ──► call brand-brain; load terminology + voice
Step 1  Parse the input    ──► extract the five log fields from the raw input
Step 2  Clarify gaps only  ──► ask for any field that can't be inferred (batch, max 3 Qs)
Step 3  Draft all three    ──► Log Entry · Decision Brief · Notion Row
Step 4  Self-review        ──► completeness + brand-voice + reversibility check
Step 5  Present            ──► inline output; offer to save to ./decisions/
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request. It returns the active brand's terminology, voice adjectives, banned words, and ICP. Use canonical product and channel names from `brand.md` throughout — never invent abbreviations or rename things. If the brand is new, `brand-brain` bootstraps it; **produce nothing until it returns**.

Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no `brand.md` exists, ask the user for the brand name, product/channel names to use, and tone (formal vs. conversational).

### Step 1 — Parse the input

From whatever the user provides (Slack thread, bullet, paragraph, transcript snippet), extract or infer:

| Field | What to find |
|---|---|
| **Decision statement** | One sentence: what was decided. Action verb, product/channel name, date if present. |
| **Context** | The situation that forced a choice. Keep to 2–3 sentences. |
| **Alternatives considered** | What else was on the table. Even "status quo" counts. |
| **Rationale** | Why this over the alternatives. Named framework if one was used (ICE, PIE, Eisenhower, MoSCoW, cost-of-delay). |
| **Owner + stakeholders** | Who made the call; who was informed; who executes. |
| **Reversibility + horizon** | Type-1 (hard to undo) vs. Type-2 (easily reversible) — from Bezos's reversibility heuristic. If Type-2: the window to revert. |
| **Revisit trigger** | The specific metric, date, or event that would reopen the decision. Not "if things change" — a named signal. |

### Step 2 — Clarify gaps only

If any of the seven fields above can't be reasonably inferred, ask — but **batch all missing fields into one message, maximum three questions**. Do not ask for fields you can infer. Never ask about brand context (that came from Step 0).

---

## The three outputs

### Output A — Decision Log Entry

Tight, scannable. Designed for a decision log doc or wiki page. No prose padding — direct noun phrases and clean signal per line.

```
## [Decision title — action + subject, ≤ 10 words]
**Date:** YYYY-MM-DD
**Decision:** [one-sentence statement]
**Context:** [2–3 sentences — the triggering situation]
**Alternatives considered:** [bullet list — at least the status quo]
**Rationale:** [2–4 sentences — why this; named framework if applicable]
**Owner:** [name / role]   **Stakeholders:** [informed list]
**Reversibility:** Type-1 / Type-2 — [if Type-2: revert window]
**Revisit trigger:** [named metric, date, or event]
**Status:** Decided | In progress | Superseded
```

### Output B — Decision Brief (one-pager)

For stakeholders who weren't in the room. Reads in 90 seconds. Brand voice from `brand-brain` applies here — this may be shared externally or across departments.

```
# [Same title]
**TL;DR:** [One sentence — the decision and its primary rationale]

## Situation
[3–5 sentences: what changed, what the stakes were, why a decision was needed now]

## Options considered
| Option | Upside | Downside |
|---|---|---|
| [Chosen] | … | … |
| [Alt 1]  | … | … |
| [Status quo] | … | … |

## Why we chose [option]
[3–5 sentences. Name the framework or criteria used. Anchor to real numbers or `[verify]`.]

## What this means
[2–3 bullets: concrete downstream implications — what changes, what stays the same]

## Reversibility & revisit
[One sentence on Type-1 vs. Type-2. The revisit trigger, stated as a named condition.]

**Owner:** [name / role]  |  **Date:** YYYY-MM-DD  |  **Next checkpoint:** [date or trigger]
```

### Output C — Notion Database Row

CSV-style, ready to paste into Notion's CSV import or a copy-paste row in a database view. Covers the standard fields for a Notion decision log template.

```
Title | Date | Status | Decision (one line) | Owner | Reversibility | Revisit Trigger | Tags
[value] | YYYY-MM-DD | Decided | [statement] | [owner] | Type-1 / Type-2 | [trigger] | [brand-relevant tags, e.g. "channel, paid, Q3-2026"]
```

If the user has a custom Notion schema, ask for the column names once and map to them.

---

## The reversibility heuristic (Bezos's two-door model)

The single most under-logged dimension of any decision. Apply it explicitly:

- **Type-1 (one-way door):** Hard or impossible to reverse. Requires deeper deliberation before logging; flag if the input skipped that step. Examples: vendor lock-in, architectural choices, pricing model resets, public positioning pivots.
- **Type-2 (two-way door):** Reversible within a defined window. Optimize for speed; log the revert window and what would trigger using it. Examples: channel tests, copy direction, feature flag toggles, budget reallocation within a quarter.

If the input doesn't specify, infer from the context and flag your inference. Never leave reversibility blank.

---

## Revisit trigger discipline

"We'll revisit this later" is not a trigger. A trigger is:
- A specific metric crossing a threshold: "CAC > $120 for 2 consecutive months"
- A calendar date: "Q4 planning review, 2026-10-01"
- An external event: "Google's MV3 enforcement deadline passes"
- A product milestone: "Feature X reaches 20% DAU adoption"

If the raw input doesn't name a trigger, surface the most likely one given the decision context and ask the user to confirm or replace it.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No log entry before the active brand loads. Canonical terminology only.
- **Seven fields, no shortcuts.** A log entry missing reversibility or a revisit trigger is not a log entry — it's a memo.
- **Infer, then ask once.** Extract everything possible from the input before asking. Batch missing fields. Max three questions.
- **Type-1 gate.** If the input describes a Type-1 decision that skipped deliberation, flag it — don't silently log it as clean.
- **Real numbers or `[verify]`.** If the rationale cites a metric, confirm it's real or mark it.
- **Separation of concerns.** This skill logs; it does not deliberate. Route the "should we?" to `go-no-go-gate-evaluator` or `tradeoff-memo-writer` first.

## What Not to Do

- Don't produce any output before `brand-brain` returns the active brand.
- Don't make the decision yourself — log what the user tells you was decided.
- Don't invent rationale, metrics, or stakeholders not present in the input.
- Don't write vague revisit triggers ("when needed," "if things change").
- Don't skip the Type-1 vs. Type-2 classification — it's the most decision-critical field.
- Don't reimplement brand scanning or interviewing — call `brand-brain`.
- Don't combine two separate decisions into one log entry.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded before any output?
- All seven log fields populated — none blank or vague?
- Revisit trigger is a named condition, not "when needed"?
- Reversibility classified as Type-1 or Type-2 with revert window if Type-2?
- Type-1 decision with no prior deliberation flagged to the user?
- Brief reads in ≤ 90 seconds; options table has at least 2 rows + status quo?
- Notion row is pasteable without reformatting — no markdown inside cells?
- All cited metrics are real or marked `[verify]`?
- Brand terminology used throughout (no invented abbreviations, no renamed products)?
