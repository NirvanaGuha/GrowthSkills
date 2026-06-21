---
name: prioritization-framework-suite
description: >
  Takes a list of initiatives, experiments, features, or backlog items and returns a fully scored,
  ranked output using the framework the operator picks — RICE, ICE, WSJF, or a Weighted Decision
  Matrix. Runs a structured scoring session: elicits or infers the right inputs per framework,
  scores every item, surfaces the ranked list with reasoning, and flags hidden assumptions or
  measurement traps so the operator can defend the order in a room. Framework-agnostic entry point:
  if the user doesn't pick, the skill recommends the right one for the context (backlog vs. campaign
  vs. multi-criteria trade-off vs. quarterly planning). Also handles a side-by-side framework
  comparison when the stakes are high enough to warrant it. Saves the scored backlog to
  ./plans/<brand>-backlog-<YYYYMMDD>.md for re-scoring next cycle. Use when the user says
  "prioritize this list," "score my backlog," "which initiative do we do first," "RICE score these,"
  "ICE score," "WSJF," "weighted matrix," "help me rank," "what should we focus on," or pastes a
  list of ideas and asks what to tackle next.
---

# Prioritization Framework Suite

A list of good ideas is just a list. This skill turns it into a ranked order you can defend — scored against a real framework, with assumptions surfaced and trade-offs named. Pick your framework or let the skill pick for you; either way, every item gets a number and a rationale, not just a gut rank.

Scope: growth initiatives, experiment backlogs, campaign options, feature requests, content topics, channel investments, or any multi-item trade-off where sequence matters.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's ICP, offer, and strategic context so scoring inputs (Reach, Impact) are grounded in the real audience and goals, not generic assumptions.
- **`experiment-results-analyzer`** — when historical test data is available, pull actual impact lift estimates rather than guessing; avoids inflated scores on unproven bets.
- **`a-b-multivariate-test-designer`** — after ranking, the top experiment candidates can be passed here for test brief generation without re-entering context.
- **`channel-roi-scorecard`** — when scoring channel-level initiatives, actual CPA/ROAS data from this skill grounds the Impact and Effort inputs.
- **`ltv-cac-payback-calculator`** — for retention or expansion initiatives, use real payback data to anchor the Impact score instead of estimates.
- **`funnel-drop-off-analyzer`** — when the backlog is CRO-focused, funnel drop-off severity is the right Reach/Impact input; pull from here rather than guessing stage weights.
- **`growth-diagnostic-deep-dive`** — when the operator hasn't defined the growth problem yet, run this first; the ranked lever list feeds directly into the items to prioritize.
- **`experiment-pipeline-backlog-prioritizer`** — sibling skill that handles the specialized case of experiment-only backlogs with PIE scoring; hand off to it when the input is purely a test queue.

---

## How a run works

```
Step 0  Load brand context  ──► call `brand-brain` skill
Step 1  Intake the backlog  ──► list of items + any existing scores/data
Step 2  Pick the framework  ──► operator chooses or skill recommends (see below)
Step 3  Gather inputs       ──► score each item; flag gaps and assumptions
Step 4  Rank and present    ──► sorted table + rationale + red flags
Step 5  Save the output     ──► ./plans/<brand>-backlog-<YYYYMMDD>.md
```

### Step 0 — Load brand context (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`) before scoring anything. The returned digest provides ICP size and awareness stage (grounds Reach), the active offer and positioning (grounds Impact), and strategic priorities that should tilt weights in a matrix. Do not score until it returns.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly. If neither exists, ask the user for: (1) primary growth goal this cycle, (2) rough audience size for top-of-funnel, (3) one constraint (time, budget, eng capacity). Then proceed.

### Step 1 — Intake

Accept items in any format: bullets, a numbered list, a raw brain dump, a pasted CSV, or a Notion/Linear export. Normalize to a flat list of named items with any existing metadata preserved as notes. If the list has more than 20 items, offer to split into must-score-now vs. parking-lot tiers before running the full framework — scores on 30 items are not better than 12 honest ones.

### Step 2 — Framework selection

If the operator picks one, use it. If not, recommend using this decision tree:

| Context | Recommended framework |
|---|---|
| Experiment / growth backlog (owned team) | **RICE** — most rigorous; separates reach from impact |
| Fast gut-check or weekly sprint triage | **ICE** — 30 seconds per item; good enough for lean teams |
| Product or eng backlog with a delivery queue | **WSJF** — penalizes delay cost; favored by SAFe / PM orgs |
| Multi-stakeholder trade-off (budget, channels, personas) | **Weighted Decision Matrix** — surfaces implicit disagreements |
| High stakes; present framework comparison to stakeholder | Run two frameworks and show where they agree / diverge |

Offer this table to the operator if they're unsure. Never pick silently — name the recommendation and the reason.

---

## The four frameworks

### RICE  (Reach · Impact · Confidence · Effort)

**Formula:** `RICE = (Reach × Impact × Confidence) / Effort`

| Input | Definition | Guidance |
|---|---|---|
| **Reach** | Estimated users or events affected per time period (usually per quarter) | Pull from brand ICP size, GA4 segment, or a funnel stage count; mark inferred values `[est]` |
| **Impact** | Effect on the goal metric per person who reaches the experience | Score 3=massive / 2=high / 1=medium / 0.5=low / 0.25=minimal; default to 1 unless evidence says otherwise |
| **Confidence** | How sure you are that Reach and Impact are right | 100%=solid data, 80%=strong signal, 50%=gut, 20%=moonshot |
| **Effort** | Person-weeks to build and ship (or content-hours for campaigns) | Use the operator's actual unit; keep it consistent across items |

**Common trap:** inflating Impact on bets you're excited about. Confidence is the correction mechanism — use it honestly. A 20% confidence on a 3×-Impact idea still scores lower than a 100%-confidence 1×-Impact quickwin.

### ICE  (Impact · Confidence · Ease)

**Formula:** `ICE = (Impact + Confidence + Ease) / 3`  — each scored 1–10.

| Input | Definition |
|---|---|
| **Impact** | Magnitude of effect on the core metric if it works |
| **Confidence** | How sure you are it will work and that the data is sound |
| **Ease** | Inverse of effort — 10 = done in hours; 1 = months of work |

Best for teams that need to move fast or triage a large list without precise data. Warn the operator: ICE is a gut-check, not a scientific ranking. Tie-break on Ease when scores are within 1 point.

### WSJF  (Weighted Shortest Job First)

**Formula:** `WSJF = Cost of Delay / Job Size`

Cost of Delay = sum of three relative scales (1–21 Fibonacci):

| Component | What to estimate |
|---|---|
| **User/Business Value** | How much value does this deliver right now vs. later? |
| **Time Criticality** | Does value decay if we wait? (seasonal, competitor window, expiry) |
| **Risk Reduction / Opportunity Enablement** | Does doing this unlock or de-risk future work? |

**Job Size** is the relative size of the job (also 1–21 Fibonacci).

WSJF favors small, high-delay-cost items — which is often the right call in a fast-moving growth backlog. Useful when you need to justify sequence to an engineering or product partner. Remind operators that absolute numbers are meaningless; only the ratio and rank matter.

### Weighted Decision Matrix

Used when items are evaluated against multiple strategic criteria that don't all carry equal weight — channel mix decisions, vendor selection, persona bets, or OKR initiative trade-offs.

**Process:**
1. **Define criteria** (3–7): e.g. Alignment to Q-goal, Audience Fit, Effort, Risk, Speed to signal.
2. **Assign weights** that sum to 100: negotiate weights with stakeholders before scoring — this is where hidden disagreements live.
3. **Score each item** 1–5 on each criterion.
4. **Compute:** `Weighted Score = Σ (criterion weight × item score)`.
5. **Sanity-check**: if the top item surprises everyone, re-examine the weights, not the scores.

Save the weight-set to the output file so next cycle's rescore is consistent.

---

## Output format (all frameworks)

```markdown
## Prioritized Backlog — [Brand] · [Framework] · [Date]

**Goal this cycle:** [from brand-brain or operator]
**Framework:** [RICE / ICE / WSJF / Matrix]
**Scoring unit (RICE/WSJF):** [person-weeks / content-hours / story points]

| Rank | Initiative | Score | Reach | Impact | Conf | Effort | Notes / Assumptions |
|------|------------|-------|-------|--------|------|--------|---------------------|
| 1    | ...        | 48.0  | 2,000 | 2      | 80%  | 2 pw   | [est] reach from GA4 segment |
| 2    | ...        | ...   | ...   | ...    | ...  | ...    | ...  |

### Red flags / assumption audit
- [item X]: Impact score is speculative — no prior test; confidence set to 50% accordingly.
- [item Y]: Effort likely underestimated if design review is needed.

### Top 3 recommended sequence
1. **[name]** — highest RICE, strong confidence; run first.
2. **[name]** — quick win; parallelizable with #1.
3. **[name]** — higher effort but unlocks downstream items #5 and #7.

### Parking lot (scored but deprioritized this cycle)
[items below the threshold, with one-line reason]
```

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No scoring before brand context loads. ICP size and strategic goal anchor every Reach and Impact input.
- **One framework at a time.** Mixing formulae in one table is not a hybrid methodology; it is noise. If a second view is needed, show it in a separate table.
- **Confidence is the honesty lever.** Never let enthusiasm inflate Impact without reducing Confidence. A 20% confidence is not a pessimism tax — it is an accurate prior.
- **Mark inferences.** Every estimated input gets `[est]`; every unconfirmed number gets `[verify]`. Fabricated scoring inputs corrupt the entire ranking.
- **Sequence unlocks matter.** A lower-ranked item that unblocks three higher-value items must have that dependency surfaced explicitly — scores alone cannot capture it.
- **Weights before scores (Matrix).** Aligning on criteria weights before scoring reveals disagreements at the strategy level, not the item level. Do this with stakeholders.

## What Not to Do

- Don't score more than 20 items with high precision — the marginal accuracy is illusory; batch into tiers first.
- Don't mix RICE and ICE scores in the same ranked column.
- Don't invent Reach numbers from thin air; show the source or mark `[est]` with the reasoning.
- Don't skip the assumption audit — the ranking is only as good as its weakest input.
- Don't present a final rank without noting where scores are close enough that a small input change would flip the order.
- Don't reimplement experiment backlog management when `experiment-pipeline-backlog-prioritizer` already covers PIE-scored test queues.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand context loaded (ICP, goal, strategic priorities)?
- Framework chosen explicitly — operator-selected or recommended with a named reason?
- Every item scored; no blanks filled with defaults silently; inferred values marked `[est]`?
- Confidence used honestly — no item with thin evidence scored above 60% without a note?
- Red flags and assumption audit section present?
- Top-3 sequence recommendation written in plain English with reasons?
- Parking-lot items listed with deprioritization rationale?
- Output saved to `./plans/<brand>-backlog-<YYYYMMDD>.md`?
