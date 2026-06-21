---
name: cfo-ready-budget-summary-slide-builder
description: >
  Takes a detailed marketing budget workbook (Excel/Sheets/CSV/paste) plus the ask context — owner,
  period, strategic rationale, approval tier — and produces three things in one pass: (1) an executive
  one-pager summarizing total ask, committed-vs-discretionary split, channel mix, and key assumptions;
  (2) a board-ready slide outline (title, three supporting data slides, one risk/scenario slide, one ask
  slide) with speaker notes per slide; (3) a CFO-lens critique flagging weak assumptions, missing proof
  of ROI, unanswered finance questions, and the single most likely cut. Calls brand-brain to anchor all
  messaging in the brand's real proof and positioning. Calls budget-variance-p-l-reconciliation-analyst
  when actuals are present. Calls data-qa-measurement-gotcha-checker as a data-quality gate before any
  number appears in the output. Calls board-exec-summary-writer for the exec one-pager if it's installed.
  Does NOT write the underlying financial model or produce a live spreadsheet; it shapes the story and
  the structure from whatever numbers the user brings. Use when the user says "help me present the
  budget," "build a CFO deck," "summarize marketing spend for the board," "I need a budget one-pager,"
  "executive budget summary," "board slide for the budget," "make the ask," or pastes a budget table
  and asks what to do with it.
---

# CFO-Ready Budget Summary & Slide Builder

Marketing budgets die in the spreadsheet — not because the numbers are wrong, but because they never get translated into CFO language. This skill takes your detailed budget and turns it into a board-ready narrative: a one-pager with the right splits, a slide outline that answers the questions finance will actually ask, and an explicit critique from the CFO's perspective so you can walk in prepared.

The framework is **Committed / Discretionary / Proof / Risk**. CFOs grant marketing budgets when four questions are satisfied: *What is locked-in spend we can't avoid? What is judgment-call spend and what does it buy? What is the evidence it works? What happens if it doesn't?* Every artifact this skill produces is organized around those four answers.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's positioning, proof points, and offer mechanics so financial framing and ROI claims stay grounded in real evidence, not fabricated benchmarks.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, the primary value proposition, and 2–3 real proof points (customer count, retention rate, revenue impact) before proceeding.
- **`data-qa-measurement-gotcha-checker`** (Step 1, gate) — validates the user's input numbers before they appear in any output. Flags broken attribution, currency/unit mismatches, period inconsistencies, and vanity metrics that finance will dismiss.
- **`budget-variance-p-l-reconciliation-analyst`** (Step 2, when actuals exist) — when the workbook includes prior-period actuals, delegates variance narrative and reconciliation to this sibling rather than rebuilding the logic here.
- **`board-exec-summary-writer`** (Step 4, when installed) — delegates the final one-pager write to this sibling for consistent exec-voice formatting. Synthesizes inline if absent.
- **`initiative-business-case-writer`** — call when any single budget line exceeds ~20% of total ask or when the user wants a standalone justification for a new program.
- **`analytics-report-reviewer`** — call when the user includes performance data (ROAS, CPL, CAC) to pressure-test the numbers before they go in the deck.
- **`infographic-data-viz-spec-writer`** — call when the user wants a visual channel-mix chart or spend-vs-return matrix embedded in the slide.
- **`deck-presentation-writer`** — call when the user needs full slide prose rather than the outline this skill produces by default.

---

## How a run works

```
Step 0  Load the brand        ──► brand-brain (brand voice, real proof, positioning)
Step 1  Quality-gate the data ──► data-qa-measurement-gotcha-checker (flag broken numbers)
Step 2  Parse the budget      ──► committed / discretionary split + channel roll-up
Step 3  Run the CFO lens      ──► four-question framework + anticipated cuts
Step 4  Write the artifacts   ──► one-pager + slide outline + critique
Step 5  Self-review           ──► check before presenting
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns the active brand's proof points, positioning line, offer mechanics, and ICP. Every ROI claim in the one-pager and every proof-point in the deck must trace to real evidence from this digest. Mark any number not confirmed there as `[verify]`.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, the primary value proposition, and 2–3 real proof points (customer count, retention rate, revenue impact) before proceeding.

### Step 1 — Quality-gate the data

**Invoke `data-qa-measurement-gotcha-checker`** on the user's input before touching any number. Common failures: blended CAC that mixes paid and organic, ROAS that excludes fulfillment, quarterly spend compared to annual targets, currency inconsistencies across territories. Any flagged item must be resolved or explicitly noted as `[unreconciled — verify before board presentation]` in the output.

### Step 2 — Parse the budget into the framework

Organize every line item into four buckets:

| Bucket | Definition | Board signal |
|---|---|---|
| **Committed** | Contracted, tooling, headcount-loaded, or inflight spend that cannot be cut without penalty or operational disruption | "Cost of staying in the game" |
| **Discretionary / Proven** | Channels with confirmed ROI (past-period ROAS, CPL benchmarks) that the brand plans to continue scaling | "Bets we've already validated" |
| **Discretionary / New** | New channels, experiments, or initiatives with no prior-period proof | "Bets we're making" |
| **Contingency / Reserve** | Unallocated buffer | "Optionality" |

Compute: total ask, committed %, proven-discretionary %, new-discretionary %, and the single largest line item (flag if >25% of total — CFOs will ask about it unprompted).

---

## The four-question CFO lens

Run this analysis before writing any artifact. The slide outline and critique are structured around the answers.

**Q1 — What is locked-in?** Identify every committed line. If committed spend exceeds ~60% of ask, the budget is not really a discretionary decision — frame it as a cost-maintenance ask, not a growth ask. That changes the conversation entirely.

**Q2 — What does the judgment-call spend buy, exactly?** For each discretionary line, require: metric it moves, expected delta, historical proof or named benchmark (`[verify]` if neither exists), and payback window. Vague "brand awareness" or "demand gen" with no metric attached will not survive CFO review.

**Q3 — What is the evidence?** From the brand-brain digest: real CAC, retention rates, revenue-per-channel, or customer counts. Finance does not accept `[verify]` items as proof — flag them explicitly so the user can fill them before the meeting.

**Q4 — What happens if the top discretionary bet misses?** Produce one scenario: if the single largest new-discretionary line delivers zero return, what is the revised total ask and the fallback plan? CFOs ask this; answer it before they do.

---

## Artifacts

### Artifact 1 — Executive one-pager (inline or via board-exec-summary-writer)

Structure (fits one page or one Notion doc section):

```
[Brand] Marketing Budget — [Period]                    Total ask: $X

COMMITTED SPEND         $X  (X%)   [3-line rationale]
PROVEN DISCRETIONARY    $X  (X%)   [metric + prior-period proof]
NEW DISCRETIONARY       $X  (X%)   [metric + hypothesis]
CONTINGENCY             $X  (X%)

LARGEST SINGLE LINE:    [channel/program]  $X  (X% of total)
  → ROI claim: [real proof or [verify]]
  → Payback window: [X months or [verify]]

KEY ASSUMPTIONS
  1. [assumption + source]
  2. [assumption + source]
  3. [assumption or [verify]]

RISK: If [top new bet] misses → total committed = $X; fallback = [action]

Prepared: [date]  Owner: [name or role]
```

Save to `./budget/[brand-slug]-budget-summary-[period].md` when the user confirms.

### Artifact 2 — Board slide outline

Six slides. For each: **title**, **3–5 bullet points** (the exact words that go on the slide, not descriptions), and **speaker notes** (what to say, including the anticipated finance question and the prepared answer).

```
Slide 1 — Context & Period
  Bullets: [period covered · total ask · prior-period comparison · one-line strategic rationale]
  Speaker notes: [what changed YoY and why; the ask in one sentence]

Slide 2 — Committed Spend Breakdown
  Bullets: [tool/vendor/headcount lines; why each is non-negotiable]
  Speaker notes: [cost of cutting each; contract terms if relevant]

Slide 3 — Proven Discretionary: Where We're Doubling Down
  Bullets: [channel · current CAC or ROAS · ask · expected delta]
  Speaker notes: [source of proof; comparison to industry benchmark [verify]]

Slide 4 — New Bets: What We're Testing and Why Now
  Bullets: [program name · hypothesis · success metric · kill criteria]
  Speaker notes: [why this window; who owns it; decision point]

Slide 5 — Risk & Scenarios
  Bullets: [downside scenario · revised total · trigger condition · mitigation]
  Speaker notes: [the answer to "what if this doesn't work"]

Slide 6 — The Ask
  Bullets: [total figure · approval tier · decision timeline · owner]
  Speaker notes: [what "yes" unlocks; what "no" delays]
```

### Artifact 3 — CFO-lens critique

After producing the artifacts, deliver a direct 5-point critique:

1. **Weakest assumption** — the number most likely to be challenged and why.
2. **Missing proof** — discretionary lines with no confirmed ROI (list them; they are liabilities in the room).
3. **Concentration risk** — if any single channel or program exceeds 25% of ask, name it and the exposure.
4. **Most likely cut** — the one line a CFO would target first, and what the marketer should preempt.
5. **Framing fix** — one reframe that changes the conversation (e.g., converting a cost-center frame to a revenue-driver frame with the brand's actual numbers).

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No proof claims before the digest loads. Every ROI assertion traces to a real data point or is marked `[verify]`.
- **Data-QA gate before output.** Broken attribution, unit mismatches, and period inconsistencies corrupt board credibility faster than any messaging failure — run the gate.
- **Committed / Discretionary / Proof / Risk — always.** This is the framework CFOs use mentally whether or not you use the labels. Organize around it.
- **Answer the cuts before they're asked.** The CFO-lens critique exists so the marketer walks in with the responses already loaded, not to embarrass anyone.
- **No invented benchmarks.** If you don't have a real CAC, ROAS, or payback figure, say `[verify]` and tell the user exactly where to get it.
- **One-pager discipline.** The executive one-pager must fit one page. If it doesn't, the story isn't clear yet — compress, don't expand.

## What Not to Do

- Don't build or modify the underlying financial model — this skill shapes the narrative from the numbers provided, not the numbers themselves.
- Don't paper over a `[verify]` with a plausible-sounding benchmark — CFOs check.
- Don't produce a slide outline before the data-QA gate clears — garbage in, embarrassment out.
- Don't pad the committed bucket to make the discretionary ask look smaller — finance will re-categorize it.
- Don't write the slide outline as vague descriptions ("slide about channels") — the bullets must be the actual words that go on the slide.
- Don't skip the CFO-lens critique — it is the highest-value output in the room.

## Quality Checklist (self-review before presenting)

- `brand-brain` called; all proof claims trace to the returned digest or are marked `[verify]`?
- `data-qa-measurement-gotcha-checker` run; all flagged items resolved or labeled `[unreconciled — verify before board presentation]`?
- Every budget line assigned to exactly one of the four buckets; totals reconcile to the stated ask?
- Largest single line flagged if >25% of total?
- All four CFO-lens questions answered (locked-in / buys what / evidence / downside scenario)?
- Slide outline has six slides; each has real bullet-point words, not descriptions; each has speaker notes including the anticipated finance question?
- CFO-lens critique includes all five points: weakest assumption, missing proof, concentration risk, most-likely cut, framing fix?
- One-pager fits one page; saved to `./budget/[brand-slug]-budget-summary-[period].md` on confirmation?
