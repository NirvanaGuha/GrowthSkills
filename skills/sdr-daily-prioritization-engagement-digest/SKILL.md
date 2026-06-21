---
name: sdr-daily-prioritization-engagement-digest
description: >
  Turns raw CRM activity, intent signals, and engagement data into a ranked daily action list and a
  weekly account engagement digest that an SDR or BDR can act on without opening five tabs. Applies
  the PACT prioritization model (Pipeline fit, Activity recency, Conversion signals, Timing/urgency)
  to score and sort every prospect and account in the working set, then formats a clean daily digest
  (top-N accounts to touch, suggested next action per account, and the signal that triggered it) plus
  a weekly roll-up suitable for a manager 1:1 or team standup. The skill composes `intent-signal-
  summarizer` for intent data interpretation, `data-qa-measurement-gotcha-checker` for CRM data
  quality gates, `account-dossier-builder` for thin-file accounts that need depth, and `deal-
  communication-pack` for open opportunities. It does NOT draft outreach copy itself — it decides
  WHAT to touch and WHY; it hands the why to the SDR so their outreach is purposeful, not robotic.
  Use when an SDR says "prioritize my book today," "which accounts should I work first," "build my
  daily hit list," "weekly digest of my accounts," "who's showing intent," "where should I spend the
  next two hours," or pastes a CRM export/activity report and asks what to do with it.
---

# SDR Daily Prioritization & Engagement Digest

An SDR's biggest productivity killer is not slow typing — it is the ten minutes per account spent deciding whether to touch it and how. This skill eliminates that decision cost. It takes your raw CRM activity, intent signals, and engagement data, runs them through the PACT scoring model, and hands back a ranked action list you can work from the top down.

Two outputs on every run: a **Daily Digest** (ranked, opinionated, immediately actionable) and, on request or weekly cadence, a **Weekly Account Engagement Digest** suited for manager syncs or pipeline reviews.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's ICP, offer mechanics, positioning, and proof before any scoring or messaging guidance is given. PACT scoring is meaningless without knowing who the ideal account looks like.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for ICP definition (firmographics, titles, pains), offer mechanics (free trial / demo / direct), and primary conversion signals before proceeding.

- **`data-qa-measurement-gotcha-checker`** (required before scoring) — gates the CRM export on data quality. Stale last-contact dates, duplicate contacts, or null engagement fields corrupt PACT scores silently; this skill surfaces them first.

- **`intent-signal-summarizer`** (call when intent data is present) — interprets Bombora/G2/6sense surge data into plain-English account summaries. Do not re-derive intent logic here; consume the output.

- **`account-dossier-builder`** (call for thin-file accounts, i.e., fewer than 2 meaningful touches) — builds enough context on a cold/new account to propose a credible first action.

- **`deal-communication-pack`** (call when open opportunities appear in the working set) — produces the Slack ping, call debrief notes, or email reply classification for active deals. Hand off; do not duplicate.

---

## How a run works

```
Step 0   Load brand context  ──► brand-brain (ICP + offer + proof)
Step 1   Ingest & clean      ──► data-qa gate; flag bad records
Step 2   Score               ──► PACT model per account/prospect
Step 3   Route               ──► thin-file? → account-dossier-builder
                                  intent surge? → intent-signal-summarizer
                                  open opp? → deal-communication-pack
Step 4   Rank & emit         ──► Daily Digest (top-N action list)
Step 5   Weekly roll-up      ──► on request or weekly trigger
```

---

## The PACT Scoring Model

PACT scores each account on four dimensions (0–3 per dimension, max 12). It is deliberately lightweight — designed to run against CRM exports without a Salesforce admin.

### P — Pipeline Fit (0–3)
How closely does the account match the brand's ICP?

| Score | Signal |
|---|---|
| 3 | Firmographics, title, and pain match ICP; in TAM; product signals present |
| 2 | 2 of 3 match; minor gap (size off by one band, secondary pain) |
| 1 | Weak match; in adjacent segment; worth nurturing but not sprinting |
| 0 | Clear mismatch; disqualify and note in CRM |

Source: brand-brain ICP digest + CRM company fields.

### A — Activity Recency (0–3)
How recently has there been meaningful two-way engagement?

| Score | Signal |
|---|---|
| 3 | Meaningful engagement (reply, demo booked, content click) in last 7 days |
| 2 | Engagement within 8–21 days (warm) |
| 1 | Last touch 22–45 days ago (cooling) |
| 0 | No activity or last touch >45 days (cold; run a re-engagement before standard sequence) |

### C — Conversion Signals (0–3)
Evidence the account is evaluating or moving toward a decision.

| Score | Signal |
|---|---|
| 3 | Intent surge (G2/Bombora) + pricing page/demo page visit + champion identified |
| 2 | One strong signal: pricing visit OR intent surge OR champion reply |
| 1 | Soft signals: content downloads, webinar attendance, job change at account |
| 0 | No signals; suppress until a trigger fires |

### T — Timing & Urgency (0–3)
External or internal forcing function creating a near-term window.

| Score | Signal |
|---|---|
| 3 | Contract renewal in <30 days, announced budget cycle, event attendance confirmed |
| 2 | Seasonal/quarter-end pressure, competitor contract rumored, hiring surge |
| 1 | Vague timing cue (noted "evaluating Q3"), no confirmed window |
| 0 | No timing signal |

**PACT score** = P + A + C + T. Sort descending. Tie-break: highest C first (conversion signals are the most durable), then highest T.

---

## Daily Digest format

```
## SDR Daily Digest — [Date]
Brand: [slug, via brand-brain]
Working set: [N accounts] | Scored: [N] | Data-quality flags: [N — see appended]

### Today's priority accounts (PACT ≥ 7)
| Rank | Account | PACT | Top signal | Suggested next action | Owner |
|------|---------|------|-----------|----------------------|-------|
|  1   | Acme Co | 10   | Pricing page x3 yesterday | Send proof-point email referencing their open cart-abandonment pain | [SDR] |
...

### Warm accounts to progress (PACT 5–6)
[Same table, 3–6 rows max]

### Watch list: signals without fit (PACT 3–4)
[1–2 line per account; flag what would move it up]

### Accounts removed / disqualified today
[Account | reason | recommended CRM disposition]

---
### Data-quality flags (from data-qa-measurement-gotcha-checker)
[Stale dates, duplicate contacts, missing fields — with CRM fix instructions]
```

Keep the Daily Digest to one screen. If the working set exceeds 30 accounts, surface only PACT ≥ 6 in the main table and attach a full scored list as an artifact saved to `./sdr-digests/[YYYY-MM-DD]-daily.md`.

---

## Weekly Account Engagement Digest format

Triggered by "weekly digest," "manager sync," "week recap," or a weekly cadence. Covers the full book.

```
## SDR Weekly Engagement Digest — Week of [Date]
Brand: [slug] | SDR: [name if provided]

### Engagement summary
- Accounts touched: [N] / [total book]
- Total touches: [N] (calls: N | emails: N | LinkedIn: N | other: N)
- Accounts with ≥1 positive reply: [N] ([%] of touched)
- New accounts added to book: [N]
- Accounts disqualified: [N]

### Top movers this week (accounts that jumped ≥2 PACT points)
[Account | old score → new score | what changed]

### Stalled accounts (PACT unchanged 2+ weeks; flag for review)
[Account | weeks stalled | recommended action: nurture / re-assign / disqualify]

### Open opportunities needing SDR support (from deal-communication-pack)
[Deal | stage | last SDR touch | recommended action]

### Pipeline impact
[Meetings booked: N | SALs created: N | Opps influenced: N | value if tracked]

### This week's focus for next week (top 5 by PACT delta potential)
[Account | current score | signal to act on | recommended first move]
```

Save to `./sdr-digests/[YYYY-MM-DD]-weekly.md`.

---

## Running without a CRM export (verbal input mode)

When the SDR describes their book verbally ("I have about 20 accounts, here are the hot ones…"), apply PACT as a structured interview: ask for P, A, C, T evidence per account in batches of 3–5, score collaboratively, and produce the same Digest. Note which scores are SDR-estimated (`[est]`) vs. data-backed.

---

## Principles

- **Score before recommending.** Never suggest an action without a PACT score anchoring it. Gut feel without evidence is not a strategy.
- **The next action must be specific.** Not "follow up" — "send the [ROI calculator / case study / competitive one-pager] referencing [specific signal]."
- **Brand-brain first.** ICP fit (the P in PACT) is meaningless without loading the brand's actual ICP. No scoring before Step 0 completes.
- **Data quality is not optional.** Corrupt CRM data produces corrupt priority lists. Run the data-qa gate; surface flags; never silently skip a bad record.
- **Compose, don't duplicate.** Intent interpretation lives in `intent-signal-summarizer`. Deal comms live in `deal-communication-pack`. Deep account research lives in `account-dossier-builder`. Route; don't rebuild.
- **Truth discipline.** Every signal cited in the Digest must come from the user's actual data. Do not invent activity, infer intent, or fabricate signals. Mark inferred scores `[est]`.

## What Not to Do

- Do not write outreach copy here — this skill decides WHO and WHY; `cold-outreach-sequence-architect` or `deal-communication-pack` writes the HOW.
- Do not score accounts before `brand-brain` returns the ICP (P dimension is undefined without it).
- Do not surface 15+ action items in the Daily Digest — above ~8 top-ranked accounts the signal-to-noise collapses and the SDR ignores the list.
- Do not skip the data-quality gate on the grounds that "the data looks fine" — stale last-touch dates are the most common invisible corruption.
- Do not merge accounts across brands in a multi-brand session; keep slugs isolated.
- Do not treat PACT as rigid in edge cases — if a T=3 event fires (e.g., contract renewal in 48 hours), escalate immediately regardless of total score.

## Quality Checklist

- `brand-brain` called; ICP loaded; P scoring anchored to it?
- `data-qa-measurement-gotcha-checker` called; flags surfaced to user before any scores emitted?
- Every top-priority account has a specific next action (not "follow up") with the triggering signal named?
- Thin-file accounts (<2 touches) routed to `account-dossier-builder`?
- Intent data present? Routed to `intent-signal-summarizer`?
- Open opps in working set? Routed to `deal-communication-pack`?
- Daily Digest fits one screen; overflow saved to `./sdr-digests/`?
- Weekly Digest includes top movers, stalled accounts, and a concrete focus list for next week?
- All scores from user data; inferred scores marked `[est]`; no invented signals?
