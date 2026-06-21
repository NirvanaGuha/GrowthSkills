---
name: weekly-wins-anomalies-slack-ping
description: >
  Turns a week's metric snapshot into one punchy, ready-to-send Slack message: the top win,
  the biggest anomaly with a probable cause, and one recommended action — all framed in the
  brand's real voice. Skips the essay. Skips the wall of numbers. Hands the team a single
  message they'll actually read and act on in under 30 seconds. Opinionated by design: it
  picks the one best win and the one most actionable anomaly, not a laundry list.
  Applies the ICE (Impact / Confidence / Ease) signal-triage framework to rank what
  deserves the headline, and the classic GA4 measurement-gotcha checklist to catch fake
  anomalies before they go out as real alerts.
  Use when the user says "write my weekly Slack update," "weekly wins message," "send the
  weekly metrics ping," "anomaly alert for the team," "weekly standup post," "summarize
  this week's numbers for Slack," "what's worth flagging this week," or pastes a metrics
  dump and asks for the Slack version.
---

# Weekly Wins & Anomalies Slack Ping

One Slack message. One win. One anomaly. One action. Done.

Every growth team has a weekly metrics review. Most of them produce a wall of numbers, a thread nobody finishes, or a "good week overall" non-statement. This skill produces the opposite: a single framed message that a channel of ten people will read in full and leave knowing what happened and what to do next.

The skill is deliberately opinionated. It does not summarize everything. It selects — using a signal-ranking framework and a measurement-gotcha pass — then writes in the brand's actual voice.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's voice, ICP, and banned words so the Slack message sounds like the team, not a template. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, voice adjectives (e.g. direct / data-first / no-hype), and any banned words or emoji rules before proceeding.
- **`data-qa-measurement-gotcha-checker`** — gates the anomaly section; every flagged spike or drop must survive a GA4 gotcha pass (attribution window, sampling, (not set), self-referral, SRM, spam hostname) before it ships as a real alert. Compose inline when absent.
- **`analytics-report-reviewer`** *(optional)* — if the user provides a longer report draft rather than raw numbers, call this first to strip unsupported claims before writing the ping.
- **`weekly-email-push-metrics-digest`** *(optional, complementary)* — for teams that want both a Slack ping (this skill) and a longer email/Notion digest (that skill) from the same data.

---

## How a run works

```
Step 0  Load brand context     ──► call brand-brain
Step 1  Ingest + triage        ──► parse the metric snapshot; apply ICE to rank signals
Step 2  Gotcha-check anomalies ──► filter fake spikes/drops before they become alerts
Step 3  Draft the ping         ──► one win + one anomaly + one action in brand voice
Step 4  Self-review            ──► quality checklist before presenting
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Use the returned voice adjectives, banned words, and tone guidance as hard overrides on every line of copy in the output.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, voice adjectives, and banned words before proceeding.

### Step 1 — Ingest + triage (ICE signal ranking)

Accept any form of weekly metric input: a paste of numbers, a screenshot, a CSV, a GA4 / ESP / push-platform export, or a paragraph summary. Extract every meaningful delta (WoW, or MoM if WoW isn't available).

**ICE framework — rank each signal on three axes (1–3 each):**

| Axis | Question | What scores high |
|---|---|---|
| **Impact** | How much does this move revenue, retention, or a north-star metric? | Checkout conversion, trial starts, activated users, CAC, LTV, top-funnel volume |
| **Confidence** | How trustworthy is the signal? (sample size, data-quality gate, replicability) | Large-sample events over a full week; survives gotcha-check |
| **Ease of action** | Can the team do something about it this week? | A/B already queued, copy swap, bid adjustment, sequence change |

Sum ICE (max 9). The **highest-scoring positive delta** becomes the win headline. The **highest-scoring negative or unexplained delta** that also passes Step 2 becomes the anomaly.

If two signals tie on ICE, prefer the one closer to revenue.

Do not expose the ICE scores in the Slack message — they are the internal selection logic only.

### Step 2 — Gotcha-check anomalies (data quality gate)

Before naming anything an anomaly, run it through the GA4 / measurement gotcha checklist. A signal that fails is flagged for the user privately ("not surfacing X — likely [cause]") but does not appear in the Slack message as a real anomaly.

**Gotcha checklist (required for every anomaly candidate):**

1. **Attribution window shift** — did the lookback window change, or a channel recently move to a longer window? (Common after GA4 or CRM config updates)
2. **Sampling** — is the metric pulled from a sampled GA4 report? Explorers above ~500k sessions sample by default; standard reports do not. Verify before alarming on a sub-1% delta.
3. **(not set) inflation** — did `(not set)` grow as a dimension value? Common after GTM breaks, hostname misconfigs, or a CDN change.
4. **Self-referral / spam hostname** — spike in sessions/users from the brand's own domain, localhost, or a known spam hostname? Strip before calling it organic growth.
5. **SRM (Sample Ratio Mismatch)** — for experiment-adjacent metrics: did one variant receive significantly more/less traffic than the split intended? If yes, the metric is unreliable.
6. **Seasonality / calendar artifact** — is the comparison period structurally different? (Holiday week, shorter month, payday effect, iOS update, competitor announcement)
7. **Tracking gap / deploy event** — did a code deploy, GTM publish, or consent-banner change coincide with the inflection?

If the anomaly survives all seven checks, it is a real anomaly. If it fails one, note the probable cause and surface it as a "watch not alarm."

### Step 3 — Draft the Slack ping

**Three-block structure (fixed):**

```
:trophy: WIN — [one sentence, specific metric + delta + context]

:rotating_light: WATCH — [one sentence, anomaly + probable cause in plain English]
  └ Confidence: [real / watch / artifact] — [one-line reason based on gotcha pass]

:arrow_right: ACTION — [one sentence, specific and owner-assignable this week]
```

**Writing rules:**
- Every number is real and sourced from the input; mark anything estimated or partially sampled `[~approx]`
- Win headline leads with the outcome, not the activity: "Trial starts +18% WoW" not "We sent 3 more emails"
- Anomaly names the probable cause confidently when it survived the gotcha-check, hedges when it didn't
- Action is specific enough to assign: names the channel, the lever, and the intended direction
- Brand voice governs tone throughout — emoji use (or not), formality, banned words
- Total message: fits in a single Slack screen (~180–240 words); does not require scrolling
- No bullet walls, no league-table of every metric, no "overall it was a solid week"

**Emoji defaults (override with brand voice):**
`:trophy:` for win, `:rotating_light:` for anomaly, `:arrow_right:` for action. If brand voice bans emoji, replace with bold labels: **WIN**, **WATCH**, **ACTION**.

### Step 4 — Self-review

Run the quality checklist (below) before presenting. If any item fails, fix before outputting.

---

## Output format

**Default:** inline Slack message, copy-ready.

**On request**, also save to `./reports/weekly-ping-[YYYY-MM-DD].md` — useful when the user feeds this into `weekly-ops-digest` or archives for trend-spotting.

---

## Principles (Non-Negotiable)

- **Signal over noise.** One win, one anomaly — not a ranked table of every metric. The value is the selection, not the transcription.
- **Gotcha-gate before alarming.** A fake anomaly destroys credibility faster than a missed one. Every spike/drop gets the seven-point gotcha pass before it ships as a real alert.
- **ICE, not gut.** The signal that gets the headline is the one that scores highest on Impact × Confidence × Ease — not the one the sender is most proud of.
- **Brand voice is load-bearing.** A Slack message in the wrong tone either gets ignored or misread. brand-brain call is not optional.
- **Specific action, specific owner.** "Keep an eye on it" is not an action. Every ping ends with something a named person (or function) can do this week.
- **Real numbers or [~approx].** Never invent a delta; never round a number so aggressively it changes meaning.

## What Not to Do

- Don't include more than one win and one anomaly — if the user wants a fuller digest, point to `weekly-email-push-metrics-digest`.
- Don't skip the gotcha-check because the anomaly "looks real" — tracking breaks are the most common cause of alarming spikes.
- Don't restate every input metric; summarize and select.
- Don't use vague language: "slight uptick," "mostly flat," "seems like." Be specific or say the data is insufficient.
- Don't surface a metric the team can't act on this week as the headline action.
- Don't let a banned word or off-brand emoji through — brand voice overrides defaults.

## Quality Checklist (self-review before presenting)

- [ ] `brand-brain` called; voice + banned words applied throughout?
- [ ] ICE triage run; win = highest-scoring positive delta, anomaly = highest-scoring negative that passed gotcha?
- [ ] Every anomaly candidate passed (or explicitly failed with cause noted) all seven gotcha checks?
- [ ] Three-block structure intact: WIN → WATCH → ACTION?
- [ ] All numbers sourced from input; estimated values marked `[~approx]`?
- [ ] Action is specific, this-week executable, and assignable?
- [ ] Total message fits a single Slack screen (~240 words max)?
- [ ] No metric walls, no filler phrases, no vague non-actions?
