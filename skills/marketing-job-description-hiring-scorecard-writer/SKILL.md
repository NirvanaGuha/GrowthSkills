---
name: marketing-job-description-hiring-scorecard-writer
description: >
  Turns a role brief into a complete, ready-to-use hiring kit for marketing and growth positions.
  Produces three interlocked artifacts: a publish-ready job description (Company / Role / About You /
  Compensation structure), a structured interview scorecard with competency-weighted rubric, and a
  candidate-evaluation matrix for calibrated, bias-resistant hiring decisions. Uses the Topgrading
  A-Player framework and structured-interview research to ensure every criterion maps to observable
  evidence — not vibes. The skill is brand-aware (tone/culture signals come from brand-brain) and
  composes icp-persona-builder to ensure the JD speaks to the right candidate archetype. Saves all
  three artifacts as a cohesive hiring kit so the process is repeatable across future opens on the
  same role. Use when the user says "write a job description," "JD for a marketer," "hiring scorecard,"
  "interview rubric," "how do I evaluate this candidate," "build a hiring process," "create a role
  description," or hands you a role title and asks for anything hiring-related.
---

# Marketing Job Description & Hiring Scorecard Writer

Give it a role, get a complete hiring kit. The JD attracts the right candidates; the scorecard makes the final call defensible and bias-resistant; the evaluation matrix lets any hiring team compare finalists on the same dimensions. All three travel together — one role brief in, one cohesive kit out.

This skill writes and structures. It does not run background checks, make offers, or handle ATS configuration. It flags any legal compliance risks it spots (protected-class language, salary-range laws) but defers to qualified employment counsel for binding advice.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's voice, culture signals, ICP, and positioning. The JD's tone, mission framing, and "Why us?" copy all come from this. Does not implement brand scanning or storage itself.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for company name, voice adjectives (3), mission/product description, ICP (buyer type), and any banned words or culture phrases before proceeding.
- **`icp-persona-builder`** *(optional but recommended)* — used to build a Candidate Persona (the ideal hire profile) in the same shape as a buyer persona: motivations, objections, where they spend time, what language resonates. Compose when installed; synthesize inline when absent.
- **`contractor-agency-brief-builder`** *(optional)* — if the role is contract or agency-facing rather than FTE, call this to adapt the output to scope-of-work format.
- **`feedback-recognition-framer`** *(optional)* — for interview debrief notes and calibration feedback, call this to frame observations in SBI format before logging.

---

## How a run works

```
Step 0  Load the brand          ──► call brand-brain (voice + culture + ICP)
Step 1  Intake the role brief   ──► 5 core questions if brief is thin
Step 2  Build candidate persona ──► call icp-persona-builder or synthesize inline
Step 3  Write the JD            ──► structured sections, on-brand voice
Step 4  Build the scorecard     ──► Topgrading-anchored competency rubric
Step 5  Build the eval matrix   ──► cross-candidate comparison table
Step 6  Self-review & save      ──► compliance scan, then save to ./hiring/[slug]/
```

---

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns voice adjectives, banned words, positioning, ICP, and real proof. Use these to write the JD's company framing and "Why join us" section in the brand's real voice. If brand-brain bootstraps a new brand, wait for it to complete before writing any copy.

**Fallback if brand-brain is absent or returns no brand:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for company name, voice adjectives (3), mission/product description, ICP (buyer type), and any banned words or culture phrases before proceeding.

### Step 1 — Intake the role brief

Accept anything: a Slack message, a bullet list, a previous JD, a title alone. If critical information is missing, ask only for the gaps in one batch — never more than 5 questions:

1. Role title and team (who they report to, approximate org size)
2. Top 3 outcomes the hire must deliver in the first 90 days
3. Must-have vs. nice-to-have experience (hard requirements only — state them as actions, not credentials)
4. Compensation range and work arrangement (remote/hybrid/onsite, FTE/contract) — flag if missing for salary-range-law jurisdictions
5. Any deal-breakers or disqualifiers on the interview side

### Step 2 — Candidate Persona

Before writing the JD, synthesize a one-paragraph candidate persona: who they are today (role/seniority), what problem or ambition drives them to look, what makes this role attractive vs. alternatives, and what language will resonate in the JD. Call `icp-persona-builder` if installed; otherwise, derive from the brand's ICP + the role brief. The JD copy and scorecard weights are calibrated to this persona.

---

## Artifact 1 — Job Description

**Framework: Structured Role Brief → AIDA-shaped JD**

Well-researched JDs (LinkedIn, Indeed, SHRM) convert better when they open with the mission/impact (Attention), describe who thrives here (Interest), specify concrete outcomes (Desire), and end with a frictionless apply CTA (Action). Structure accordingly.

### Section map

```
[Brand voice + mission hook — 2–3 sentences, real differentiation, no platitudes]

## The Role
One paragraph: what the hire owns, who they collaborate with, why it matters to the company's trajectory now.

## What you'll do (the first 90 days, then ongoing)
- 90-day milestones: 3 concrete, measurable outcomes
- Ongoing responsibilities: 5–7 bullet points (action-led; "Own X," "Build Y," not "Responsible for")

## What we're looking for
- Must-haves: [3–5 hard requirements, framed as demonstrated skills, NOT years of experience or degree requirements unless legally required]
- Nice-to-haves: [2–3 genuinely optional]
- What does NOT disqualify you: [de-bias signal — explicitly name 1–2 credential assumptions the company doesn't make]

## Why join us
[3–4 sentences of real pull factors from brand.md — product-market tailwinds, team caliber, growth trajectory, compensation philosophy. Mark unconfirmed claims [verify].]

## Compensation & logistics
[Range, equity if any, benefits highlights, work arrangement, hiring timeline]

## How to apply
[Single CTA: where to submit, what to include — keep it short]
```

**Voice enforcement.** Pull voice adjectives and banned words from brand-brain. If the brand is direct, cut filler ("passionate," "rockstar," "ninja"). If the brand is warm, soften imperatives. Never write "we're a family."

**Legal compliance scan.** Before finalizing, flag: protected-class language, age-biased terms ("recent grad," "digital native"), vague subjective criteria that can't be evaluated, salary-range disclosure obligations (CO, CA, NY, WA — flag if range is omitted), and any absolute-credential requirements that may disproportionately screen out protected groups. These are flags, not legal opinions.

---

## Artifact 2 — Interview Scorecard

**Framework: Topgrading A-Player Scorecard (Bradford Smart)**

The scorecard defines, before any interview, what "outstanding" looks like for each competency — anchored in specific behavioral evidence, not gut feel. Every interviewer uses the same card; calibration becomes a data comparison, not a debate.

### Scorecard structure

```
Role: [title]    Level: [IC / lead / manager / director+]    Opened: [date]

## Mission of this role
One sentence: what outcome the right hire creates in 12 months.

## Outcomes (from Step 1)
1. [90-day outcome, measurable]
2. [6-month outcome]
3. [12-month outcome]

## Competencies & weights
[see table below]

## Disqualifiers
[Any single criterion that is an immediate no, regardless of overall score]

## Scorecard summary
Overall score: [weighted average]
Hire / No-hire / More info needed
Interviewer + date
```

### Competency rubric

Select 4–6 competencies from the role brief. Calibrate weights (must sum to 100). For each, define behavioral anchors at 1 / 3 / 5.

| Competency | Weight | 1 — Below bar | 3 — Meets bar | 5 — Exceeds bar |
|---|---|---|---|---|
| [e.g. Analytical thinking] | 25% | Uses gut/opinion; can't walk through reasoning | Structures problems; picks reasonable proxy metrics | Builds models from scratch; identifies leading vs lagging signal; changes recommendation when data shifts |
| [e.g. Cross-functional execution] | 20% | Works in silos; others don't trust their timelines | Keeps stakeholders informed; mostly delivers on time | Named as the go-to by engineering/sales unprompted; drives decisions that aren't theirs to make |
| [e.g. Copywriting / messaging] | 20% | Generic voice; can't explain why a word choice works | On-brand; edits well; adapts tone by channel | Originates a voice; can critique their own work and a competitor's in the same breath |
| … | … | … | … | … |

**Standard competencies by level (apply selectively):**
- IC: craft depth, self-direction, speed of learning, written communication
- Lead/Manager: prioritization under constraint, delegation, people development, cross-functional credibility
- Director+: narrative (can tell a story to a board), organizational design, external radar (trends, competitors, talent)

**Interview question bank.** For each competency, provide 2 behavioral questions (STAR-format anchors) and 1 situational question. Keep them specific — not "tell me about a time you did X" but "tell me about the last campaign where you owned both strategy and execution — what was the result and what would you do differently."

---

## Artifact 3 — Candidate Evaluation Matrix

A simple cross-candidate table. Fill one row per finalist after scorecards are submitted. Forces calibration before the debrief meeting rather than during it.

```
| Candidate | Comp 1 (wt%) | Comp 2 (wt%) | … | Wtd Score | Disqualifier? | Hire? | Notes |
|---|---|---|---|---|---|---|---|
| [Name] | | | | | | | |
```

- Scores come from individual interviewers' scorecards — the matrix aggregates, it does not override.
- The "Hire?" column is populated only after the debrief; it is not a pre-debrief consensus shortcut.
- One optional column: "Biggest open question" — used to decide if a further conversation is worth scheduling vs. a no-hire.

---

## Persistence

Save the full kit to `./hiring/[role-slug]/`:
- `jd.md` — the job description (publish-ready markdown)
- `scorecard.md` — the competency scorecard with question bank
- `eval-matrix.md` — blank candidate table, ready to fill

Confirm in one line: *"Hiring kit saved to `./hiring/[role-slug]/` — share `jd.md` to post, share `scorecard.md` with every interviewer before the first screen."*

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No JD copy before the active brand is loaded. Voice is not optional.
- **Outcomes over credentials.** JDs specify what the hire will deliver, not what degrees or years prove they can. Credentials appear only when legally required or genuinely necessary for the role.
- **Observable evidence only.** Every scorecard criterion must be answerable with a specific story or work sample — never "seems smart" or "good culture fit."
- **Weights before interviews.** Competency weights are set before any candidate is seen. Changing them after is p-hacking for hiring.
- **Flag, don't fabricate.** Mark any unconfirmed company claim `[verify]`. Never invent proof points or compensation figures.
- **Bias-reducing by default.** Remove age-coded, credential-biased, and subjective-personality language from every JD. The "What does NOT disqualify you" section is non-optional.

## What Not to Do

- Don't write JD copy before brand-brain returns the active brand.
- Don't use "passionate," "rockstar," "ninja," "guru," "self-starter," "fast-paced environment," or "like a family" — ever.
- Don't write a scorecard after you've already reviewed resumes — that defeats the structured-interview method.
- Don't set all competency weights equal — that signals no one thought about what actually matters.
- Don't conflate the scorecard with the eval matrix — one is per-interviewer per-competency, the other is the cross-candidate synthesis.
- Don't omit the compensation range for roles in salary-disclosure jurisdictions without flagging it.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or bootstrapped) before any JD copy written?
- Candidate persona synthesized and JD framing anchored to it?
- JD: AIDA structure; outcomes in 90-day / ongoing form; must-haves stated as demonstrated skills, not years; de-bias "What does NOT disqualify you" section included; "Why join us" uses only real proof (or `[verify]`); apply CTA is one clear action?
- Legal compliance scan complete: no protected-class language, salary-range disclosure flagged if missing, no vague subjective criteria?
- Scorecard: 4–6 competencies; weights sum to 100; behavioral anchors at 1/3/5 for each; 2 STAR + 1 situational question per competency; disqualifiers listed?
- Eval matrix: blank template with all competency columns and a "Biggest open question" column?
- All three artifacts saved to `./hiring/[role-slug]/` and path confirmed to user?
