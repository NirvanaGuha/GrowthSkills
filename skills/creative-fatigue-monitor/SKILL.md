---
name: creative-fatigue-monitor
description: >
  Diagnoses creative fatigue across paid ad accounts — Meta, Google, TikTok, LinkedIn, or any
  platform that exposes frequency + CTR/CPM trend data. Takes raw ad performance data (CSV export,
  pasted table, or manual numbers) and applies our Hook-Resonance Decay (HRD) working model — a house framework, not an industry standard — to flag each active
  creative as Fresh, Watch, Fatiguing, or Burned. Outputs a ranked fatigue report with per-creative
  rotation recommendations, a refresh brief for the worst offenders, and an audience saturation
  note. Composes data-qa-measurement-gotcha-checker for data integrity before drawing conclusions,
  and hands rotation briefs off to social-ad-copy-writer or ad-to-landing-page-message-match-auditor
  when creative replacement is needed. Use when the user says "my ads are fatiguing," "CTR is
  dropping," "frequency is too high," "which creatives should I rotate," "ads aren't performing like
  they used to," "creative refresh brief," "audit my ad account for fatigue," or pastes a
  performance table and asks what's wrong.
---

# Creative Fatigue Monitor

Ad performance doesn't die suddenly — it decays. Frequency climbs while CTR falls; CPM rises as the algorithm de-prioritizes a signal it has learned to distrust. This skill reads that decay curve, names the creatives in each danger zone, and tells you exactly what to do with each one — rotate, refresh, or kill — before you burn more budget showing the same exhausted creative to an audience that has already stopped seeing it.

Our working model: **Hook-Resonance Decay (HRD)** (a house framework we use here, not an established industry standard) — a structured read of frequency × engagement-rate trajectory × CPM delta × estimated audience saturation. Every creative is assigned a fatigue tier. Decisions follow the tier, not gut feel.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's ICP, voice, offer, and banned words so refresh recommendations stay on-voice and on-strategy.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP description, and top 2–3 active offers before proceeding.
- **`data-qa-measurement-gotcha-checker`** (Step 1, data gate) — checks the input data for attribution-window mismatches, sampling, conversion-lag bias, and self-referral noise before any fatigue call is made.
- **`analytics-report-reviewer`** (optional, output review) — reviews the final fatigue report for unsupported claims or weak assumptions before presenting to a stakeholder.
- **`social-ad-copy-writer`** (optional, downstream) — called when the user wants net-new replacement creative for Burned or Fatiguing ads.
- **`ad-to-landing-page-message-match-auditor`** (optional, downstream) — called when refreshing the hook angle to check the new ad still matches the landing page.
- **`bid-budget-pacing-checker`** (optional, downstream) — called when fatigue findings suggest bid floor or budget reallocation, not just creative swap.
- **`a-b-multivariate-test-designer`** (optional, downstream) — called when the user wants a structured rotation test plan rather than a direct swap.

---

## How a run works

```
Step 0  Brand context    ──► call brand-brain
Step 1  Data gate        ──► call data-qa-measurement-gotcha-checker (or flag manually)
Step 2  Classify         ──► apply HRD tiers per creative
Step 3  Rank + report    ──► fatigue table, saturation note, rotation queue
Step 4  Refresh brief    ──► one-paragraph creative brief per Burned/Fatiguing ad (or call social-ad-copy-writer)
Step 5  Hand off         ──► optional: call downstream siblings as needed
```

---

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Receive: active brand slug, ICP + awareness tendency, voice adjectives, banned words, offer mechanics. Refresh recommendations must stay on-voice and reference only real proof from the brand digest. Mark anything unconfirmed `[verify]`.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP description, and top 2–3 active offers before proceeding.

### Step 1 — Data quality gate

Before making any fatigue call, run (or simulate) `data-qa-measurement-gotcha-checker` on the input data. Specifically:

- **Attribution window alignment** — ensure all creatives share the same click/view window; mixing 1-day-click with 7-day-click inflates some CTRs.
- **Reporting lag** — last 1–3 days of conversion data are incomplete; exclude or flag them.
- **Audience overlap / campaign structure** — overlapping ad sets inflate per-creative frequency; note it.
- **Minimum impression threshold** — discard any creative with fewer than 1,000 impressions (or user's stated threshold); too small to call.
- **Platform sampling** — GA4 / reporting dashboards sometimes sample; note if source is sampled.

If the user's data has unresolvable quality issues, surface them and ask for a cleaner export before proceeding. Never produce a fatigue verdict on bad data.

---

## Hook-Resonance Decay (HRD) — our working framework

HRD (our house framework, not an established industry model) models creative performance as a three-phase curve: **Hook phase** (rising engagement, algorithm learning), **Resonance plateau** (peak CTR/CPM efficiency), **Decay phase** (falling CTR, rising frequency, CPM inflation as the algorithm re-prices a declining signal). The goal is to catch creatives at the inflection — before they burn spend in full Decay.

### Primary fatigue signals (collect for each creative)

| Signal | What to collect | Why it matters |
|---|---|---|
| **Frequency** | Avg. impressions / unique reach (7-day, 14-day, 28-day) | Core saturation indicator — audience is re-seeing the same creative |
| **CTR trend** | CTR this week vs. prior week vs. launch week | Rate-of-decay is more actionable than a snapshot |
| **CPM delta** | CPM this period vs. prior period | Algorithmic signal: platform de-prioritizes exhausted creatives, raising cost |
| **Hook Rate** | 3-sec video views ÷ impressions (video only) | Leading indicator — drops before CTR does |
| **Engagement Rate** | (clicks + reactions + saves) ÷ impressions | Broader signal than CTR; catches video-heavy and awareness campaigns |
| **Conversion Rate trend** | CVR this week vs. launch week | Lagging but decisive — confirms or overrides upstream signals |
| **Spend share** | % of total campaign spend going to this creative | High spend on a fatiguing creative = compounding loss |

### HRD Fatigue Tiers

Classify every creative into exactly one tier:

```
┌─────────────┬──────────────────────────────────────────────────────────────────────────┐
│  FRESH      │ Freq < 2.5 | CTR flat or rising | CPM flat or falling | < 7 days live   │
│             │ Action: leave it, monitor weekly                                          │
├─────────────┼──────────────────────────────────────────────────────────────────────────┤
│  WATCH      │ Freq 2.5–4 | CTR declining ≤15% WoW | CPM rising ≤10% | 7–21 days live  │
│             │ Action: earmark for refresh; write a replacement now                      │
├─────────────┼──────────────────────────────────────────────────────────────────────────┤
│  FATIGUING  │ Freq 4–7 | CTR declining 15–30% WoW | CPM rising 10–25% | 21+ days live │
│             │ Action: begin rotation within 48 h; immediate refresh brief               │
├─────────────┼──────────────────────────────────────────────────────────────────────────┤
│  BURNED     │ Freq > 7 | CTR declining > 30% WoW | CPM rising > 25%                   │
│             │ Action: pause within 24 h; rebuild from scratch                           │
└─────────────┴──────────────────────────────────────────────────────────────────────────┘
```

**Override rules** (any single signal can escalate a tier):
- Hook Rate drop > 40% WoW on video → escalate one tier regardless of frequency.
- CPM spike > 50% WoW → immediately BURNED regardless of frequency.
- CVR drop > 35% WoW with stable landing page → escalate to FATIGUING minimum.
- Only two signals available (e.g., no frequency data) → flag confidence as LOW and note which signals are missing.

### Audience saturation note

After classifying all creatives, estimate audience saturation: `total campaign frequency × audience size → estimated unique exposures`. If > 60% of the target audience has been reached ≥ 3 times, recommend audience expansion alongside creative rotation (new lookalikes, interest layers, or exclusion refresh).

---

## Output format

```markdown
## Creative Fatigue Report — [Brand] / [Campaign or Ad Account] / [Date range]
Brand: [slug, via brand-brain]
Data quality: [PASS / PASS-with-caveats / BLOCKED — note from Step 1]
Audience saturation: [estimate and note]

### Fatigue Summary
| Creative ID / Name | Tier | Freq | CTR Δ WoW | CPM Δ WoW | Spend % | Priority action |
|---|---|---|---|---|---|---|
| ... | BURNED | 9.2 | −42% | +31% | 18% | Pause within 24 h |
| ... | FATIGUING | 5.1 | −22% | +14% | 27% | Rotate within 48 h |
| ... | WATCH | 3.4 | −9% | +6% | 12% | Brief replacement now |
| ... | FRESH | 1.8 | +4% | −2% | 43% | Monitor weekly |

### Rotation Queue (priority order)
1. [Creative name] — BURNED — pause immediately — see refresh brief below
2. [Creative name] — FATIGUING — rotation brief below
3. [Creative name] — WATCH — replacement in progress / briefed

### Refresh Briefs
**[Creative name] (BURNED — rebuild):**
Angle that ran: [the hook/angle the burned creative used]
Why it fatigued: [saturation + angle exhaustion diagnosis, 1 sentence]
New angle to test: [specific angle shift, on-brand, uses brand proof]
Format recommendation: [static/video/carousel, if evident from the data]
ICP fit note: [how this angle maps to the brand's ICP + awareness stage]
> Call `social-ad-copy-writer` with this brief to generate replacement copy.

**[Creative name] (FATIGUING — refresh):**
[same structure, shorter — hook swap may be sufficient]

### Downstream actions
- [ ] Pause: [list]
- [ ] Brief to `social-ad-copy-writer`: [list]
- [ ] Message-match check via `ad-to-landing-page-message-match-auditor` for any new hook angle
- [ ] Audience expansion recommendation: [yes/no + note]
- [ ] Budget reallocation: call `bid-budget-pacing-checker` if spend > 20% locked to BURNED creatives
```

Save to `./paid/fatigue-report-[brand]-[YYYY-MM-DD].md` when asked or when the report covers > 5 creatives.

---

## Principles (Non-Negotiable)

- **Data quality before verdict.** Never assign a fatigue tier without passing Step 1. Bad data produces confident wrong answers.
- **Trend over snapshot.** A single-period CTR number is meaningless without the prior week and launch week to compare against. Ask for it.
- **Tier is decisive.** Once a tier is assigned, the action follows. Don't hedge — say what to do and when.
- **Brand-brain first.** Refresh briefs are on-voice, on-ICP, and use only real proof from the brand digest. Unconfirmed proof is `[verify]`.
- **Rotate the angle, not just the creative.** A reskin of a BURNED hook still carries the burned message. Specify what angle shifts.
- **Compose, don't re-derive.** Hand replacement creative work to `social-ad-copy-writer`; hand message-match checks to `ad-to-landing-page-message-match-auditor`; hand budget shifts to `bid-budget-pacing-checker`. This skill diagnoses and briefs.
- **Low-confidence flags are mandatory.** If fewer than two HRD signals are available for a creative, label the tier LOW CONFIDENCE and say why.

## What Not to Do

- Don't classify creatives with < 1,000 impressions (or stated threshold) — too small to read the curve.
- Don't confuse frequency inflation from overlapping ad sets with genuine creative fatigue — note the structural cause.
- Don't recommend pausing a FRESH creative because the campaign overall is underperforming — separate structural budget issues (call `bid-budget-pacing-checker`) from creative issues.
- Don't invent proof points or angle recommendations that contradict the brand's known positioning.
- Don't output a single-period snapshot as a fatigue verdict — always require at least two time periods.
- Don't re-implement brand scanning, audience persona work, or A/B test design — call the relevant sibling.

## Quality Checklist (self-review before presenting)

- `brand-brain` called; active brand digest loaded; refresh briefs are on-voice with real proof?
- `data-qa-measurement-gotcha-checker` run (or manual equivalent); data quality decision noted in header?
- Every creative with ≥ 1,000 impressions classified into exactly one HRD tier?
- At least two HRD signals used per classification; LOW CONFIDENCE flagged where fewer than two are available?
- Override rules checked (Hook Rate drop, CPM spike, CVR drop)?
- Audience saturation estimate included?
- Rotation queue is ordered and actionable with a named deadline (24 h / 48 h / weekly)?
- Refresh brief for every BURNED and FATIGUING creative — with a specific angle shift, not just "try something new"?
- Downstream skill hand-offs identified (`social-ad-copy-writer`, `ad-to-landing-page-message-match-auditor`, `bid-budget-pacing-checker`)?
- Output saved to `./paid/fatigue-report-[brand]-[YYYY-MM-DD].md` when > 5 creatives or explicitly requested?
