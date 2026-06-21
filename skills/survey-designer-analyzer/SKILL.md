---
name: survey-designer-analyzer
description: >
  Designs and analyzes research surveys for growth and content marketers. Two modes: Design Mode
  takes a research goal and produces a complete survey instrument — question types, sequencing,
  branching logic, response scales, and a pre-flight quality critique — ready to paste into
  Typeform, Google Forms, Qualtrics, or any survey tool. Analysis Mode takes a CSV or pasted
  results and produces a narrative findings report with frequency tables, cross-tab highlights,
  open-text theme extraction, and quality-of-data flags (satisficing, acquiescence, low n). Both
  modes are anchored to the active brand's ICP via brand-brain so question framing, language, and
  insight synthesis reflect who actually answered. Use when the user says "design a survey,"
  "write survey questions," "analyze survey results," "what do my survey responses say," "I ran
  a survey and want insights," "NPS follow-up survey," "customer research survey," or hands over
  a CSV of responses asking for a report. Does not run NPS/CSAT scoring loops (see
  nps-csat-feedback-loop-designer) or conduct live interview synthesis (see jtbd-customer-interview-suite).
---

# Survey Designer & Analyzer

Research questions that are poorly framed produce answers that are poorly trusted. This skill applies the **Total Survey Error (TSE) framework** — the practitioner standard for controlling specification, measurement, processing, and nonresponse error — so both the instrument you design and the analysis you produce are defensible to a skeptical stakeholder.

Two modes. One standard: every survey is built from a stated research goal, and every analysis is evaluated against whether the data actually answers that goal.

---

## Skills this calls

- **`brand-brain`** (required) — loads active brand's ICP, voice, and proof before designing questions or framing findings. Ensures language matches how the target segment thinks and speaks.
- *(optional, when installed)* `icp-persona-builder` — pull persona-level awareness stages to calibrate question depth and assumed vocabulary.
- *(optional, when installed)* `jtbd-customer-interview-suite` — if qualitative interview data already exists, use it to seed hypotheses before writing survey items; avoid re-asking what transcripts already answered.
- *(optional, when installed)* `voice-of-customer-mining-pipeline` — seed open-text question stems from real language patterns rather than internal jargon.
- *(optional, when installed)* `nps-csat-feedback-loop-designer` — hand off NPS/CSAT segment routing after analysis; don't rebuild it here.

---

## How a run works

```
Step 0  Load brand context  ──► call brand-brain; get ICP, voice, banned words
Step 1  Identify mode       ──► Design (goal → instrument) | Analysis (data → findings)
Step 2  Gate check          ──► TSE gate: is the research goal answerable by survey?
Step 3  Do the work
Step 4  Quality pass        ──► self-review against the checklist before presenting
```

### Step 0 — Load brand context (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns the active brand's ICP (role, awareness level, vocabulary tendencies), voice adjectives, and banned words. Use the ICP to set the respondent framing and language register for every question. Banned words apply to question stems and options — if the brand never uses "leverage," neither does the survey.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + `brand.md`; if none exists, ask the user for ICP role and one sentence on their awareness level, then proceed.

---

## Mode A — Design Mode

Triggered by: "design a survey," "write questions," "I want to run a survey," "build a questionnaire," or any research goal without attached data.

### Phase 1 — TSE Gate (before writing a single question)

Apply the **specification error test**: can a survey actually answer this goal, or is the goal better served by analytics, interviews, or behavioral data? If the goal conflates multiple populations, lacks a clear decision it should inform, or would require >15 minutes to answer honestly, flag it and propose a scoped alternative before proceeding.

State the gate outcome in one line: *"Research goal is survey-appropriate / Here's how I'd scope it instead: …"*

### Phase 2 — Instrument architecture (the Dillman principles)

Dillman's Tailored Design Method (the TSE-grounded standard for self-administered surveys) drives the instrument structure:

| Principle | Application |
|---|---|
| **Funnel order** | Begin broad/easy, move to sensitive/specific; never open with demographic questions |
| **Question simplicity** | One construct per question; no double-barreled items ("How useful and clear was X?") |
| **Response scale symmetry** | Likert scales must be symmetric around a neutral midpoint; 5-point for attitude, 7-point for fine gradations |
| **Acquiescence control** | Reverse-code at least 1 in 5 attitude items; flag pairs that measure the same construct |
| **Branching discipline** | Skip logic only when genuinely needed; never branch more than 2 levels deep without a survey tool that handles it gracefully |
| **Completion ceiling** | 10 items max for ad-hoc; 20 items max for considered research; >20 requires explicit stakeholder approval |

### Phase 3 — Question-type selection

| Goal | Best type | Pitfall to avoid |
|---|---|---|
| Measure satisfaction/agreement | 5-pt Likert | Don't stack >4 matrix rows — respondents satisfice (pick same column throughout) |
| Rank priorities | Forced-rank (drag or 1st/2nd/3rd) | Don't offer >5 options; ranking >5 is cognitively unreliable |
| Understand vocabulary | Open-text | Place after closed items; never open with open-text |
| Segment respondents | Multiple choice (single-select) | Ensure exhaustive + mutually exclusive options; include "Other (specify)" if uncertain |
| Measure frequency | Behavioral frequency scale (Never → Daily) | Anchor to a specific time window ("In the last 30 days") |
| Capture NPS/CSAT | 0–10 NPS or 1–5 CSAT | Add a required follow-up open-text: "What's the main reason for your score?" |

### Phase 4 — Deliver the instrument

Output format:

```
## Survey: [Title]
Research goal: [one sentence]
Respondent: [ICP role, via brand-brain]
Estimated completion time: X min
Tool assumption: [Typeform / Google Forms / Qualtrics / Generic]

---
### Section 1 — [Label]
Q1. [Question stem]
   Type: [Single-select / Multi-select / Likert-5 / Open-text / etc.]
   Options: [list, or "open"]
   Branch: [if X → skip to Q_, else continue]
   Rationale: [one line — what this answers and why this type]

[continue for all questions]

---
### Pre-flight critique
| Issue | Affected items | Fix |
```

Include a `Pre-flight critique` table before handing off: flag double-barreled items, leading language, order effects, missing "none of the above," and acquiescence risk. Fix what you can inline; flag the rest for the user.

Save to `./research/surveys/[slug]-survey.md` when working in a project context; inline otherwise.

---

## Mode B — Analysis Mode

Triggered by: pasted results, CSV attachment, "analyze my survey," "what do responses say," or any data input with question headers.

### Phase 1 — Data quality audit (before any interpretation)

Run a **measurement + nonresponse error screen** before computing a single frequency. Report:

| Check | Flag threshold | Why it matters |
|---|---|---|
| **Straightlining** | >15% of matrix respondents pick same column every row | Satisficing — those rows are unreliable |
| **Speeder responses** | Completion time < 1/3 of median | Likely non-attentive; weight or flag |
| **Low n warning** | Subgroup n < 30 | Percentages on small n mislead; report as counts |
| **Open-text length** | >30% responses under 5 words | May indicate low engagement or tool truncation |
| **Missing data pattern** | >20% skip on any single item | Item may be confusing, sensitive, or mistargeted |

Report the audit in a table; flag which questions should be interpreted with caution before moving to findings.

### Phase 2 — Quantitative synthesis (frequency + cross-tabs)

For each closed question:
1. **Frequency table** — response counts + percentages (top-box and bottom-box for Likert items).
2. **Cross-tab highlights** — break by the most decision-relevant dimension available (segment, persona, tenure, plan tier). Only report differences that are >10 percentage points AND the smaller cell has n ≥ 30.
3. **Trend note** — if prior wave data exists, note directional change without fabricating statistical significance unless n supports a chi-square.

Use the **STARR narrative frame** for each finding cluster:
- **Signal** — what the numbers show
- **Tension** — where subgroups diverge or results contradict expectations
- **Assumption risk** — what the data cannot tell you (nonresponse bias, self-report limits)
- **Recommendation** — one clear action or follow-up hypothesis
- **Replication flag** — mark findings as Confirmed / Suggestive / Directional based on n and consistency

### Phase 3 — Open-text theme extraction

Use inductive coding in three passes:
1. **First read** — scan all responses; note recurring words and emotional tone without categories yet.
2. **Code** — assign each response 1–3 descriptive codes (not pre-loaded categories; let the data speak).
3. **Cluster** — group codes into 3–7 themes; quantify the n and % of responses per theme.

Present as:

```
### Open-text themes — Q[#]: "[question stem]"
Total responses: N | Avg length: X words

| Theme | n | % | Representative quote |
|---|---|---|---|
| [Theme 1] | | | "…exact respondent language…" |
```

Never paraphrase quotes to make them more articulate; use verbatim. Mark `[edited for length]` only when truncating, never for clarity edits.

### Phase 4 — Deliver the findings report

```
## Survey Findings: [Title]
Research goal: [restated]
Respondent n: [total] | Filtered n (post quality audit): [n]
Data quality flags: [list or "none"]

### Key findings
[3–5 bullets, each one a STARR-framed insight]

### Detailed findings
[per-question or per-theme breakdown]

### What this data cannot tell you
[nonresponse bias, self-selection, timing artifacts]

### Recommended next steps
[max 3 — specific, not generic]
```

Save to `./research/surveys/[slug]-findings.md` when in a project context.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No survey question is written and no findings framed without the ICP loaded from `brand-brain`. Question language must match how the respondent segment thinks, not how the brand talks to itself.
- **Research goal is the gate.** Every question must trace back to a decision the findings will inform. Questions that don't serve the goal are cut, not kept for "interest."
- **TSE over intuition.** Measurement error (bad questions) and nonresponse error (who didn't answer) are more dangerous than sampling error. Fix the instrument before the field; audit the data before the analysis.
- **Counts before percentages.** Never report a percentage without its n. Never report a subgroup percentage when n < 30.
- **Verbatim quotes, not polished summaries.** Respondents' words are the evidence. Your job is to code and cluster, not to improve their prose.
- **One question = one construct.** Double-barreled items are corrected inline, never left for the client to sort out.
- **Honest caveats over false confidence.** "Directional" is a legitimate finding level. Overstating certainty from a 50-person opt-in panel corrupts downstream decisions.

---

## What Not to Do

- Don't write questions before `brand-brain` returns the ICP. Survey language calibrated to the wrong persona produces biased data.
- Don't design past the completion ceiling (20 items) without flagging; don't design past 25 items without explicit pushback.
- Don't compute percentages on subgroups smaller than n = 30 — report as counts and note the limitation.
- Don't paraphrase open-text quotes — verbatim is the standard.
- Don't report cross-tab differences smaller than 10 percentage points as meaningful unless a prior hypothesis named that specific split.
- Don't skip the pre-flight critique in Design Mode or the data quality audit in Analysis Mode — these are the reason the output is trustworthy.
- Don't rebuild NPS/CSAT routing logic — call `nps-csat-feedback-loop-designer` after analysis.
- Don't rebuild qualitative interview synthesis — call `jtbd-customer-interview-suite` for transcript-heavy work.

---

## Quality Checklist (self-review before presenting)

**Design Mode**
- [ ] `brand-brain` called; ICP loaded; question language matches respondent vocabulary?
- [ ] TSE gate passed (or scope reframed)?
- [ ] Every question maps to the research goal; no orphan items?
- [ ] No double-barreled, leading, or jargon-heavy items?
- [ ] Branching depth ≤ 2 levels; completion estimate stated?
- [ ] Acquiescence risk addressed (at least 1 reverse-coded item in attitude blocks)?
- [ ] Pre-flight critique table included with actionable fixes?

**Analysis Mode**
- [ ] Data quality audit completed and flagged before any interpretation?
- [ ] No percentage reported without its n; subgroups < 30 reported as counts?
- [ ] Cross-tab differences ≥ 10pp AND n ≥ 30 before reporting?
- [ ] STARR frame applied: Signal, Tension, Assumption risk, Recommendation, Replication flag?
- [ ] Open-text quotes verbatim; themes quantified with n + %?
- [ ] "What this data cannot tell you" section present?
- [ ] Findings saved to `./research/surveys/` in project context?
