---
name: weekly-email-push-metrics-digest
description: >
  Turns raw ESP and push-platform weekly stats into a Slack- or Notion-ready
  status update — key metrics, WoW delta with significance flag, top-performing
  send callout, and one prioritized action. Works with any ESP (Klaviyo, Mailchimp,
  Brevo, ActiveCampaign, HubSpot) and any push platform (PushEngage, OneSignal,
  WebEngage, MoEngage, Iterable). Reads the active brand's list-health baselines
  and voice from brand-brain so every digest sounds on-brand and uses the right
  benchmarks — not generic industry averages. Saves a dated artifact to
  ./email-push-reports/ for trend-over-time comparisons. Use when the user says
  "compile my weekly email stats," "write my channel digest," "what happened with
  email/push this week," "WoW email performance," "weekly push summary," "write
  my Slack/Notion channel update," or pastes ESP/push exports and asks for a
  summary.
---

# Weekly Email & Push Metrics Digest

Raw ESP and push exports in → a clear, decision-ready weekly status in two minutes. Not a
spreadsheet recitation — a narrative that surfaces what moved, why it matters, and one
thing to do about it. Built on the DELTA framework: Deltas (WoW changes), Exceptions
(standout sends), Levers (what drove it), Trends (3-week direction), Action (one
prioritized next step).

Every digest is calibrated to the brand's own baselines, not generic "industry benchmarks"
that never match your list.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's voice, ICP, offer, and
  any list-health baselines stored in `brand.md` or companion files. The digest uses the
  brand's baseline open/click/push CTR rather than industry averages; voice governs the
  tone of the narrative and Slack ping.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active`
  and that brand's `brand.md` directly; if none exists, ask the user for (1) brand name,
  (2) their current baseline open rate, click rate, push CTR, and unsubscribe rate, and
  (3) preferred digest format (Slack or Notion) before proceeding.

- **`email-a-b-test-planner-performance-analyzer`** (optional) — if an A/B test ran during
  the week, hand off the variant data for a deeper significance read rather than doing it
  inline here.

- **`lifecycle-email-push-copy-reviewer`** (optional) — when the top-performing send is
  flagged for template extraction, pass its copy here for a structured breakdown.

- **`weekly-wins-anomalies-slack-ping`** (optional) — when the user wants a compressed
  one-paragraph Slack alert (not the full digest), compose the full digest first, then
  extract the single-paragraph version via this skill.

- **`send-time-cadence-recommender`** (optional) — if a cadence anomaly (too many sends,
  fatigue signals in unsubscribe rate) surfaces, call this skill for a structured
  recommendation.

---

## How a run works

```
Step 0  Load the brand            ──► brand-brain (voice + baselines)
Step 1  Parse the inputs           ──► normalize stats from any ESP / push format
Step 2  Apply the DELTA framework  ──► compute deltas, flag exceptions, read trend
Step 3  Draft the digest           ──► channel-correct format (Slack / Notion / email)
Step 4  Self-review                ──► checklist before presenting
Step 5  Save artifact              ──► ./email-push-reports/[brand]-[YYYY-WW].md
```

---

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`), passing the user's
request and any named brand. Use the returned digest for:

- **Baselines:** brand-stored open rate, CTOR, push CTR, unsubscribe rate. If missing,
  ask the user once ("What are your current baseline open and click rates?") and note them
  as `[user-stated, unverified]`.
- **Voice:** tone adjectives and banned words govern the narrative and Slack message.
- **ICP / lifecycle stage:** shapes which metrics deserve emphasis (e.g., eCommerce brand
  emphasizes revenue per recipient; SaaS emphasizes CTOR and unsubscribe rate).

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active`
and that brand's `brand.md` directly; if none exists, ask the user for (1) brand name,
(2) their current baseline open rate, click rate, push CTR, and unsubscribe rate, and
(3) preferred digest format (Slack or Notion) before proceeding.

---

### Step 1 — Parse the inputs

Accept any of these formats (raw paste, CSV, screenshot, or manual numbers):

| Channel | Minimum viable stats |
|---|---|
| Email | Sends, opens (or open rate), clicks (or CTR/CTOR), unsubscribes, revenue (if available) |
| Push | Impressions or reach, clicks (CTR), direct opens, opt-outs (if available) |
| Both | Accept each channel's stats separately; compute a combined engagement summary |

Normalize the data: if given raw counts, compute rates. If given rates without counts,
note it — averages across unequal-size sends will be marked `[weighted avg not possible]`.

Ask for prior-week numbers if not supplied — the delta is the product's whole value.
If prior-week data is unavailable, note it and present current-week values only.

---

### Step 2 — Apply the DELTA framework

**D — Deltas (WoW changes)**
Compute absolute and percentage WoW for every metric. Flag significance:
- Green (within ±10% of baseline): normal variation, no action required.
- Amber (±10–25% of baseline): worth watching, note probable cause.
- Red (>±25% of baseline, or unsubscribe rate spike >0.5% for any single send): flag
  prominently with a probable-cause hypothesis.

**E — Exceptions (standout sends)**
Identify the single best-performing send (highest CTOR or CTR, or revenue if available)
and the single worst. State what set them apart: subject line structure, send time, segment,
offer type, or content format — not just "the subject line was good."

**L — Levers (what drove it)**
Name up to three contributing factors: a new segment, a different cadence, a campaign
theme, a behaviorally triggered message, a list quality event (import, prune, or churn).
Distinguish correlation from confirmed causation. Do not invent causes; if unclear, say so.

**T — Trend (3-week direction)**
If prior digests exist in `./email-push-reports/`, load the last two weeks and state the
3-week direction per metric: improving / stable / declining. A single week is not a trend.
If this is the first run, note "baseline week — trend available next week."

**A — Action (one prioritized next step)**
One thing, not a menu. Ranked by expected impact vs. effort:
- If a critical metric is in the Red zone → surface that as the action.
- If everything is Green → extract the winning send's angle for a follow-on test.
- If trend is declining for 2+ weeks → recommend a cadence or segmentation review (call
  `send-time-cadence-recommender` or `segmentation-rfm-strategy-builder` inline).

---

### Step 3 — Draft the digest

Two formats available; ask if unclear.

#### Slack format (default for quick async)

```
*[Brand] Email + Push — Week of [Mon DD MMM]*

*Email* | [N] sends · [OR]% open · [CTOR]% CTOR · [UNS]% unsub
*Push*  | [N] sends · [CTR]% CTR · [OPT]% opt-out
WoW: open [+/-X pts] · CTOR [+/-X pts] · push CTR [+/-X pts]

*This week's standout:* "[Subject line or push title]" — [CTOR]% CTOR
([what set it apart — one clause])

*Watch:* [amber/red flag with one-sentence cause hypothesis, or "all green"]
*Action:* [one sentence — what to do next week and why]
```

Keep it to 200 words or fewer. No markdown tables in Slack format (they render as plain
text). Use `*bold*` for Slack-native emphasis.

#### Notion / doc format (for archived digests or stakeholder reporting)

A structured document with:
1. **Summary line** — one sentence with the week's headline metric and verdict.
2. **Email performance table** — sends, OR, CTOR, CTR, revenue (if available), unsub rate,
   WoW delta per metric, zone flag (Green/Amber/Red).
3. **Push performance table** — same structure.
4. **Standout send** — subject line / title, segment, OR/CTOR/CTR, what made it work.
5. **Trend** — 3-week sparkline (text: ↑ ↓ →) per core metric.
6. **Action item** — bolded, with owner and due date if the user provides them.

The Notion format may also include a 2–3 sentence narrative expanding on the DELTA read
for stakeholders who won't parse a table.

---

### Step 4 — Self-review

Before presenting, check:
- Brand-brain called and baselines / voice loaded (or fallback executed)?
- WoW deltas computed; every metric with a Red flag has a probable-cause hypothesis?
- Standout send identified with a structural reason, not a vague compliment?
- Action is one specific thing, not a list of options?
- Slack format is under 200 words; no raw table markup in Slack output?
- Unconfirmed causes or benchmarks marked `[verify]` or `[user-stated]`?

---

### Step 5 — Save the artifact

Save to `./email-push-reports/[brand-slug]-[YYYY-WW].md` (ISO week number, e.g.,
`pushengage-2026-W25.md`). On the next run, load prior weeks from this folder for the
trend read. If the directory does not exist, create it. Never overwrite an existing week's
file — append `_v2` if re-run on the same week.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No digest before the brand context loads. Its baselines override
  industry averages; its voice governs every word of the narrative.
- **Deltas, not snapshots.** A metric without WoW context is close to meaningless. If
  prior data is absent, say so explicitly and ask for it.
- **One action.** A digest that ends in a five-item to-do list is a report, not a decision
  tool. Name the single highest-leverage next step.
- **Cause discipline.** State probable cause for any anomaly; distinguish "likely because"
  from "confirmed because." Never invent causation.
- **Truth only.** Real numbers from the export. If a number looks wrong (e.g., 0.0% open
  rate on a 10k send), flag it as a possible data-export error before including it.
- **Channel parity.** Email and push are peers in this digest. Do not omit push metrics
  because email is the primary channel — push CTR trend is often the leading indicator.

## What Not to Do

- Do not compare to "industry average" benchmarks unless the brand's `brand.md` explicitly
  stores them as a target — use the brand's own baselines.
- Do not list every metric in the Slack format. The table belongs in Notion; Slack gets the
  headline number and the delta.
- Do not generate a digest without at least one channel's data. If the user hasn't pasted
  stats, ask for them.
- Do not call it a "good week" or "strong performance" without a number. Vague positivity
  corrodes trust in the digest.
- Do not overwrite a previously saved artifact. Append `_v2`.
- Do not call `email-a-b-test-planner-performance-analyzer` when there was no A/B test;
  the inline DELTA read is sufficient.

## Quality Checklist

- Brand-brain called; baselines and voice loaded (or fallback completed)?
- WoW delta computed for every reported metric; zone flags (Green/Amber/Red) assigned?
- At least one probable-cause hypothesis for every Amber or Red metric?
- Standout send named with a structural insight (segment, timing, subject structure,
  offer mechanic) — not just "high open rate"?
- Trend direction stated (or "baseline week" noted)?
- One Action item, not a menu — specific, with a lever it targets?
- Format correct: Slack under 200 words with no table markup; Notion has structured tables?
- Artifact saved to `./email-push-reports/[slug]-[YYYY-WW].md`?
