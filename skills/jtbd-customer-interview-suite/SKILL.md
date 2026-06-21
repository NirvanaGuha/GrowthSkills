---
name: jtbd-customer-interview-suite
description: >
  Turns raw customer interview transcripts and research goals into structured, actionable insight —
  a JTBD map with force-tagged jobs, a tailored discussion guide for future interviews, and a curated
  quote bank organised by job, force, and funnel stage. Applies Clayton Christensen's Jobs-to-be-Done
  theory (the four forces / switch interview model) as its primary framework, with optional Outcome-
  Driven Innovation layering. Calls brand-brain for ICP + voice context so every output is calibrated
  to the actual customer profile — not a textbook generic. Works in three modes: Analyse (transcripts
  in → JTBD map + quote bank), Guide (research goals in → tailored discussion guide), or Full (both in
  one pass). Use when you have interview transcripts to decode, need to run customer discovery, want to
  extract messaging from VOC data, are building a persona or ICP, or need to uncover switch-moment
  triggers and job pains. Trigger phrases: "analyse interview," "JTBD map," "customer jobs," "switch
  interview," "build discussion guide," "VOC analysis," "pull quotes from interviews," "what job are
  we hired for," "customer insight," "jobs to be done."
---

# JTBD & Customer Interview Suite

Raw transcripts in, structured growth insight out. This skill decodes what customers are actually hiring your product to do — the functional job, the emotional job, the social job, and the forces that drove the switch — then turns that into a JTBD map, a quote bank, and a discussion guide ready for the next round of interviews.

The framework is rigorous, not decorative. Every finding is tagged to a force, a job layer, and a funnel implication. Every quote is exact and attributed. Every question in the guide is designed to surface the switch moment, not confirm what you already believe.

---

## Skills this calls

- **`brand-brain`** (required first) — loads the active brand's ICP, awareness tendency, and voice; gates every output to the real customer profile. Do not produce analysis or a guide before brand-brain returns.
- **`icp-persona-builder`** — call when no persona exists yet or when findings reveal an ICP mismatch worth documenting properly.
- **`voice-of-customer-mining-pipeline`** — call when the source material is reviews, support tickets, or community posts rather than interview transcripts; it pre-processes that data before this skill maps jobs.
- **`win-loss-interview-synthesizer`** — call after this skill completes if the interviews are specifically win/loss sales calls; that skill extends the JTBD map with deal-stage framing.
- **`positioning-messaging-architect`** — downstream beneficiary: hand it the completed JTBD map and quote bank to power a positioning rewrite.
- **`content-brief-builder`** — downstream beneficiary: the job pain language and quote bank feed directly into content briefs that match the customer's actual vocabulary.

---

## How a run works

```
Step 0  Load the brand      ──► call brand-brain (ICP, awareness tendency, voice)
Step 1  Detect the mode     ──► Analyse | Guide | Full
Step 2  Orient to the job   ──► identify the primary functional job from the inputs
Step 3  Do the work         ──► apply the switch interview model + ODI scoring (Analyse / Full)
                                 build a calibrated discussion guide (Guide / Full)
Step 4  Deliver the output  ──► JTBD map, quote bank, guide — all in brand voice
Step 5  Flag downstream uses ──► name which sibling skills benefit from the output
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns ICP (role, company type, awareness stage), voice adjectives, banned words, and proof points. Use the ICP as the lens for all analysis — the "customer" is the person described here, not a generic buyer. If no brand brain exists, brand-brain bootstraps it before returning.

**Fallback if brand-brain is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`. If neither exists, ask for: who the customer is, what they were doing before the product, and what success looks like — then proceed. Prefer the skill call.

---

## Mode detection

| Trigger | Mode |
|---|---|
| Transcripts provided (text, file path, or paste) | **Analyse** (default when transcripts present) |
| "build a guide," "discussion guide," "interview questions," research goals only | **Guide** |
| Both transcripts and a request to build a guide, OR explicit "full" | **Full** |

When unclear, ask one question: "Do you have transcripts to analyse, interview questions to build, or both?"

---

## Analyse mode — transcript → JTBD map + quote bank

### The Four-Forces Switch Model (Christensen)

Every switch decision — whether a new purchase, a churn, an upgrade, or a no-decision — is shaped by four forces. Tag every insight to a force:

| Force | Direction | What it sounds like |
|---|---|---|
| **Push** (of the situation) | Away from old | "I was frustrated that…" / "It kept breaking…" / "My old tool couldn't…" |
| **Pull** (of the new solution) | Toward new | "I heard you could…" / "The thing that attracted me was…" |
| **Anxiety** (about the new) | Resists switch | "I was worried about…" / "What if it doesn't…" / "I wasn't sure how…" |
| **Habit** (of the old) | Resists switch | "We'd always just…" / "The team was used to…" / "Switching felt like…" |

A strong switch needs Push + Pull to outweigh Anxiety + Habit. Weak adoption is almost always Anxiety or Habit winning.

### Three layers of the job

Every functional job has two underlying layers. Tag all three:

- **Functional job** — the practical task: "send targeted notifications without a developer."
- **Emotional job** — how they want to feel: "confident my campaigns are working without staring at dashboards."
- **Social job** — how they want to be perceived: "look like a data-driven marketer to my CMO."

### Processing transcripts — step by step

1. **Read all transcripts first** before extracting anything. Form a first impression of the primary job.
2. **Extract switch moments** — find the exact trigger event (the "first thought" / "what made you start looking"). This is the job's context of use: the situation, not the person.
3. **Map the four forces** across the full transcript. Pull exact quotes (no paraphrase), tag each to a force and a job layer.
4. **Score job importance and satisfaction** (ODI layer, optional but recommended for 5+ transcripts):
   - Ask implicitly from the transcript: how important was solving this? how satisfied were they before?
   - Jobs that score high importance + low prior satisfaction = **opportunity scores** worth naming.
5. **Identify competing solutions** — what were they using or doing before? What did they consider but reject?
6. **Note language patterns** — exact words they use for the pain, the outcome, the product. This is the copy vocabulary.

### JTBD map output format

```markdown
## JTBD Map — [Brand / Product name]
*Based on: [N] interviews · [Date range] · [Customer segment from brand-brain ICP]*

### Primary job
> "[Exact quote that best captures the job — verbatim, attributed to Interview N]"

**When** [situation / trigger context]
**I want to** [functional job — verb + outcome, their words where possible]
**So I can** [emotional or social job]

### Forces breakdown

| Force | Key finding | Representative quote (Interview #, verbatim) |
|---|---|---|
| Push | | |
| Pull | | |
| Anxiety | | |
| Habit | | |

### Job layers
- **Functional:** [one sentence]
- **Emotional:** [one sentence]
- **Social:** [one sentence]

### Competing solutions considered
[Bulleted list with brief note on why each was hired or rejected]

### Opportunity scores (if 5+ transcripts)
| Job / outcome statement | Importance | Satisfaction | Score* |
|---|---|---|---|
*Score = Importance + max(0, Importance − Satisfaction). Jobs scoring 10+ are high opportunity.*

### Vocabulary bank (for copy)
Pain words: [comma-separated exact customer phrases]
Outcome words: [comma-separated]
Anxiety words: [comma-separated]
```

### Quote bank output format

Append after the JTBD map:

```markdown
## Quote Bank
*All quotes verbatim. No paraphrase. [verify] if transcript source is unclear.*

### By force
**Push**
- "[Quote]" — Interview N, [role/segment if known]

**Pull**
- "[Quote]" — Interview N

**Anxiety**
- "[Quote]" — Interview N

**Habit**
- "[Quote]" — Interview N

### By funnel stage (downstream use)
**TOFU (awareness / pain recognition)**
- "[Quote that names the problem in the customer's words]"

**MOFU (evaluation / solution consideration)**
- "[Quote about evaluating options or desired outcome]"

**BOFU (decision / switch trigger)**
- "[Quote about what made them choose / almost not choose]"
```

Save to `./jtbd/[brand-slug]-jtbd-map.md` (append if file exists, do not overwrite prior interviews).

---

## Guide mode — research goals → discussion guide

A discussion guide built for switch interviews, not satisfaction surveys. The goal is to uncover the job and forces — not to validate existing assumptions.

### Guide structure (the Bob Moesta switch-interview arc)

1. **First thought** — "When did you first think about [the job / the change]?" (surfaces the push, the trigger moment)
2. **Timeline reconstruction** — "Walk me through what happened from that first thought to the decision." (surfaces forces in sequence)
3. **The moment of commitment** — "What made you decide [the switch] right then?" (isolates the deciding push/pull)
4. **The shopping episode** — "How did you look for options? What else did you consider?" (maps competing solutions and anxiety)
5. **First use** — "What did you do first? What did you expect?" (reveals the hired outcome and emotional job)
6. **Progress check** — "What does success look like for you 3 months from now?" (surfaces the social + emotional job)

### Calibrating the guide to the brand

Use the brand-brain ICP and awareness tendency to:
- **Frame the opening** — match the customer's vocabulary, not product jargon.
- **Adjust depth** — enterprise buyers have longer switch timelines (probe the internal approval / anxiety phase); SMB buyers often switch impulsively (probe the trigger moment more).
- **Pre-fill known context** — if brand-brain surfaces common push signals or anxieties, include probing questions specifically for those.

### Guide output format

```markdown
## Discussion Guide — [Brand] Customer Interviews
*Goal: [stated research goal] · Segment: [ICP from brand-brain] · Est. duration: 45–60 min*

### Before you start
- This is a listening interview, not a product demo. You speak ~20% of the time.
- Follow the energy: if they linger on a moment, go deeper. Don't rush to the next question.
- Every answer wants a follow-up: "Tell me more about that." / "What did that feel like?" / "What happened next?"

### Opening (5 min)
[2–3 rapport questions grounded in the customer's known context from brand-brain ICP]

### Core arc — switch interview (30–35 min)
**Block 1: First thought**
Q: [tailored first-thought question]
Probe: "What specifically made you think about it at that moment?"

**Block 2: Timeline**
Q: [tailored timeline question]
Probe: "What were you using/doing before?" / "What was the frustrating part?"

**Block 3: Decision moment**
Q: [tailored commitment question]
Probe: "What almost stopped you?" / "Who else was involved in that decision?"

**Block 4: Options considered**
Q: [tailored shopping question]
Probe: "Why did you rule [alternative] out?" / "What were you worried about with [product]?"

**Block 5: First use + success**
Q: [tailored first-use question]
Probe: "Was that what you expected?" / "What would have made it better?"
Q: [tailored success question]

### Wrap (5 min)
- "If you had to explain to a colleague why you chose [product], what would you say?"
- "What would have to be true for you to recommend this to someone like you?"
- "Is there anything I didn't ask that you think is important?"

### Force-tagging cheat sheet (for the analyst)
After the interview, tag each notable response to: Push / Pull / Anxiety / Habit + Functional / Emotional / Social
```

Save to `./jtbd/[brand-slug]-discussion-guide.md`.

---

## Full mode

Run Analyse first on any transcripts provided, then run Guide using the findings to calibrate the questions. Name patterns from the transcripts that the guide should now probe more deeply. Deliver both outputs in sequence.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No output before brand-brain returns the ICP. The "customer" is always the ICP — never a generic persona.
- **Verbatim quotes only.** Never paraphrase or reconstruct a quote. If a quote is approximate or from field notes, mark it `[paraphrase]`. If the source is unclear, mark `[verify]`.
- **Situation, not demographics.** JTBD is about the context of use, not who the customer is. "A 35-year-old marketer" is not a job. "When I'm onboarding a new channel without developer support" is.
- **Four forces, not one.** Weak analysis names only the pull. Real analysis maps all four — because the anxieties and habits are where growth is actually blocked.
- **Questions uncover, not confirm.** Every guide question opens a story; none leads the witness. No "Did you find it easy to use?" — only "Walk me through what happened the first time."
- **ODI scores are math, not opinion.** Importance and satisfaction come from the transcripts; never assign them subjectively. If transcripts don't support a score, say so.

---

## What Not to Do

- Don't paraphrase quotes. The exact customer vocabulary is the asset — sanitizing it destroys the copy value.
- Don't reimplement ICP resolution or brand scanning here — call brand-brain.
- Don't produce output before brand-brain returns.
- Don't build satisfaction-survey-style guides ("On a scale of 1–10…"). The switch interview is narrative, not numerical.
- Don't conflate the functional job with a product feature. "Send push notifications" is a feature. "Keep my customers coming back between purchases without spending more on paid" is a job.
- Don't skip the social and emotional job layers — they are often what appears in winning ad copy and email subject lines.
- Don't overwrite a prior JTBD map; append to `[brand-slug]-jtbd-map.md` with a dated header so insights accumulate.

---

## Quality Checklist (self-review before presenting)

- brand-brain called and active brand loaded before any output was produced?
- ICP from brand-brain used as the analytical lens throughout?
- Every quote verbatim, attributed (Interview N), and tagged to a force and job layer?
- All four forces represented in the JTBD map (not just push + pull)?
- Functional, emotional, and social job layers all named?
- Opportunity scores only present when 5+ transcripts support them; scores marked `[verify]` if transcript evidence is thin?
- Vocabulary bank populated with exact customer language (not brand copy)?
- Discussion guide questions open-ended and narrative — no leading, no rating scales?
- Guide calibrated to the ICP's awareness stage and known push/anxiety signals from brand-brain?
- Output saved to `./jtbd/[brand-slug]-jtbd-map.md` and/or `./jtbd/[brand-slug]-discussion-guide.md`?
- Downstream sibling skills named for the user (positioning-messaging-architect, content-brief-builder, etc.)?
