---
name: post-quality-reviewer-voice-auditor
description: >
  Structured quality gate for any social post or batch before it goes live. Scores each draft across
  five axes — hook strength, CTA presence and quality, platform-fit (format/length/algorithm), brand-voice
  consistency, and AI-slop risk — using a 1–5 rubric per axis with a single publish/revise/kill verdict
  per post. Works on a single draft or an entire calendar batch. Brand context comes from `brand-brain`
  (not re-derived here). When a post scores below threshold the reviewer surfaces the specific failure
  and, where applicable, calls the appropriate sibling skill to fix it rather than rewriting from scratch.
  Outputs a reviewer scorecard plus inline line-level callouts. Saves batch scorecards to
  `./social-review/[slug]-review.md` when three or more posts are reviewed together. Use when the user
  says "review this post," "QA my drafts," "check brand voice," "is this on-brand," "audit my social
  batch," "will this hook work," "does this fit LinkedIn/TikTok/Instagram/X," "is this AI-sounding,"
  or hands over any social draft and asks for a verdict before publishing.
---

# Post Quality Reviewer & Voice Auditor

Every social post that ships without a review is a brand-voice lottery. This skill is the quality gate — a structured reviewer that scores drafts on the five dimensions that determine whether a post earns attention, stays on-brand, and fits the platform it was written for. It gives verdicts, not opinions. It flags what broke and routes to the right fix skill rather than rewriting from scratch.

It reviews. It does not generate source posts — those come from `linkedin-post-writer`, `x-thread-writer`, `instagram-caption-reel-script-writer`, `tiktok-script-hook-generator`, or the `social-content-calendar-builder`.

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads the active brand's voice adjectives, banned words, offer mechanics, proof, and ICP. This skill does not re-derive brand context.
- **`headline-hook-generator`** (optional, on-demand) — called when a hook scores ≤2 and the user wants a fixed version, not just the flag.
- **`cta-variant-generator`** (optional, on-demand) — called when the CTA scores ≤2 and the user wants a replacement.
- **`de-slop-humanize-pass`** (optional, on-demand) — called when AI-slop risk scores ≥4 and the user wants a rewrite pass.
- **`editorial-style-guide`** (optional, reference) — consulted when style-guide rules exist for the brand and a voice failure needs precise attribution.

---

## How a run works

```
Step 0  Load the brand         ──► call `brand-brain`; extract voice, banned words, ICP
Step 1  Identify the mode      ──► Single (default) | Batch (3+ posts or "review my batch")
Step 2  Score each post        ──► five axes, 1–5 each, see rubric below
Step 3  Apply verdict per post ──► Publish / Revise / Kill
Step 4  Surface fixes          ──► inline callouts; call fix skills if user wants rewrites
Step 5  (Batch) write scorecard ──► save to ./social-review/[brand-slug]-review.md
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** before reading the first draft. Use the returned digest: voice adjectives, banned words/phrases, ICP + awareness tendency, positioning, real proof. If `brand-brain` is absent, read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none, ask the user to install `brand-brain` or answer a 4-question mini-setup (ICP, voice adjectives, banned words, offer) before proceeding.

---

## The Five-Axis Rubric

Score 1–5. For each axis the reviewer must produce: **score + a one-line rationale + a specific line-level callout** (quote the offending or exemplary text). Never give a score without evidence.

### Axis 1 — Hook Strength (lines 1–2 / first frame)

| Score | Meaning |
|-------|---------|
| 5 | Pattern-interrupt or scroll-stopper; creates tension, contradiction, bold claim, or stakes the reader cares about before they can scroll past |
| 4 | Clear value signal; gives a reason to keep reading but doesn't break pattern |
| 3 | Neutral open; sets context but doesn't pull |
| 2 | Generic ("Here's something I learned") or buries the lead |
| 1 | No hook; opens with self-promotion, a date, or a throat-clear |

**Threshold to flag: ≤3 on LinkedIn/X/TikTok (high-competition feeds); ≤2 on Instagram caption (hook is less critical than visual)**. Recommend `headline-hook-generator` on scores ≤2.

### Axis 2 — CTA Presence & Quality

| Score | Meaning |
|-------|---------|
| 5 | Clear, single, specific ask matched to the post's awareness stage; frictionless and on-brand |
| 4 | Present and directional but slightly generic ("check it out") |
| 3 | Weak or buried (scroll to find it); or an implicit CTA relying on the reader to infer |
| 2 | No CTA or a double-stacked competing CTA |
| 1 | CTA contradicts the post's offer or drives to a dead destination |

**Threshold to flag: ≤3.** Apply the Schwartz awareness ladder: a problem-aware post shouldn't close with "Buy now." Recommend `cta-variant-generator` on scores ≤2.

### Axis 3 — Platform Fit

Evaluate against the real spec for the declared platform. If platform is not stated, infer from format or ask before scoring.

| Platform | What "5" looks like |
|----------|---------------------|
| **LinkedIn** | ≤1,300 chars before "see more"; hook in line 1; no markdown headers or bullet overload; thought-leadership or professional insight angle; first-person narrative earns higher engagement [verify exact char limit per current LinkedIn spec] |
| **X / Twitter** | ≤280 chars per tweet; or thread with each tweet standalone; no filler bridge tweets; hooks ≤70 chars |
| **Instagram** | Caption ≤2,200 chars; hook in line 1 above fold; hashtags (5–15, relevant, mixed broad/niche) in caption or first comment; visual described or assumed strong |
| **TikTok** | Hook in first 3 seconds (first 1–2 lines of script); trending audio or hook variant noted; ≤60s if engagement-focused; CTA as pattern-interrupt not outro |
| **Facebook** | Short (≤80 chars for link posts) or long-form story; no hashtag overload (≤3 organic) |

Score 1 for a post that would be clipped, penalized, or algorithmically suppressed on its platform. Score 3 for a post that technically fits but ignores platform conventions. Score 5 for a post that feels native.

### Axis 4 — Brand Voice Consistency

Use the returned `brand-brain` digest as the scoring key. Voice adjectives = the target. Banned words = automatic deductions.

| Score | Meaning |
|-------|---------|
| 5 | All voice adjectives present; zero banned words; tone matches ICP; could only have been written by this brand |
| 4 | Mostly on-voice; one mild drift (adjective missing, slightly generic phrasing) |
| 3 | Neutral — not off-brand but not distinctively on-brand |
| 2 | One banned word used OR tone inconsistency large enough to confuse brand perception |
| 1 | Multiple banned words OR tone directly contradicts brand positioning |

Every banned-word hit is quoted verbatim. Every voice adjective present is noted. If the brand has a style guide loaded via `editorial-style-guide`, cite specific rules. Recommend `de-slop-humanize-pass` on scores ≤2 when the failure is generic/AI-sounding prose.

### Axis 5 — AI-Slop Risk

This axis scores the likelihood a reader tags the post as AI-generated filler — regardless of how it was written.

| Score | Meaning |
|-------|---------|
| 1 | No slop signals; specific, textured, human-paced, with friction and specificity |
| 2 | Mostly clean; one or two filler phrases |
| 3 | Several slop patterns; feels templated but readable |
| 4 | Unmistakable AI skeleton: em-dash overload, "In today's X landscape," numbered list of vague insights, hollow motivational close |
| 5 | Full-slop: reads like a generic LinkedIn thought-leadership carousel from a bot |

Common slop signals (flag any): *"In today's fast-paced world," "game-changer," "navigate the landscape," "it's not just about X, it's about Y," "I wanted to share," preamble before the actual take, closing with "What do you think?", over-reliance on em-dashes and colons as structural devices, list of 3–5 identical-length bullet insights with no concrete detail.*

Recommend `de-slop-humanize-pass` on scores ≥4.

---

## Verdict Thresholds

After scoring all five axes, assign a verdict:

| Verdict | Condition |
|---------|-----------|
| **Publish** | No axis ≤2; total score ≥18/25; no banned words |
| **Revise** | Any axis = 3 on a threshold-gated dimension, OR total 13–17, OR one axis = 2 |
| **Kill** | Any axis ≤1 OR banned word + voice axis ≤2 OR total ≤12 |

A "Revise" verdict always names the specific axis to fix and, if the fix is a sibling skill, names the skill.

---

## Scorecard Format

### Single-post output (inline)

```
## Review — [Post title / first 6 words] · [Platform] · [Brand slug]
Brand voice loaded via brand-brain: [slug]

| Axis | Score | Note |
|------|-------|------|
| Hook Strength | /5 | "[quoted text]" — [rationale] |
| CTA Presence & Quality | /5 | "[quoted text]" — [rationale] |
| Platform Fit | /5 | [platform] — [rationale] |
| Brand Voice | /5 | Voice ✓/✗ · Banned hits: [none / "word"] |
| AI-Slop Risk | /5 | [rationale; slop signals if any] |
| **Total** | **/25** | |

**Verdict: [Publish / Revise / Kill]**
Priority fix: [axis] → [specific action or skill to call]
```

### Batch output (3+ posts)

One scorecard table with a row per post, then per-post inline callouts below. Saved to `./social-review/[brand-slug]-review-[YYYY-MM-DD].md`.

```
## Batch Review — [Brand slug] — [date]
Brand context: [slug] via brand-brain

| # | Post (6-word excerpt) | Platform | Hook | CTA | Platform | Voice | Slop | Total | Verdict |
|---|-----------------------|----------|------|-----|----------|-------|------|-------|---------|

### Per-post callouts
[One section per post: quoted line-level issues, banned-word flags, fix skill recommendations]
```

---

## Principles

- **Brand-brain first.** No review begins before `brand-brain` returns the active brand digest. Voice and banned words are the scoring key — not the reviewer's assumptions.
- **Verdicts, not opinions.** Every score needs quoted evidence from the post. Scores without quotes are disallowed.
- **Review, don't rewrite.** This skill identifies and flags. Rewriting belongs to the generating skill or the relevant fix skill. Do not produce replacement post copy unless explicitly asked.
- **Platform rules are non-negotiable.** A post that breaks the platform's format conventions (length, structure, algorithmic signals) cannot score above 3 on Platform Fit, regardless of how good the copy is.
- **Slop is a brand-voice risk.** AI-generated patterns erode brand distinctiveness. Score it independently even when the post is technically on-brand.
- **One verdict per post.** No hedging. Publish means publish; Kill means kill. "Revise" with a blank fix action is not a verdict.

## What Not to Do

- Don't skip `brand-brain` and infer voice from the drafts themselves — drafts may already be off-voice.
- Don't rewrite the post unless the user asks for it; route to the fix skill instead.
- Don't conflate Platform Fit with Hook Strength — a great hook on the wrong format still fails platform fit.
- Don't give a Publish verdict when a banned word appears anywhere in the post.
- Don't use vague feedback ("this could be stronger") — every note must cite a specific line.
- Don't score AI-slop on the tool used to write it; score the text itself.

## Quality Checklist (self-review before presenting)

- `brand-brain` called; voice adjectives and banned-words list extracted and used as scoring key?
- Every score has a quoted line-level citation from the post?
- Platform identified (inferred or confirmed) before Platform Fit axis scored?
- Banned-word hits surfaced verbatim and reflected in Voice score (≤2)?
- Verdict matches the threshold logic (no manual override without stated rationale)?
- Fix skills named specifically for any axis scoring below threshold?
- Batch (3+ posts): scorecard saved to `./social-review/[brand-slug]-review-[YYYY-MM-DD].md`?
