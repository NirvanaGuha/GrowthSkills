---
name: ga4-weekly-traffic-digest
description: >
  GA4 property + date range → formatted weekly traffic summary (sessions, users, channels, top pages,
  WoW delta). Pulls the current week and prior week from the GA4 Data API (or accepts a pasted export),
  runs it through a structured 5-section analysis (Volume, Channels, Content, Acquisition Quality,
  Anomalies), flags known GA4 measurement gotchas before you draw conclusions, and writes a
  decision-grade narrative digest a growth lead can act on — not just a numbers table. Calls
  `data-qa-measurement-gotcha-checker` to catch self-referral, spam hostnames, broken events, and model
  blending before surfacing any findings. Use when the user says "run the weekly traffic digest,"
  "GA4 weekly summary," "what happened to traffic this week," "WoW traffic," "weekly report," or
  pastes a GA4 export and asks for commentary.
---

# GA4 Weekly Traffic Digest

One property, one week, one decision-ready summary. This skill does the mechanical and interpretive work — pulling sessions/users/channels/top pages, computing WoW deltas, flagging GA4's endemic measurement problems, and writing the narrative in plain language so you act on signal, not noise.

It does not redesign your measurement setup, run attribution modelling, or generate any copy or campaign ideas. For full growth diagnosis see `growth-diagnostic-deep-dive`; for paid-channel decomposition see `channel-roi-scorecard`; for funnel drop-off interpretation see `funnel-drop-off-analyzer`.

---

## Skills this calls

| Skill | When |
|---|---|
| **`brand-brain`** | Required first step — loads brand slug, GA4 property ID, and any known measurement quirks saved in brand context. |
| **`data-qa-measurement-gotcha-checker`** | Called before presenting any numbers — runs the standard GA4 pitfall checklist on the data. |
| `channel-roi-scorecard` | Optional — if paid-channel spend data is present, hand off to this skill for ROAS/CPA layer. |
| `funnel-drop-off-analyzer` | Optional — if the digest surfaces a sharp drop-off pattern, this skill takes the funnel data. |
| `growth-diagnostic-deep-dive` | Optional — escalate to this when a WoW anomaly needs a full root-cause diagnosis. |

---

## How a run works

```
Step 0  Load the brand          ──► brand-brain (gets property ID, quirks, hostname filter)
Step 1  Resolve data source     ──► API pull (preferred) or accept pasted export
Step 2  QA gate                 ──► data-qa-measurement-gotcha-checker on the raw pull
Step 3  Run the 5-section framework
Step 4  Write the digest narrative
Step 5  Offer to save + surface escalation paths
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill. It returns the active brand's GA4 property ID, any saved hostname filters (e.g., `pushengage.com` only, not SDK subdomains), known data quirks (broken events, junk traffic sources, excluded referrals), and the brand slug. If no property ID is in the brain, ask for it once and offer to save it back.

**Fallback:** if `brand-brain` is absent, read `~/.brandbrain/brands/.active` + `brand.md` directly, or ask the user for the property ID and hostname filter.

### Step 1 — Resolve the data source

**Preferred (live pull):** Use the `analytics-mcp` connector or the GA4 Data API. Pull two ISO weeks side-by-side (current `[Mon–Sun]` and prior). Default dimensions: `sessionDefaultChannelGroup`, `landingPage`, `deviceCategory`. Default metrics: `sessions`, `totalUsers`, `newUsers`, `engagedSessions`, `engagementRate`, `bounceRate`, `screenPageViews`. Apply the brand's hostname filter as a dimension filter on `hostName`.

**Fallback (pasted export):** Accept a CSV or table paste. Note the date range(s) and whether prior-week data is included. Mark any WoW delta as `[verify — prior week not supplied]` if only one period given.

If the date range is not specified by the user, default to the most recently completed Mon–Sun week vs. the week before.

### Step 2 — QA gate (non-skippable)

Invoke `data-qa-measurement-gotcha-checker` on the raw data. Do not present numbers until it returns. If it flags issues, surface them at the top of the digest in a **Data quality notes** block before the analysis — not buried at the end.

---

## The 5-Section Framework (VCQCA)

A weekly traffic digest earns trust by being consistently structured. Every run uses the same five sections in the same order so stakeholders can scan, not relearn.

### Section 1 — Volume headline

| Metric | This week | Prior week | WoW Δ | Δ% |
|---|---|---|---|---|
| Sessions | | | | |
| Total users | | | | |
| New users | | | | |
| Engaged sessions | | | | |
| Engagement rate | | | | |

One sentence: "Traffic was [up/down/flat] week-over-week. [Leading cause if obvious, else defer to Channels.]"

Do not call out a number as significant unless the absolute change is material (use the brand's baseline to calibrate; default: flag moves >10% WoW or >15% on a low-volume property). Small fluctuations on low-volume sites are noise — say so.

### Section 2 — Channel breakdown

| Channel | Sessions | WoW Δ% | Share | Δ share pts |
|---|---|---|---|---|
| Organic Search | | | | |
| Direct | | | | |
| Referral | | | | |
| Organic Social | | | | |
| Email | | | | |
| Paid Search | | | | |
| Paid Social | | | | |
| (Unassigned) | | | | |

**Interpretation rules:**
- A spike in Direct often means UTM tagging broke, not that brand traffic grew. Flag it.
- Unassigned > 5% of sessions is a measurement problem, not a channel. Flag it.
- Organic Social spikes on a Monday/Tuesday trace to weekend posts. Note it.
- Paid changes should cross-reference spend (hand off to `channel-roi-scorecard` if spend data is available).

### Section 3 — Content (top landing pages)

Top 10 landing pages by sessions this week, with WoW delta, engagement rate, and a one-column note (new post, algorithm bump, social share, etc. — leave blank if unknown; never invent).

Flag any page where engagement rate dropped >10 pts WoW — signals a traffic-quality shift (bot, wrong SERP, broken redirect) not necessarily a UX problem.

### Section 4 — Acquisition quality

| Metric | This week | Prior week | WoW Δ |
|---|---|---|---|
| Engagement rate | | | |
| Avg engaged session duration | | | |
| Pages per session (if available) | | | |

Quality check: if sessions went up but engagement rate dropped, the new traffic is lower-quality. Call this out explicitly — headline growth that hides quality erosion is a common stakeholder trap.

### Section 5 — Anomalies and open questions

List every WoW move the analysis cannot explain from the data alone. Format:

> **[Metric] was [X], [Δ%] WoW.** Possible causes: [list]. Recommended next step: [one action].

Keep this honest. "We don't know why" is a valid answer. Never fabricate an explanation.

---

## The digest narrative

After the five sections, write a 3–5 sentence plain-English summary paragraph. Structure:

1. The headline move (sessions up/down/flat, why if known).
2. The channel story (which channel drove it; any quality flag).
3. The content story (standout page, if any).
4. The one thing to watch or act on next week.

Senior standard: the narrative should be quotable in a Slack message to a non-analyst stakeholder. No jargon, no GA4 internal terms (say "organic search traffic" not "Organic Search channel group"), no invented causes.

---

## GA4 gotchas — always check before concluding

These are the most common ways GA4 weekly data misleads. `data-qa-measurement-gotcha-checker` covers them systematically; this is a quick in-skill reminder of the ones that corrupt weekly analysis most often.

| Gotcha | What it looks like | Check |
|---|---|---|
| **Hostname pollution** | Sessions from non-brand subdomains, staging, localhost inflate total sessions | Hostname dimension filter; check for unexpected hosts |
| **Broken session-start event** | `session_start` fires but `page_view` does not → sessions count, engagement drops | Check event counts for `page_view` vs `session_start` |
| **UTM stripping on redirect** | OAuth/payment redirect pages show as Direct after an email or paid click | Review Direct spike against last-click and assisted |
| **Sampling on large reports** | GA4 samples at >10M events in the UI (not API) | Use API or filter to smaller date range |
| **Data freshness lag** | Yesterday's data is incomplete until ~48h after event time | Never run a digest on an in-progress day |
| **Unassigned channel group** | Traffic with no medium → Unassigned; often UTM errors | Investigate if Unassigned > 3–5% of sessions |
| **Spam referral traffic** | Sessions from known spam domains spike and then vanish | Add internal traffic filter + referral exclusion |
| **GA4 attribution model shift** | Google quietly updated last-click → data-driven for many properties | Check if model changed in Admin > Attribution Settings |

---

## Principles (Non-Negotiable)

- **QA before narrative.** `data-qa-measurement-gotcha-checker` runs before any numbers are presented. Data quality problems belong at the top of the report, not a footnote.
- **Signal over noise.** Don't flag every small delta. Calibrate to the property's typical variance; small moves on large-volume sites are not news.
- **Volume ≠ quality.** Engagement rate, engaged session duration, and new-vs-returning patterns always accompany raw session counts. Never headline sessions without a quality read.
- **Honest unknowns.** If the data doesn't explain the anomaly, say so. Fabricated explanations are worse than gaps.
- **Consistent structure, every week.** The value of a digest compounds when stakeholders know where to look. VCQCA order is non-negotiable.
- **Brand-brain first.** Property ID, hostname filter, and saved quirks come from `brand-brain`. Don't ask the user to re-supply them each week once saved.

## What Not to Do

- Don't present numbers before the QA gate runs.
- Don't invent causes for anomalies — flag them as open questions.
- Don't treat a Direct spike as brand-intent traffic without ruling out broken UTMs.
- Don't compare to a holiday week or a week with a known spike without noting the baseline distortion.
- Don't run the digest on a partial week and present it as a full-week number without marking it `[partial week — data incomplete]`.
- Don't omit Unassigned or lump it into Direct — surface it separately.
- Don't redesign the measurement setup inside this skill; if setup problems surface, route to `data-qa-measurement-gotcha-checker` or flag for the user to address.

## Quality Checklist (self-review before presenting)

- `brand-brain` called; property ID and hostname filter loaded?
- Date range confirmed as completed Mon–Sun weeks (not partial)?
- `data-qa-measurement-gotcha-checker` run; any flags surfaced at the top of the digest?
- All five VCQCA sections present and in order?
- Volume table includes WoW delta AND engagement rate — not sessions alone?
- Channel table flags Direct spike, Unassigned > 5%, and any channel with suspiciously high share?
- Quality section present (engagement rate, engaged duration)?
- Anomalies section honest — no invented explanations?
- Narrative is 3–5 sentences, jargon-free, quotable in Slack?
- Offer to save to `./reports/traffic-digest-[YYYY-WNN].md` made?
