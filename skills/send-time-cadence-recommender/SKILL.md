---
name: send-time-cadence-recommender
description: >
  Turns audience timezone data + historical open/click CSVs into concrete send-window recommendations
  and an optimal cadence schedule that reduces list fatigue without sacrificing revenue. Two modes:
  Window Mode diagnoses the best send time per segment (hour-of-day × day-of-week heatmap, with a
  single recommended slot per segment); Cadence Mode determines the right message frequency —
  emails per week, push per day, gap rules — so you're not bleeding unsubscribes and suppression
  rates. Uses the Circadian Open-Rate (COR) framework: map engagement to biological rhythm + job
  context + device, not just raw peak clicks. Works for email, push notification, and SMS/WhatsApp
  channels. Calls brand-brain to load ICP, channel mix, and any cadence preferences or suppression
  rules already captured. Outputs a segment-keyed recommendation table and, on request, a ready-to-
  import send-schedule CSV. Use when the user says "best time to send," "when should I send," "how
  often should I email," "send frequency," "cadence optimization," "reduce unsubscribes," "open rate
  by time," "push timing," or pastes an ESP/platform CSV and asks what to do with it.
---

# Send-Time & Cadence Recommender

Every message has a right moment. This skill finds it — per segment, per channel — using the Circadian Open-Rate (COR) framework instead of the cargo-cult advice ("send Tuesday 10 AM") that every sender follows and therefore destroys.

Two jobs, one skill:
- **Window Mode** — best send slot (day × hour) for each segment, derived from your own engagement data.
- **Cadence Mode** — right frequency and gap rules to maximize LTV engagement without burning the list.

Neither job is guesswork. If your data says Thursday 7 PM wins for dormant users, that's what goes in the table — with a stated rationale, not a canned recommendation.

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads the active brand's ICP, channel mix, historical cadence context, any stated suppression rules, and banned practices already on record. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for (1) channel(s) in scope, (2) ICP timezone spread, (3) any hard suppression windows (quiet hours, compliance blackouts), and (4) current send frequency, before proceeding.
- **`lifecycle-email-push-copy-reviewer`** *(optional)* — after Windows + Cadence are set, pass any newly-scheduled sequence through it to confirm copy still lands given the new timing context (e.g. a "morning coffee" hook misfires if the send shifts to 9 PM).
- **`email-compliance-auditor-gdpr-can-spam`** *(optional)* — if quiet-hours or consent-window rules are in scope, route the cadence plan through compliance audit before shipping.
- **`bulk-scheduling-csv-builder`** *(optional)* — converts the approved send-schedule table into a platform-correct bulk-upload CSV for Buffer, Hootsuite, Later, or Klaviyo.
- **`weekly-email-push-metrics-digest`** *(optional)* — post-send, cross-reference actual performance against the recommended windows to close the feedback loop.

---

## How a run works

```
Step 0  Load brand context ──► brand-brain (always)
Step 1  Determine mode     ──► Window / Cadence / Both
Step 2  Parse inputs       ──► timezone data, historical CSV, or stated estimates
Step 3  Apply COR framework ──► segment × channel heatmap + fatigue index
Step 4  Output             ──► recommendation table + rationale
Step 5  (Optional) export  ──► send-schedule CSV on request
```

### Step 0 — Load brand context (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's digest, including ICP job-function + timezone spread, primary channel stack (email / push / SMS), any suppression or quiet-hours rules previously captured, and current send frequency if on record.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for (1) channel(s) in scope, (2) ICP timezone spread, (3) any hard suppression windows, and (4) current send frequency before proceeding.

---

## The COR Framework (Circadian Open-Rate)

Most "best send time" advice averages across the entire internet and ignores three variables that actually drive opens. COR builds around all three:

### Variable 1 — Biological rhythm × device context

| Rhythm window | Typical behavior | Device | Best for |
|---|---|---|---|
| Early wake (5–7 AM) | Triage scan, mobile | Phone | Push alerts, flash sales, news digests |
| Commute (7–9 AM) | Curated reading, mobile | Phone | Newsletters, event invites |
| Desk focus (9–11 AM) | Active work, desktop | Laptop | B2B emails, SaaS lifecycle triggers |
| Post-lunch (12–1 PM) | Casual browse, mixed | Mixed | Consumer, e-commerce, low-commitment offers |
| Mid-afternoon slump (2–4 PM) | Distraction-seeking | Mixed | Curiosity hooks, content, community |
| Wind-down (6–8 PM) | Personal time, mobile | Phone/tablet | B2C, entertainment, personal finance |
| Late evening (8–10 PM) | Impulse decisions | Phone | Flash offers, re-engagement, abandoned cart |

ICP job function modulates this: a nurse reads at 6 AM, a lawyer at 8 PM, a developer at 2 AM. Load from brand-brain; ask only if absent.

### Variable 2 — Channel-specific ceiling

Each channel has a tolerance ceiling before suppression events spike:

| Channel | Typical fatigue threshold | Hard ceiling rule |
|---|---|---|
| Email (transactional) | Unlimited | Not subject to marketing cadence |
| Email (marketing) | 1–3/week for most ICPs | >4/week → unsubscribe rate climbs non-linearly [verify for your list] |
| Web push notification | 1–2/day max; 3–5/week sustainable | >2/day without segmentation → opt-out event |
| SMS / WhatsApp | 2–4/month marketing | TCPA/DPA compliance gates before cadence; route to `email-compliance-auditor-gdpr-can-spam` |
| In-app message | Context-triggered; no fixed frequency | Over-prompting → suppression or churn |

### Variable 3 — Segment engagement lifecycle stage

Not all subscribers are equal. Apply stage-appropriate windows and cadence:

| Stage | Definition | Recommended approach |
|---|---|---|
| Hot (0–30 days, active) | Opening regularly, clicking | Maintain; don't over-send; preserve goodwill |
| Warm (31–90 days) | Occasional engagement | Reduce frequency slightly; optimize window |
| Cool (91–180 days) | Low open rate | Cut to 1x/week; shift to highest-performing window |
| Cold (180+ days, dormant) | Near-zero engagement | Sunset path or `win-back-re-engagement-campaign-builder` |
| New subscriber (0–7 days) | Honeymoon window | Higher frequency acceptable; use welcome sequence |

---

## Window Mode — best send slot per segment

**Input needed (in priority order):**
1. Historical ESP/push export: open timestamp (UTC preferred) + subscriber timezone + segment tag. Even 4 weeks of data produces usable signal.
2. If no export: subscriber timezone distribution + your ICP job context (loaded from brand-brain or stated).
3. If pure greenfield (no data, no history): use COR biological defaults for the ICP, flagged as `[estimated — validate after 4 sends]`.

**Output format:**

```
## Send-Window Recommendations — [Brand] [Channel]
Data basis: [n sends / n weeks / estimated]

| Segment | Size | Top window (day × hour, local TZ) | Runner-up | Avoid | Confidence |
|---|---|---|---|---|---|
| Hot subscribers | 2,400 | Tue 10 AM | Thu 9 AM | Fri PM, weekends | High |
| Warm subscribers | 8,100 | Thu 8 AM | Mon 10 AM | Mon–Wed PM | Medium |
| New (0–7 days) | 310 | Within 5 min of signup | Tue 10 AM | — | n/a |
| Dormant (180d+) | 1,200 | Thu 7 PM | Sat 9 AM | Weekday AM | Low — test first |

Rationale: [2–3 sentences per segment explaining the COR logic or data signal behind the pick]
```

If the data spans <3 weeks or <500 sends per segment, flag confidence as Low and recommend a 2-slot A/B test using the `email-a-b-test-planner-performance-analyzer` sibling before committing.

---

## Cadence Mode — frequency and gap rules

**Input needed:**
1. Current send frequency (from brand-brain or stated).
2. Unsubscribe rate, complaint rate, or suppression rate trend (if available).
3. Revenue events (sales, product launches, seasonal peaks) — cadence should flex around these without burning baseline.

**Output format:**

```
## Cadence Recommendation — [Brand] [Channel]

Baseline: [X sends/week] → Recommended: [Y sends/week]
Rationale: [1–2 sentences — what signal drove the change]

Gap rules:
- Minimum gap between any two sends: [X hours]
- Hard quiet window: [e.g. Sat 9 PM – Sun 9 AM unless transactional]
- Burst exception: [e.g. up to 3 sends/week for flash sales, max 2 consecutive days]
- Sunset trigger: suppress contacts with 0 opens / 0 clicks in [X days] → route to win-back

Fatigue index (current): [Low / Medium / High / Critical]
Projected unsubscribe impact of current cadence: [directional — cite your data or mark [verify]]

Seasonal flex schedule:
[Month / event → adjusted cadence]
```

---

## Both modes (most runs)

Run Window then Cadence. Present them together. Finish with a consolidated send-schedule table:

```
## Consolidated Send Schedule — [Brand slug]

| Segment | Channel | Send day | Send time (local TZ) | Frequency | Next review |
|---|---|---|---|---|---|
```

Save to `./send-schedule/[brand-slug]-send-schedule.md` if the user asks for a persistent artifact. Pass to `bulk-scheduling-csv-builder` to generate the importable CSV.

---

## Data inputs and quality gates

Before accepting a CSV, run these gates — flag and halt if breached:

- **Timezone normalization:** all timestamps must be converted to local subscriber time before analysis. UTC-only CSVs without timezone offset produce misleading peaks (e.g. apparent 3 AM "winners" for a US East audience are really 11 AM UTC sends).
- **Volume floor:** fewer than 200 opens per segment → confidence is Low; recommend holding at COR defaults until volume builds.
- **Survivorship bias check:** if the CSV contains only opens (not all sends), peak hours are inflated by the opens that happened — not a reliable sample of when sending works best. Flag explicitly.
- **Bot-open contamination:** Apple MPP and corporate mail scanners inflate morning open spikes for B2B lists. If B2B ICP and MPP-inflation is suspected, weight click data over open data.

---

## Principles

- **Your data beats everyone else's benchmarks.** Industry averages are a starting point, never a destination. If your audience opens at 9 PM Thursday, ship at 9 PM Thursday — and note it.
- **Segment before you optimize.** A single "best time" for the whole list is almost always wrong. Dormant users, hot users, and new subscribers have different rhythms. Build the table.
- **Fatigue is compounding.** One extra send per week feels small. Over 90 days it's 13 extra sends — and unsubscribes are not linear. Model the trend, don't just count the rate.
- **Compliance constraints are hard floors.** Quiet hours, consent windows, and TCPA/GDPR blackouts are not suggestions. Route to `email-compliance-auditor-gdpr-can-spam` when in scope; never guess jurisdiction rules.
- **Greenfield estimates are hypotheses.** Label them `[estimated — validate after 4 sends]`. Close the loop with `weekly-email-push-metrics-digest`.
- **Never fabricate engagement data.** If the user has no historical data, use COR defaults and say so. Do not invent open rates or claim a slot "typically" performs X% better without a cited source.

---

## What not to do

- Don't default to "Tuesday 10 AM" without data — that is precisely what everyone else sends, which is why it's degrading.
- Don't produce a single universal send time for a heterogeneous list; always segment.
- Don't run cadence analysis without checking compliance constraints first for SMS/WhatsApp channels.
- Don't accept a UTC-only CSV without flagging the timezone normalization requirement.
- Don't treat Apple MPP-inflated open spikes as signal for B2B ICPs — weight clicks.
- Don't recommend cadence increases during high-fatigue-index periods; recommend a cooldown first.
- Don't persist files inside the skill folder — save all output artifacts to `./send-schedule/` in the user's CWD.

---

## Quality checklist (self-review before presenting)

- [ ] `brand-brain` called first; ICP timezone spread and channel mix loaded (or fallback executed)?
- [ ] Mode(s) determined and executed (Window / Cadence / Both)?
- [ ] COR framework applied — biological rhythm, channel ceiling, and segment lifecycle stage all addressed?
- [ ] Data quality gates run — timezone normalization confirmed, volume floor checked, survivorship bias flagged if applicable, MPP contamination flagged for B2B?
- [ ] Output is segment-keyed, not a single aggregate recommendation?
- [ ] Confidence levels stated; greenfield estimates labeled `[estimated — validate after 4 sends]`?
- [ ] Cadence recommendation includes gap rules, quiet window, burst exception, and sunset trigger?
- [ ] Compliance scope confirmed; SMS/WhatsApp routed to `email-compliance-auditor-gdpr-can-spam` if applicable?
- [ ] Consolidated send-schedule table complete; saved to `./send-schedule/` or passed to `bulk-scheduling-csv-builder` if requested?
- [ ] No invented benchmark numbers; every external claim either cited or marked `[verify]`?
