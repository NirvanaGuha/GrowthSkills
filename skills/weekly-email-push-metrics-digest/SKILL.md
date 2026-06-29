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
thing to do about it. Built on DELTA, a house mnemonic we use here (not an established
model): Deltas (WoW changes), Exceptions (standout sends), Levers (what drove it),
Trends (3-week direction), Action (one prioritized next step).

The job of a digest is to suppress the noise so the one real signal gets seen. The
significance gate in Step 2 is therefore the core engine: every WoW delta is filtered
through **both** a % band **and** a volume / absolute-count floor — because a rate is a
proportion and its week-to-week noise scales with √(p(1-p)/n), so a 200-send segment and a
40k blast that show the same %delta are not the same news. Small sends get labeled
`[low volume — not significant]` instead of triggering a false alarm. Email and push each
get their own bands, because push CTR runs lower and noisier on a different denominator.

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

- **`sample-size-calculator`** (optional, Step 2) — when a WoW swing sits right on a band
  edge and you need a real p-value or the minimum n that would make the move significant,
  hand it the two weeks' counts rather than guessing. The Step 2 bands are the fast gate;
  this is the precise read when the fast gate is ambiguous.

---

## How a run works

```
Step 0  Load the brand            ──► brand-brain (voice + baselines)
Step 1  Parse the inputs           ──► normalize stats from any ESP / push format
Step 2  Apply the DELTA mnemonic   ──► compute deltas, flag exceptions, read trend
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

Normalize the data: if given raw counts, compute rates. **Always carry the recipient
count (n) and the engagement event counts alongside every rate** — Step 2's significance
gate needs them, and a rate handed over with no n cannot be flagged Amber/Red at all (it
defaults to `[low volume — not significant]` until the count is supplied). If given rates
without counts, note it and ask for the denominators; averages across unequal-size sends
are separately marked `[weighted avg not possible]`.

Ask for prior-week numbers if not supplied — the delta is the product's whole value.
If prior-week data is unavailable, note it and present current-week values only.

---

### Step 2 — Apply the DELTA mnemonic

**D — Deltas (WoW changes)**
Compute absolute and percentage WoW for every metric, then gate each delta through the
significance bands below **before** flagging anything.

**Why the gate exists.** A rate is a proportion, and every proportion carries sampling
error. The noise in an observed rate scales with √(p(1-p)/n): the smaller the send (n), the
wider the swing you'll see week to week with nothing actually changing. A flat ±% cutoff
treats a 200-send segment and a 40k blast as equally trustworthy — they are not. On 200
sends, a ±10–15% open-rate move is well inside normal sampling noise; on 40k, a 3% move is
real. So every flag is gated on **both** the % delta **and** the send volume / absolute
count behind it. (This is plain sampling-error logic, not a named framework. For a precise
p-value or required-n when a swing sits on the line, hand the two weeks' counts to the
`sample-size-calculator` sibling skill.)

**Significance bands (gate on volume first, then %):**

| Recipients (n) for the metric | Treat the metric as… | Band logic |
|---|---|---|
| **< 1,000** | low-volume | Label every WoW swing `[low volume — not significant]`. Never assign Red/Amber on % alone. The *only* flag allowed here is an absolute-count trigger (see unsub/opt-out rule below). |
| **1,000–9,999** | normal | Green ≤ ±10% · Amber ±10–25% · Red > ±25% — **and** require the absolute change to clear the floor in the band table below before flagging Amber/Red. |
| **≥ 10,000** | high-volume | Green ≤ ±7% · Amber ±7–20% · Red > ±20%. Larger n means smaller real moves matter; tighten the bands. |

**Absolute-count floor (must clear alongside the %):** a flag also needs a real-count move,
not just a ratio move. Require the engagement event count (opens, clicks) to shift by **≥ 30
events** before an Amber/Red fires on a rate. Below that, label `[low volume — not
significant]` regardless of the percentage.

**Unsubscribe / opt-out spike — dual trigger (count AND rate):** flag Red only when the
send clears **both** `> 0.5% of recipients` **and** `> 5 absolute unsubscribes`. A single
extra unsubscribe on a 200-person segment is 0.5%+ but is one human clicking once — it
never reads Red. On a 30k send, 0.5% is 150 people leaving and is a genuine Red.

**Push gets its own bands (channel parity in the mechanics, not just the principle).** Push
CTR runs lower and noisier than email, and the relevant volume is *delivered* notifications
(reach), not subscribers on file. Apply this row to every push metric the same way the
email table gates email:

| Push metric | Volume gate (delivered / reach) | Green | Amber | Red |
|---|---|---|---|---|
| **Push CTR** | < 2,000 delivered → `[low volume — not significant]`; flag on % only at ≥ 2,000 | ≤ ±15% | ±15–30% | > ±30% |
| **Click count floor** | — | — | — | needs **≥ 20 absolute clicks** moved to fire Amber/Red |
| **Opt-out spike** (push analog of the email unsub rule) | — | — | — | Red only when **> 0.3% of delivered AND > 10 absolute opt-outs** |

Push thresholds sit wider than email's on purpose: a baseline push CTR of ~3–5% on a few
thousand deliveries swings more in raw % terms than a 25–40% open rate on a large list, so
the same noise produces a bigger ratio. The opt-out gate is tighter on rate (0.3% vs 0.5%)
because a push opt-out is a harder, more deliberate exit than an email unsubscribe — but it
still requires a real absolute count so a 500-delivery test never reads Red on two taps.

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
2. **Email performance table** — sends (n), OR, CTOR, CTR, revenue (if available), unsub
   rate, WoW delta per metric, zone flag (Green / Amber / Red / `low volume`). Show n so the
   reader can see why a sub-1,000 send wasn't flagged.
3. **Push performance table** — same structure, on push's own bands (delivered/reach as n).
4. **Standout send** — subject line / title, segment, OR/CTOR/CTR, what made it work.
5. **Trend** — 3-week sparkline (text: ↑ ↓ →) per core metric.
6. **Action item** — bolded, with owner and due date if the user provides them.

The Notion format may also include a 2–3 sentence narrative expanding on the DELTA read
for stakeholders who won't parse a table.

---

### Step 4 — Self-review

Before presenting, check:
- Brand-brain called and baselines / voice loaded (or fallback executed)?
- WoW deltas computed; **every flag gated through the volume + absolute-count floor** before
  it was assigned a band — no Amber/Red on a sub-1,000 send or a sub-30-event move?
- Every sub-threshold swing labeled `[low volume — not significant]` rather than dropped or
  flagged?
- Push metrics scored on push's own bands (not the email cutoffs); push opt-out gated on
  count AND rate?
- Every metric with a Red flag has a probable-cause hypothesis?
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
- **Volume before %.** Never flag a delta on percentage alone. A rate is a proportion; its
  noise scales with √(p(1-p)/n). A swing on a small send is sampling noise until proven
  otherwise — gate every flag on the recipient count and the absolute event move, and label
  sub-threshold swings `[low volume — not significant]`. A false Red is worse than a missed
  Amber: it spends a reader's trust on nothing.
- **Each channel on its own bands.** Push CTR is a different metric on a different
  denominator than email opens — it gets push thresholds, not email's. Parity means equal
  rigor, not identical numbers.
- **One action.** A digest that ends in a five-item to-do list is a report, not a decision
  tool. Name the single highest-leverage next step.
- **Cause discipline.** State probable cause for any anomaly; distinguish "likely because"
  from "confirmed because." Never invent causation.
- **Truth only.** Real numbers from the export. If a number looks wrong (e.g., 0.0% open
  rate on a 10k send), flag it as a possible data-export error before including it.
- **Channel parity.** Email and push are peers in this digest. Do not omit push metrics
  because email is the primary channel — push CTR trend is often the leading indicator. Parity
  is enforced in Step 2's mechanics (push has its own bands and opt-out gate), not just
  asserted here.

## What Not to Do

- Do not compare to "industry average" benchmarks unless the brand's `brand.md` explicitly
  stores them as a target — use the brand's own baselines.
- Do not list every metric in the Slack format. The table belongs in Notion; Slack gets the
  headline number and the delta.
- Do not generate a digest without at least one channel's data. If the user hasn't pasted
  stats, ask for them.
- Do not flag Red/Amber on a small send. A 200-recipient segment moving 20% is noise, not
  news — label it `[low volume — not significant]`. Do not fire the unsub/opt-out alarm on
  one or two extra unsubscribes off a tiny list; it needs the absolute-count floor too.
- Do not score push on email's cutoffs. Push CTR runs lower and noisier — use the push bands.
- Do not call it a "good week" or "strong performance" without a number. Vague positivity
  corrodes trust in the digest.
- Do not overwrite a previously saved artifact. Append `_v2`.
- Do not call `email-a-b-test-planner-performance-analyzer` when there was no A/B test;
  the inline DELTA read is sufficient.

## Quality Checklist

- Brand-brain called; baselines and voice loaded (or fallback completed)?
- WoW delta computed for every reported metric; each delta gated on **volume + absolute
  count** before a band was assigned (Green / Amber / Red / `low volume`)?
- Sends under the volume floor labeled `[low volume — not significant]`, not flagged?
- Unsub/opt-out flags cleared **both** the rate **and** the absolute-count trigger?
- Push metrics scored on push's own bands, with recipient/reach n shown in the table?
- At least one probable-cause hypothesis for every Amber or Red metric?
- Standout send named with a structural insight (segment, timing, subject structure,
  offer mechanic) — not just "high open rate"?
- Trend direction stated (or "baseline week" noted)?
- One Action item, not a menu — specific, with a lever it targets?
- Format correct: Slack under 200 words with no table markup; Notion has structured tables?
- Artifact saved to `./email-push-reports/[slug]-[YYYY-WW].md`?
