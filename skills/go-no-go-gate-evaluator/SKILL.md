---
name: go-no-go-gate-evaluator
description: >
  Structured go/no-go evaluation for any project, campaign, launch, or initiative — before
  resources are committed and momentum makes saying no feel impossible. Takes the project
  description and its gate criteria (or infers sensible defaults when none are supplied) and
  returns a per-criterion pass/fail verdict scored against a named decision framework (RICE,
  ICE, or a custom rubric), followed by a clear Proceed / Pause / Kill recommendation with the
  primary rationale. Composes with `idea-evaluation-stress-test-suite` (adversarial pressure
  test), `validity-threat-checker` (experiment-design risks), `risk-log-builder` (risk
  register for Paused items), `pre-mortem-post-mortem-generator` (pre-mortem before Go),
  `project-plan-generator-reviewer` (plan review after Go), and `tradeoff-memo-writer`
  (memo for borderline decisions). Brand context comes from `brand-brain` so gate verdicts
  stay anchored to the active brand's positioning, voice, and strategic constraints — not
  generic best practice. Use when the user says "go or no-go," "should we launch this,"
  "gate review," "green-light check," "is this ready to launch," "kill/pause criteria,"
  "decision gate," "should we proceed," or hands over an initiative and asks whether to move
  forward.
---

# Go/No-Go Gate Evaluator

Most initiatives die slowly because no one applied a hard gate at the right moment. This skill applies one — fast, opinionated, documented. It takes what you have, stress-tests it against explicit criteria, and returns a verdict you can defend to a VP or use to redirect the team before you're sunk.

It evaluates. It does not rebuild the thing being evaluated. If the project plan needs rewriting, it says so and hands off to `project-plan-generator-reviewer`. If the experiment design has validity threats, it flags them and hands off to `validity-threat-checker`. The gate evaluator's job is the verdict, the rationale, and the next action — not the repair work.

---

## Skills this calls

- **`brand-brain`** (required) — resolves the active brand's strategic context: positioning, ICP, known constraints, banned-words, proof standards. Gate verdicts reference real strategy, not generic frameworks.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no brand.md exists, ask the user for [the brand's core strategic constraints: ICP, primary growth goal, resource ceiling, and any hard no-go rules].
- **`idea-evaluation-stress-test-suite`** *(optional, call when depth is requested or the decision is high-stakes)* — adversarial pressure test and scored ranking; surfaced as "deeper eval" mode here.
- **`validity-threat-checker`** *(optional, call when the initiative is an experiment or test)* — flags SRM, novelty, instrumentation bias, and seasonality threats before Go.
- **`risk-log-builder`** *(optional, call on Pause verdicts)* — converts flagged blockers into a risk register with owner and mitigation fields.
- **`pre-mortem-post-mortem-generator`** *(optional, recommend after a Go verdict on any high-stakes launch)* — surfaces failure modes before they happen.
- **`tradeoff-memo-writer`** *(optional, call on borderline or split-criteria cases)* — structures the competing priorities into a defensible one-pager.
- **`project-plan-generator-reviewer`** *(optional, call when plan quality is a gate criterion and the plan is present)* — reviews the existing plan rather than rebuilding it.

---

## How a run works

```
Step 0  Load brand context        ──► call brand-brain; ingest constraints + positioning
Step 1  Establish the gate        ──► parse supplied criteria OR infer defaults for the initiative type
Step 2  Score each criterion      ──► Pass / Fail / Conditional per criterion; weight by framework
Step 3  Compose optional depths   ──► call sibling skills when warranted or requested
Step 4  Render verdict            ──► Proceed / Pause / Kill + primary rationale + next action
Step 5  Save (if requested)       ──► ./decisions/[slug]-gate.md
```

---

## Step 0 — Load brand context (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`) before evaluating anything. It returns the active brand's digest: strategic positioning, ICP, growth goal, known constraints, proof standards, and banned terms. Use the positioning and ICP to calibrate the "strategic fit" criterion; use resource constraints to calibrate effort and opportunity cost thresholds. Do not evaluate until it returns.

**Fallback if brand-brain is absent or returns no brand:** do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no brand.md exists, ask the user for [the brand's core strategic constraints: ICP, primary growth goal, resource ceiling, and any hard no-go rules].

---

## Step 1 — Establish the gate

### If criteria are supplied
Use them verbatim. Categorize each as **hard gate** (one Fail = Kill) or **soft gate** (weighted). If the user doesn't label them, ask in one line — or proceed with sensible defaults and note the assumption.

### If no criteria are supplied
Infer the initiative type from the brief and apply the matching default gate set:

| Initiative type | Default hard gates | Default soft gates |
|---|---|---|
| Campaign / content launch | Strategic fit · Legal/compliance clear · Audience defined | Budget approved · Creative QA'd · Tracking in place · Brief complete |
| Experiment / A/B test | Hypothesis stated · Sample size sufficient · Tracking validated | Novelty-effect window · SRM guard in place · Rollback plan |
| Product feature launch | Success metric defined · Engineering scope confirmed | Onboarding copy ready · Support brief written · Rollback plan |
| Partnership / co-marketing | Audience overlap confirmed · Brand safety clear | Deal terms signed · Asset split agreed · Attribution defined |
| Paid campaign launch | Offer approved · Landing page live · Tracking verified | Budget approved · Audience segments QA'd · Frequency cap set |

State which type was inferred and invite correction before scoring.

---

## Step 2 — Score each criterion (RICE-derived rubric)

This skill uses a **RICE-derived gate rubric** — not because RICE was designed for binary gates, but because its four dimensions (Reach, Impact, Confidence, Effort) map cleanly onto the questions a gate review must answer:

- **Reach** — does this actually touch the right audience at meaningful scale?
- **Impact** — if it works, does it move a metric that matters to the brand's current goal?
- **Confidence** — is there evidence (prior data, user research, analogues) that it will work? Or is this a hunch?
- **Effort** — is the resource ask proportionate to the expected return?

Each gate criterion is scored **Pass / Fail / Conditional**:

- **Pass** — criterion is clearly met with real evidence.
- **Fail** — criterion is not met; mark as hard or soft and carry to verdict.
- **Conditional** — criterion is partially met; state the condition that must be satisfied to flip it to Pass.

Flag any claim as `[verify]` if it is asserted but not supported by evidence in the brief.

```
### Gate scorecard — [Initiative name]
Brand: [slug, from brand-brain]
Evaluated: [date]

| # | Criterion | Type | Verdict | Notes |
|---|---|---|---|---|
| 1 | [criterion] | Hard / Soft | Pass / Fail / Conditional | [one-line note; [verify] if unconfirmed] |
...

Hard Fails: [count]
Soft Fails: [count]
Conditionals: [count]
```

---

## Step 3 — Optional depth passes

Call sibling skills when:
- The user asks for a "deeper eval" or "stress test" → call `idea-evaluation-stress-test-suite`
- The initiative is an experiment → call `validity-threat-checker`
- The verdict is Pause → offer to call `risk-log-builder` for the blocker list
- The verdict is Proceed and the stakes are high → recommend `pre-mortem-post-mortem-generator`
- The criteria produce a split or genuinely borderline result → call `tradeoff-memo-writer`

Always name what you're doing and why before invoking a sibling. Never invoke silently.

---

## Step 4 — Render the verdict

Three outcomes — and only three. No "it depends" verdicts without a stated condition that converts them.

### Proceed
All hard gates pass. Soft fails are acknowledged but do not block. State:
- The primary reason this clears (tied to the brand's current strategic goal)
- Any soft fails to monitor
- Recommended immediate next action (one sentence)
- If high-stakes: *"Recommend running a pre-mortem via `pre-mortem-post-mortem-generator` before committing budget."*

### Pause
One or more hard gates are Conditional, or soft fails aggregate into material risk. State:
- The specific blockers (not a list of everything — the actual stoppers)
- The minimum conditions to flip to Proceed
- A clear owner and timeline if known
- Offer: *"I can build a risk register for these blockers via `risk-log-builder`."*

### Kill
One or more hard gates Fail outright, or the initiative fails on strategic fit to the brand. State:
- The disqualifying criterion and why it fails (real reasoning, not procedure)
- Whether the underlying idea has merit in a different form (redirect, not just rejection)
- One sentence on what to do with the freed resources

```
## Verdict: [PROCEED / PAUSE / KILL]

**Primary rationale:** [one or two sentences; no hedge; tied to a named criterion and the brand's goal]

**Next action:** [who does what by when]

[Soft-fail watch list, if Proceed]
[Blocker list with flip conditions, if Pause]
[Redirect suggestion, if Kill]
```

---

## Step 5 — Persistence

Save the full gate review (scorecard + verdict) to `./decisions/[initiative-slug]-gate.md` when:
- The user requests it explicitly, or
- The verdict is Pause or Kill (default: save automatically and confirm in one line)

Never overwrite an existing gate file without asking. Proceed verdicts are inline-only unless saved is requested.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** Gate criteria are calibrated against the real brand's strategic constraints, not generic templates. Load it before scoring anything.
- **Three verdicts only.** Proceed, Pause, or Kill. A conditional verdict must name the specific condition and a responsible party — no open-ended "it depends."
- **Hard gates are binary.** A hard gate cannot be "mostly Pass." It passes or it doesn't.
- **Evidence over assertion.** Claimed metrics, audience sizes, and competitive differentiators are `[verify]` until supported by real data in the brief.
- **Compose, don't rebuild.** If the project plan is weak, say so and hand off to `project-plan-generator-reviewer`. If the experiment design is suspect, hand off to `validity-threat-checker`. Stay in the gate role.
- **Kill is a service, not a failure.** A Kill verdict that saves three months of misdirected effort is the highest-value output this skill produces. Say so.

---

## What Not to Do

- Don't produce a verdict before `brand-brain` returns the active brand's context.
- Don't invent gate criteria that weren't supplied and don't match the initiative type — state which defaults you applied and invite correction.
- Don't use weasel verdicts ("mostly green," "lean toward proceed"). Name the outcome.
- Don't treat a Conditional on a hard gate as a Pass — it's a Pause until the condition is met.
- Don't reimplement brand scanning, interviewing, or storage — call `brand-brain`.
- Don't build the project plan, rewrite the campaign brief, or fix the experiment — evaluate and hand off.
- Don't mark every soft criterion as a blocker to pad the output. Triage — only name the actual stoppers.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or fallback used) before scoring?
- Gate criteria sourced from the user or inferred defaults stated and confirmed?
- Every criterion scored Pass / Fail / Conditional with a one-line note; unconfirmed claims marked `[verify]`?
- Hard vs. soft gate distinction applied; hard Fail or unresolved hard Conditional → Pause or Kill, not Proceed?
- Verdict is one of exactly three options with a primary rationale tied to a named criterion and the brand's goal?
- Next action is specific (who / what / by when where known)?
- Sibling skills called or offered where warranted (depth eval, risk log, pre-mortem, tradeoff memo)?
- Pause/Kill artifacts saved to `./decisions/[slug]-gate.md` (or save offered)?
