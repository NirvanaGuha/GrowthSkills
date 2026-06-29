---
name: social-performance-review-summarizer
description: >
  Takes a raw platform analytics export — CSV, copy-pasted native insights, or a structured
  data dump from LinkedIn, Instagram, Facebook, X/Twitter, TikTok, or Pinterest — and
  returns an executive-ready performance summary with a top-posts ranking, platform-accurate
  engagement-rate benchmarks, trend narrative, and a prioritized action list the social
  manager can act on in the next publishing cycle. Applies a 3-horizon social review
  structure we use here: (1) What happened (data synthesis), (2) What it means (benchmark-calibrated
  interpretation), (3) What to do next (copy-ready recommendations). Designed to replace
  the Sunday-afternoon manual Excel grind. Calls `brand-brain` for voice/ICP context and
  `post-quality-reviewer-voice-auditor` for the creative diagnosis pass on underperforming
  posts. Output defaults to a Notion/Slack-ready document; optionally escalates to
  `board-exec-summary-writer` for a board-deck page. Use whenever the user says "review
  my social performance," "summarize my analytics," "what worked this week/month," "write
  a social report," "top-performing posts," "engagement breakdown," "social recap," or
  hands over a platform export and asks what to learn from it.
---

# Social Performance Review Summarizer

Raw analytics exports are noise until someone patterns them against benchmarks and connects the dots to the next action. This skill does that in one pass: it ingests the export, applies platform-accurate engagement-rate norms, surfaces the top and bottom posts with a diagnostic, writes the executive narrative, and hands back a crisp action list — all inside the brand's voice so the output is ready to share, not just to file away.

It does not manage brand context itself and it does not rewrite posts. Those live in `brand-brain` and `post-quality-reviewer-voice-auditor` respectively.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's ICP, voice, positioning, and real proof before interpreting the data. Which posts over-indexed tells you something only if you know who you're targeting.
- **`post-quality-reviewer-voice-auditor`** (optional, recommended) — called on the bottom-3 posts when the user wants a creative diagnosis alongside the numbers. Do not re-implement its scoring logic here.
- **`weekly-wins-anomalies-slack-ping`** (optional) — if the user wants a one-shot Slack alert spun off from this summary, call it and pass the key metrics.
- **`board-exec-summary-writer`** (optional) — if the user needs the output formatted for a board deck or investor update, delegate the narrative section to it after producing the data layer here.
- **`social-content-calendar-builder`** (context, when available) — the upstream calendar that produced this period's posts; reference it to map performance back to content themes.

---

## How a run works

```
Step 0  Load the brand         ──► call brand-brain; note ICP, voice, benchmark platform
Step 1  Parse the export        ──► normalize to canonical schema (see below)
Step 2  Compute rates           ──► engagement rate per post, platform-accurate formula
Step 3  Rank + diagnose         ──► top-3 / bottom-3 posts; pattern across the middle
Step 4  Benchmark               ──► compare to platform norms + brand's own historical baseline
Step 5  Write the summary       ──► 3-horizon structure (our working model); brand voice; no invented numbers
Step 6  Action list             ──► ≤5 prioritized, each tied to a specific finding
Step 7  (optional) escalate     ──► call post-quality-reviewer-voice-auditor on bottom posts
                                     call board-exec-summary-writer if deck format requested
Step 8  Save                    ──► ./social-reviews/[brand-slug]-[YYYY-MM-DD].md
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill. It returns the active brand's digest: ICP + awareness tendency, voice adjectives, banned words, positioning. Use the ICP to frame what "good engagement" means for this audience — B2B SaaS ICP on LinkedIn has different norms than D2C on Instagram.

**Fallback if brand-brain is not installed:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if absent, ask the user for: (a) brand name, (b) primary platform(s), (c) ICP in one sentence, (d) 2–3 voice adjectives. Proceed only once you have these four inputs.

### Step 1 — Parse and normalize

Accept any of:
- **Native CSV export** (LinkedIn Analytics, Meta Business Suite, X Analytics, TikTok Creator Studio, Pinterest Analytics)
- **Copy-pasted insights panel** (screenshot description or plain text)
- **Structured object** (JSON/dict from an analytics MCP or sheet)

Normalize each post to the canonical schema:

```
post_id | date | format (post/reel/story/carousel/pin) | impressions
| reach | engagements | likes | comments | shares/saves | link_clicks | profile_visits
| video_views (if applicable) | caption_snippet (≤100 chars)
```

Flag missing fields explicitly; never impute reach from impressions or vice versa. If the export lacks `reach`, compute ER on impressions and say so.

---

## The 3-Horizon Social Review Structure (our working model)

### Horizon 1 — What happened (data layer)

**Period summary table** — absolute numbers, always:

| Metric | This Period | Prior Period | Delta |
|---|---|---|---|
| Impressions | | | |
| Reach | | | |
| Engagements | | | |
| Avg. Engagement Rate | | | |
| Link Clicks / Traffic | | | |
| Follower Change | | | |
| Posts Published | | | |

Mark `[no prior period provided]` if no comparison data was given — do not fabricate deltas.

**Top-3 posts** (by ER, not raw engagements — see ER formulas below):
For each: post date, format, ER, absolute engagements, caption snippet, and one-line diagnosis of *why it likely worked* (hook, format, timing, topic).

**Bottom-3 posts** (by ER): same fields, one-line diagnosis of likely miss.

### Horizon 2 — What it means (benchmark layer)

**Platform-accurate engagement rate formulas** — use the correct denominator per platform:

| Platform | ER Formula | 2024–25 average benchmarks |
|---|---|---|
| LinkedIn (company page) | (likes + comments + shares + clicks) / impressions | ~2–4% organic [verify current] |
| Instagram (feed post) | (likes + comments + saves + shares) / reach | ~1–5% depends on account size [verify] |
| Instagram Reels | (likes + comments + shares + saves) / reach | ~3–7% [verify] |
| Facebook (page post) | (reactions + comments + shares) / reach | ~0.5–2% [verify] |
| TikTok | (likes + comments + shares + saves) / views | ~3–9% [verify] |
| X / Twitter | (likes + reposts + replies + bookmarks) / impressions | ~0.5–2% [verify] |
| Pinterest | (saves + clicks + close-ups) / impressions | ~0.5–3% [verify] |

Always use the brand's own historical baseline as the primary benchmark when available; published industry ranges are secondary and marked `[verify]` because they shift. State the formula used in the report.

**Narrative interpretation** — 3–5 sentences answering: Did the brand beat/miss its own baseline? Which formats or content themes drove the variance? Are there ICP-alignment signals in the comment/save patterns worth noting?

### Horizon 3 — What to do next (action layer)

Exactly **5 prioritized actions**, each following this structure:

```
[Priority #] [Action verb + specific action]
Finding: [the data point that triggered this]
Why it matters: [one sentence]
Suggested next step: [specific and executable — not "post more Reels"]
```

Action types to draw from (pick only the ones the data actually supports):
- Double down on a proven format or topic theme
- Kill or reduce a consistently underperforming format
- Optimize publish timing (only if time-of-day data is present)
- A/B test a hypothesis surfaced by the top-post diagnosis
- Escalate a strong post to paid amplification
- Fix a structural issue (bio link, CTA missing, caption too long for platform)
- Kick a creative diagnosis (delegate to `post-quality-reviewer-voice-auditor`)

Never invent actions not grounded in the data. If fewer than 5 findings warrant action, write fewer.

---

## Output format

```markdown
# Social Performance Review — [Brand] / [Platform(s)] / [Period]
*Brand context loaded from brand-brain: [slug] — [ICP one-liner]*
*ER formula used: [state it]*

## Period Summary
[table]

## Top Posts
[ranked list, 3 posts]

## Bottom Posts
[ranked list, 3 posts]

## What It Means
[3–5 sentence narrative]

## Action List
[≤5 prioritized actions]

## Notes & Data Gaps
[missing fields, imputed metrics, [verify] items]
```

Save to `./social-reviews/[brand-slug]-[platform]-[YYYY-MM-DD].md`.

For a Slack ping: call `weekly-wins-anomalies-slack-ping` and pass the top win, biggest anomaly, and first action.
For a board-deck page: call `board-exec-summary-writer` and pass the Horizon 2 narrative + Period Summary table.

---

## Principles

- **Brand-brain first.** No interpretation before the ICP and voice are loaded. The same ER number means different things for a B2B vs D2C brand.
- **Correct denominator.** ER on impressions is not ER on reach; confusing them makes the benchmark comparison meaningless. State the formula used, every time.
- **Own baseline > industry average.** The brand's prior-period trend is more predictive than a published industry number. If there is no baseline, say so rather than benchmarking against a stale range.
- **Diagnose, don't just rank.** A list of numbers with no interpretation is useless. Every top/bottom post gets a one-line cause hypothesis.
- **Actions tied to findings.** No action exists in the output that is not directly traceable to a data point in the report.
- **No invented numbers.** Missing data is flagged, not estimated. Metrics are marked `[verify]` when derived from external benchmarks rather than the export.
- **Composition over duplication.** Creative diagnosis of underperformers belongs to `post-quality-reviewer-voice-auditor`; call it, don't rebuild it.

---

## What not to do

- Do not compute ER without stating the formula and denominator.
- Do not benchmark against industry averages without marking them `[verify]` — they shift.
- Do not write actions like "post more consistently" unless you have cadence data showing inconsistency.
- Do not rewrite the underperforming posts — that is `post-quality-reviewer-voice-auditor`'s job; call it.
- Do not omit the brand-brain call because "it's just a metrics report." ICP context determines what good looks like.
- Do not produce a Horizon 3 action list before completing Horizon 1 data and Horizon 2 benchmarks — order is non-negotiable.
- Do not invent prior-period deltas if no comparison data was provided.

---

## Quality checklist

- `brand-brain` called and ICP + voice loaded before any interpretation?
- Each post normalized to canonical schema; missing fields flagged (not imputed)?
- ER computed with platform-correct formula stated explicitly in the output?
- Top-3 and bottom-3 ranked by ER (not raw engagements), each with a cause diagnosis?
- Horizon 2 narrative answers did-beat-miss, format variance, and ICP-alignment signals?
- Action list: every item traceable to a specific data point; speculative items removed?
- Bottom posts referred to `post-quality-reviewer-voice-auditor` for creative diagnosis (or note that it was skipped)?
- Report saved to `./social-reviews/[brand-slug]-[platform]-[YYYY-MM-DD].md`?
- `[verify]` on all benchmark ranges; no invented deltas?
