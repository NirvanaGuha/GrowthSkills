---
name: analytical-reasoning-toolkit
description: >
  Turns any decision, goal, or proposed action into a rigorous structured analysis so a junior
  operator reasons like a senior one. Five interlocking lenses applied in order: (1) layered
  consequence map — first-, second-, and third-order effects across time horizons; (2) inverted
  failure framing — pre-mortem via prospective hindsight to surface what "dead on arrival" looks
  like before committing; (3) cost-benefit EV calculation — expected-value math on effort,
  probability of success, and downside, surfacing the real unit economics of the decision; (4)
  cognitive-bias audit — names the specific biases most likely to be distorting this decision class
  and asks the debiasing question; (5) opportunity-cost and steelman comparison — forces the
  explicit trade-off against the best rejected alternative and articulates the strongest possible
  case for a position you might be instinctively opposing. Output is a structured decision brief
  the user can share, archive, or act on directly. Use when the user says "help me think through
  this," "should I do X," "is this worth it," "am I missing something," "pressure-test this plan,"
  "devil's advocate," "steelman this," "what could go wrong," "EV calculation," "decision
  framework," "pre-mortem," or presents a strategy, initiative, or trade-off they want analyzed.
---

# Analytical Reasoning Toolkit

There is no shortage of decisions. There is a shortage of structured thinking before they are made. This skill does not give you the answer — it makes the reasoning visible, so the right answer becomes obvious (and so you can defend it later).

Five named lenses, applied in order, each answering the question the others leave on the table. Brand context informs every lens — the same decision looks different for a bootstrapped SaaS versus a funded enterprise play.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's ICP, offer, positioning, and proof so consequence maps and EV math are anchored to this business, not a generic one.
- **`pre-mortem-post-mortem-generator`** (optional, Lens 2) — call it when the inversion step warrants a full pre-mortem document rather than an inline section; pass the initiative and the failure modes surfaced here.
- **`experiment-results-analyzer`** (optional, Lens 3) — call it when the EV calculation depends on interpreting historical test data; it returns significance, uplift, and confidence intervals this skill can use as probability inputs.
- **`prioritization-framework-suite`** (optional, Lens 3) — call it when the user has a list of competing options and needs RICE/ICE/WSJF scoring before the opportunity-cost comparison in Lens 5.
- **`growth-diagnostic-deep-dive`** (optional) — call it when the decision depends on diagnosing what is actually driving or dragging the business; feed its lever list into Lens 1's consequence map.
- **`competitive-intelligence-dossier`** (optional, Lens 5) — call it when the steelman of the alternative requires understanding what a competitor is doing well; feed findings into the comparison.

---

## How a run works

```
Step 0  Load the brand              ── call brand-brain (always first)
Step 1  Frame the decision          ── clarify: decision · decision-maker · reversibility · deadline
Step 2  Apply the five lenses       ── consequence map → inversion → EV → bias audit → opp-cost/steelman
Step 3  Synthesize to a brief       ── verdict, confidence, recommended next action
Step 4  Save if complex             ── ./reports/[slug]-decision-[date].md on request or when the brief exceeds one screen
```

---

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** to load the active brand's context — ICP, offer mechanics, pricing, real proof, positioning, banned words. Every lens is calibrated to this brand. Do not proceed before it returns.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user for ICP and business model before proceeding, or recommend installing `brand-brain`.

---

### Step 1 — Frame the decision (2–4 questions, batched)

Before applying any lens, nail the frame. Ask only what is missing:

- **Decision as one sentence.** What exactly is on the table? (Not a goal — an action or choice.)
- **Reversibility.** Type 1 (hard to undo: pricing change, brand repositioning, key hire, large spend) or Type 2 (easy to undo: copy test, email sequence, landing page)?
- **Deadline and decision-maker.** When does the window close? Who has final call?
- **Data in hand.** What evidence already exists? (Avoids asking for numbers the user has already shared.)

If the user's request answers these, proceed directly. Do not over-interview.

---

## The Five Lenses

### Lens 1 — Layered Consequence Map (Systems Thinking)

Map effects in three rings, across two time horizons (short: ≤90 days, long: 6–18 months):

| Ring | Question | What to surface |
|---|---|---|
| First-order | What happens directly if we do this? | Immediate mechanics, costs, resource draw |
| Second-order | What does that enable or prevent? | Downstream decisions unlocked or foreclosed; stakeholder reactions |
| Third-order | What does *that* change about the system? | Competitive positioning shifts; customer-behavior loops; org capabilities built or eroded |

For growth decisions, always trace the full conversion chain: does this make the top of funnel bigger, the middle stickier, or the bottom more efficient? Does it compound or require constant re-investment?

Flag any third-order effect that could dominate the first-order gain.

---

### Lens 2 — Inverted Failure Framing (Prospective Hindsight / Pre-Mortem)

Assume it is 12 months from now and this decision has clearly failed. Work backward.

1. **Name the most plausible failure modes** — not all possible failures, the 3–5 most likely. For each: what assumption did it rest on that turned out wrong?
2. **Identify the leading indicators** — what would you see in weeks 2–6 that signals the failure is unfolding? (These become your kill-switch metrics.)
3. **Surface the "hidden requirement"** — what condition must be true for this to work that no one has explicitly stated? (Often the actual crux.)

Prospective hindsight, per research by Gary Klein, improves failure-mode identification more than open-ended brainstorming because it bypasses optimism bias. Be specific — "competition responded faster" is not a failure mode; "Competitor X launched a free tier within 60 days, removing our price-to-value gap" is.

If the failure framing warrants a full document (large initiative, high stakes, team review), call `pre-mortem-post-mortem-generator` and reference the output here.

---

### Lens 3 — Cost-Benefit EV Calculation

Expected value is a forcing function. It makes implicit probability assumptions explicit.

**Formula:** `EV = (P_success × Upside) − (P_failure × Downside) − Cost`

Walk through each input:
- **Upside** — what is the realistic best-case outcome, in the brand's own success metric (revenue, signups, CAC reduction, retention rate)? Use real numbers or `[verify]`.
- **Downside** — what is the concrete worst-case cost (money, time, attention, reputation, opportunity)? Do not use "low" or "medium" — assign numbers or ranges.
- **P_success** — what is the honest prior probability this works, given base rates for this decision class? Name the base rate explicitly (e.g., "email tests typically show 15–35% lift on open rate; uplift-to-conversion is less predictable [verify]"). If the user has historical test data, call `experiment-results-analyzer` for a calibrated number.
- **Cost** — total fully-loaded cost: cash, headcount-hours at a cost estimate, attention tax on the team.

Output the EV as a range, not a point estimate. Flag if the math is dominated by P_failure × Downside — that means the downside is structurally too large relative to the probable gain.

**Sensitivity check:** which single input, if wrong by 2×, changes the decision? That is the assumption to pressure-test first.

---

### Lens 4 — Cognitive Bias Audit

Name the 2–3 biases most likely distorting this specific decision class, ask the debiasing question, and give the user the concrete check.

| Decision class | High-frequency biases | Debiasing move |
|---|---|---|
| New channel / tactic | Novelty bias, sunk-cost on current channels | "What is the base rate for this channel for companies at our stage?" |
| Pricing / packaging change | Anchoring to current price, loss aversion | "What would a new entrant price this at, knowing what we know?" |
| Hire / team expansion | Overconfidence in onboarding speed, status quo bias | "What does the role need to look like in 6 months, not today?" |
| Content / SEO investment | Planning fallacy (timeline), confirmation bias on keyword intent | "How long did our last three content bets take to show ROI?" |
| Paid acquisition scale | Survivorship bias on creative winners, ROAS gaming | "Are we measuring incrementality or last-click?" |
| Product feature | Build bias, streetlight effect on loudest users | "Which non-users would this feature need to attract for the math to work?" |

Apply the most relevant row(s). If none match, reason from first principles about which cognitive shortcuts are most activated by this decision's structure (urgency, social proof, irreversibility, familiarity).

---

### Lens 5 — Opportunity-Cost & Steelman Comparison

**Opportunity cost:** What is the best thing you are NOT doing by doing this? Be specific — not "other things," but the single highest-value alternative given the same budget/attention. If a ranked alternative list exists, call `prioritization-framework-suite`; otherwise reason from the brand's current constraint (usually: top-of-funnel volume, conversion rate, or retention).

State the trade-off as: *"Choosing X means not doing Y. Y would likely produce [outcome] by [when]. The implicit bet is that X returns more per unit of attention."*

**Steelman:** Articulate the strongest possible case for the option you (or the user) are most skeptical of. A steelman is not a rebuttal — it is the best version of the opposing argument, granted every reasonable assumption it needs. If the steelman is more compelling than the current framing, say so.

---

## Decision Brief (synthesis)

After the five lenses, synthesize to:

```
## Decision Brief — [decision in one line]

Brand: [slug, via brand-brain]
Reversibility: [Type 1 / Type 2]
Decision deadline: [date or "open"]

### Verdict
[Proceed / Proceed with conditions / Pause and pressure-test / Do not proceed]
Confidence: [Low / Medium / High] — [one sentence on what would change it]

### The crux
[The single assumption this decision stands or falls on]

### Recommended next action
[One concrete step with an owner and a date — not a list of considerations]

### Kill-switch metrics
[2–3 leading indicators to watch; thresholds that trigger a reversal]
```

Save to `./reports/[brand-slug]-decision-[YYYY-MM-DD].md` if the user asks or the brief is complex enough to warrant archiving.

---

## Principles

- **Brand-brain first.** Load the active brand before any lens. Consequence maps and EV math grounded in this business are worth ten generic frameworks.
- **Five lenses, not one.** Each lens catches a different class of error; skipping one is how smart people make dumb decisions.
- **Numbers or `[verify]`.** EV math with made-up numbers is worse than no math. If a probability cannot be estimated honestly, say so and name what data would resolve it.
- **Prospective hindsight over brainstorming.** Frame failure as already having occurred; it surfaces what open-ended "what could go wrong" misses.
- **Steelman before rebuttal.** Never argue against a position you cannot articulate in its strongest form.
- **One crux.** Every decision has a single assumption it lives or dies on. Name it explicitly.

## What Not to Do

- Do not produce output before `brand-brain` returns. Generic EV math untethered from the brand's actual metrics is noise.
- Do not apply all five lenses at equal depth regardless of decision size — Type 2 decisions get lighter treatment; Type 1 decisions get the full run.
- Do not confuse activity for analysis: lists of considerations are not decisions briefs. Deliver a verdict.
- Do not invent base rates or proof points. Mark them `[verify]` and name where the user should look.
- Do not call `pre-mortem-post-mortem-generator` for every decision — only when the initiative is large enough to warrant a standalone document.
- Do not save outputs to the skill folder; use `./reports/` in the user's working directory.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded before any lens was applied?
- Decision framed as a specific action, not a goal or aspiration?
- Consequence map has all three rings and distinguishes short from long horizon?
- Failure framing names specific failure modes (not generic risks) with leading indicators and a hidden requirement?
- EV math has explicit numbers or `[verify]` — no vague "high/low" estimates?
- Sensitivity check names the single input that would flip the decision if wrong by 2×?
- Bias audit names 2–3 specific biases with debiasing questions relevant to this decision class?
- Opportunity cost is a named specific alternative, not "other things"?
- Steelman is the strongest possible case, not a weak straw man?
- Decision brief has a verdict, a crux, a next action with an owner, and kill-switch metrics?
