---
name: board-exec-summary-writer
description: >
  Converts a detailed strategy document, marketing plan, KPI set, campaign debrief, or growth
  report into a board-ready one-pager or exec-deck talking points. Applies the Situation /
  Complication / Resolution (SCR) pyramid to distill any length of input into the three things
  a board or C-suite actually needs: what's true now, what decision or action it demands, and
  what the risks are. Output is on-brand (voice, number discipline, banned words) because this
  skill always loads the active brand context from `brand-brain` before writing a single word.
  Two modes: One-Pager (prose, one page, shareable) and Talking-Points Deck (slide-by-slide
  speaker notes). Use whenever the user says "write a board update," "summarize this for the
  exec team," "I need a one-pager for leadership," "board deck talking points," "exec summary,"
  "make this exec-ready," "turn this into a board slide," or hands over a long strategy doc and
  needs the distilled version for leadership. Writes summaries only — does not rebuild the
  underlying strategy, plan, or KPI model.
---

# Board & Exec Summary Writer

Every board packet and exec deck has the same silent contract: executives will not read past the first page if it fails to tell them what they need to decide. This skill enforces that contract. It takes any detailed input — strategy doc, growth plan, KPI dashboard, campaign debrief, annual review — and produces either a one-page summary or a deck of sharp talking points that land in a 10-minute read or a 5-minute standup.

The framework is the SCR Pyramid (Situation / Complication / Resolution), borrowed from McKinsey's Minto Pyramid Principle. It is not a template; it is a discipline: lead with what's true, surface the tension that makes action necessary, then make the recommendation clear. Every number is cited, every risk is named, and nothing goes to the executive without the brand voice check.

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads the active brand's voice, banned words, offer mechanics, and real proof. No exec copy ships before brand context is confirmed.
- **`positioning-reviewer`** *(optional)* — when the input includes a positioning statement, call this to sanity-check it before surfacing it upward.
- **`roi-business-case-calculator`** *(optional)* — when the input includes a budget ask or ROI projection, call this to pressure-test the numbers before they go to the board.
- **`analytics-report-reviewer`** *(optional)* — when the input is a metrics report, call this to flag unsupported claims before they become board-level talking points.
- **`tradeoff-memo-writer`** *(optional)* — when the recommendation involves competing options, call this to get a clean tradeoff framing before drafting the summary.

---

## How a run works

```
Step 0  Load the brand         ──► call brand-brain; get voice, banned words, proof
Step 1  Classify the input     ──► what type of doc? what decision does it demand?
Step 2  Pick the mode          ──► One-Pager (default) | Talking-Points Deck (on request)
Step 3  Apply the SCR Pyramid  ──► Situation → Complication → Resolution
Step 4  Self-review            ──► brand voice, number discipline, decision clarity
Step 5  Deliver + save         ──► inline or saved to ./board/
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request and any named brand. It returns the active brand's digest — voice adjectives, banned words, real proof, offer mechanics, positioning, and ICP. Do not write a single sentence of the summary until it returns.

Obey the returned voice and banned words as hard overrides. Use only real proof from the brand digest; mark any metric or claim not confirmed in the digest as `[verify]`. Anchor the executive narrative to the brand's real positioning — do not introduce claims the brand has not made.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If none exists, ask the user to install `brand-brain` (preferred) or answer a 4-question setup (what the brand is · ICP · offer mechanics · 3 voice adjectives + banned words), then proceed.

### Step 1 — Classify the input and surface the decision

Before writing anything, read the full input and answer these three questions internally:

1. **What type of document is this?** (Strategy plan / KPI report / campaign debrief / budget request / annual review / other)
2. **What is the one decision or action this asks of the executive audience?** If the input does not make this explicit, surface it as the key missing element and flag it to the user.
3. **Who is the audience?** Board (financial governance, risk, long-horizon) vs. exec team (operational decisions, short-horizon). This sets the vocabulary, specificity, and recommendation frame.

If the input does not contain enough substance to produce a credible summary, say so directly. Do not generate a confident-sounding summary from thin material.

### Step 2 — Pick the mode

- **One-Pager (default).** A single-page prose document: 4–6 paragraphs, hard limit ~400 words, shareable as a standalone document or email attachment.
- **Talking-Points Deck (on request).** Triggered by "deck," "slide," "talking points," "speaker notes," or an explicit request for a deck structure → 5–7 slide stubs with a headline, 3–4 speaker-note bullets each, and a clear narrative arc.

When unsure, default to One-Pager and offer the Talking-Points Deck at the end.

---

## SCR Pyramid — the framework

The SCR Pyramid (Barbara Minto / McKinsey) structures every summary in the same order an executive's brain needs the information:

```
S  Situation    ── What is undeniably true right now?
C  Complication ── What has changed, or what tension makes action necessary?
R  Resolution   ── What should we do (or decide), and what does success look like?
```

Supporting evidence — numbers, milestones, risks — hangs under the Resolution, not above it. Burying the recommendation in paragraph three is the most common executive-communication failure; the SCR structure prevents it.

### How to apply SCR to growth/marketing documents

| Input type | Situation | Complication | Resolution |
|---|---|---|---|
| Strategy doc | Market position + current performance | Gap between where we are and where the plan requires us to be | Recommendation: invest, pivot, or hold — with success criteria |
| KPI / metrics report | Metric snapshot (period, actuals vs. target) | Which metrics are off-track and why (root cause, not just the delta) | Next action: investigate, accelerate, or accept + owner + date |
| Campaign debrief | Campaign goal + result summary | What performed above/below expectation and the attributed cause | What we carry forward, what we change, and what we test next |
| Budget request | Current state requiring the investment | Why the status quo is inadequate (cost, opportunity, risk) | Ask (amount, timeline, expected return) + fallback if not approved |
| Annual review | Year-in-review headline metrics | Where the strategy worked vs. where it missed | Updated priorities + the one decision leadership must make now |

### Number discipline

Executives distrust summaries bloated with metrics. Apply the "3-number rule": each SCR section gets at most three headline numbers. Everything else is relegated to an appendix note or source doc. Every number must:
- Have a time period attached (not "revenue is up" — "Q2 revenue: $X, up Y% YoY")
- Have a source (GA4, CRM, finance model — whatever the input used)
- Be marked `[verify]` if it was not present in the input or the brand brain

### Risk articulation

Boards need risks named, not softened. Every Resolution must include a **Risk** line:

```
Risk: [what could prevent this from working] — [what we'd see early / tripwire]
```

Naming a tripwire (an early-warning signal) is what separates a credible board summary from a promotional one.

---

## One-Pager output format

```
## [Brand Name] — [Document title / topic] — [Date]
**Audience:** [Board | Exec team | Name of individual]

**Situation**
[1–2 sentences. What is undeniably true. Current state, current performance.]

**Complication**
[1–2 sentences. The tension that demands attention. What changed, what's at risk, what the gap is.]

**Recommendation**
[1–2 sentences. The specific action or decision requested. Owner. Timeline.]

**Key metrics** (max 3)
- [Metric 1: value, period, source]
- [Metric 2: value, period, source]
- [Metric 3: value, period, source]

**Risk**
[Named risk + tripwire signal]

**Next step**
[Single next action, owner, date]
```

Save to `./board/[slug]-exec-summary-[YYYYMMDD].md` when the user asks to save; otherwise deliver inline.

---

## Talking-Points Deck output format

When in Talking-Points Deck mode, produce 5–7 slide stubs in this arc:

| Slide | Title convention | Content |
|---|---|---|
| 1 | State of Play | Situation in one headline sentence + 2–3 supporting bullets |
| 2 | The Challenge | Complication — what has changed and why it matters now |
| 3 | Our Recommendation | Resolution headline + owner + timeline |
| 4 | The Numbers | Up to 3 headline metrics with source + period; no data dumps |
| 5 | Risks & Tripwires | Named risks with early-warning signals |
| 6 | Next Steps | 3 actions max, with owner and date |
| 7 *(optional)* | Appendix / Source Data | Supporting detail for questions |

Each slide stub has:
- **Headline:** the one sentence a board member could tweet (the "So what")
- **Bullets:** 3–4 speaker-note bullets (what to say, not what to show)
- **Handoff note:** any visual or chart the designer needs to build

Save to `./board/[slug]-talking-points-[YYYYMMDD].md` when asked to save.

---

## Principles

- **Brand-brain first.** No summary before brand context loads. Voice, banned words, and real proof are non-negotiable.
- **Decision clarity above all.** If the audience cannot identify the single decision within 60 seconds, the summary has failed.
- **SCR, not BLUF-lite.** The Situation grounds the reader; the Complication creates urgency; the Resolution earns authority. Skipping any leg collapses the structure.
- **3-number rule per section.** Metric density kills credibility. Surface the headline numbers; cite the source doc for the rest.
- **Name the risk.** A summary without a risk line is advocacy, not analysis. Boards notice the omission.
- **Honest proof only.** Real numbers with sources, or `[verify]`. Never invent a metric to strengthen the narrative.
- **Calibrate the audience.** Board language is governance-frame (capital, risk, long horizon). Exec-team language is operational (owner, quarter, KPI). Write to the stated audience, not the author.

## What Not to Do

- Don't write a summary before `brand-brain` returns the active brand.
- Don't bury the recommendation. It goes in the Resolution slot, not at the end of a long narrative.
- Don't generate a confident summary from thin input — say the substance is insufficient and ask for more.
- Don't soften risks to protect the recommendation. Name them; let the executive decide.
- Don't exceed the 3-number-per-section limit. If more numbers are critical, they go in the appendix.
- Don't reimplement brand scanning, voice analysis, or proof verification here — call `brand-brain`.
- Don't rebuild the underlying strategy or KPI model — summarize what's given; flag gaps.
- Don't use jargon, exclamation marks, or emojis unless the brand explicitly permits them.

## Quality Checklist (self-review before presenting)

- `brand-brain` called; active brand loaded; voice + banned words honored throughout?
- Every number has a time period, a source, and is `[verify]`-tagged if unconfirmed in input?
- SCR structure intact: Situation grounds, Complication creates urgency, Resolution leads with the ask?
- One-Pager: under ~400 words; exactly one recommended action with owner + date?
- Talking-Points: 5–7 slides; each has a tweetable headline and 3–4 speaker-note bullets?
- Risk line present with a named tripwire signal?
- Audience calibration confirmed (board vs. exec team)?
- No invented proof, no softened risks, no buried recommendations?
