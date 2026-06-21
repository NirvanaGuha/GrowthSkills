---
name: attribution-model-configurator
description: >
  Ad spend sources + GA4/CRM setup → attribution model recommendation with configuration spec
  (first-touch, linear, data-driven, last-touch, time-decay, position-based). Takes a business's
  channel mix, conversion window, and GA4/CRM state and returns: a reasoned model recommendation
  with tradeoffs, a GA4 reporting-attribution and ads-attribution configuration spec, a cross-channel
  data-driven-vs-rules comparison, and a QA checklist covering the classic attribution gotchas
  (attribution windows, sampling, direct/(none) dark traffic, self-referral, cross-device collapse,
  SRM in A/B tests, (not set), GA4 session vs event scope). Composes data-qa-measurement-gotcha-checker
  as a data-quality gate, channel-roi-scorecard and ltv-cac-payback-calculator for per-model
  revenue math, and analytics-report-reviewer to pressure-test the final spec before delivery.
  Use when someone says "which attribution model should I use," "set up attribution in GA4,"
  "data-driven vs last-click," "attribution window," "our conversions look inflated," "multi-touch
  attribution," "first-touch vs last-touch," "configure Google Ads attribution," "our ROAS looks off
  because of attribution," or hands over a GA4 property + channel list and asks how to set it up right.
---

# Attribution Model Configurator

Every attribution configuration is a business decision disguised as a settings menu. The wrong model doesn't just miscount conversions — it misallocates budget, misprices channels, and fires the wrong campaigns. This skill cuts through the model names, surfaces the real tradeoffs for your specific channel mix and funnel shape, and delivers a spec you can implement in GA4 and your CRM today.

It does not invent data. If the measurement setup is broken upstream — missing UTMs, self-referral loops, wrong conversion scope — no model choice fixes that. This skill finds those issues first.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's context: channel mix (if documented), ICP/funnel length, offer type (trial, demo, ecomm, freemium), and any known proof or benchmark numbers.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their channel list (paid search, paid social, email, organic, direct), typical conversion window in days, and primary conversion type (purchase, lead, trial, demo) before proceeding.
- **`data-qa-measurement-gotcha-checker`** (Step 2 gate) — runs a data-quality pass against the GA4/CRM setup before any model recommendation lands. Attribution config on broken plumbing is theater.
- **`channel-roi-scorecard`** (Step 4, optional) — per-model revenue math: what does each channel's attributed revenue look like under the recommended model vs. the current one? Delta = the budget reallocation signal.
- **`ltv-cac-payback-calculator`** (Step 4, optional) — longer-window revenue math; essential when funnel length exceeds the default 30-day attribution window.
- **`analytics-report-reviewer`** (Step 5) — reviews the final spec for unsupported claims, missing caveats, and visualization issues before delivery.

---

## How a run works

```
Step 0  Brand + context load  ──► brand-brain (channel mix, funnel shape, offer type)
Step 1  Intake                ──► collect inputs (structured interview if not provided)
Step 2  Data-quality gate     ──► data-qa-measurement-gotcha-checker
Step 3  Model recommendation  ──► Shapley/DDM framework → recommendation + tradeoffs
Step 4  Configuration spec    ──► GA4 + Google Ads + CRM settings, window settings
Step 5  Review pass           ──► analytics-report-reviewer on the full spec
Step 6  Deliver               ──► recommendation doc + QA checklist, saved to ./attribution/
```

---

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Extract: known channel mix, funnel length / buying cycle, conversion type (ecomm purchase, SaaS trial, lead, demo), any documented proof points around channel performance.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their channel list, typical conversion window in days, and primary conversion type before proceeding.

---

### Step 1 — Intake interview (if inputs are not already provided)

Ask in one batched block — never one question at a time:

| Input | Why it matters |
|---|---|
| Channel mix (list all active channels) | Determines whether multi-touch makes sense at all |
| Primary conversion event + GA4 key-event name | Scopes the attribution surface |
| Typical time from first touch to conversion (days) | Sets the window floor |
| Current GA4 reporting-attribution model + window | Baseline to diff against |
| Current Google Ads conversion attribution model | Often misaligned with GA4 |
| CRM or MMP in use (HubSpot, Salesforce, Adjust, etc.) | Cross-system reconciliation scope |
| Monthly conversion volume (approximate) | DDM eligibility check — GA4 requires ~400 conversions/30 days minimum |
| Pain driving this request ("ROAS looks inflated," "paid social undervalued," etc.) | Directs the tradeoff emphasis |

---

### Step 2 — Data-quality gate

Before recommending any model, invoke `data-qa-measurement-gotcha-checker`. A model recommendation is worthless if the input data has:

- **Self-referral contamination** — payment processors (Stripe, PayPal), SSO redirects, or third-party checkout appearing as a traffic source, collapsing multi-touch paths into direct/(none).
- **UTM coverage gaps** — campaigns without UTMs collapse into organic/direct, inflating those channels and understating paid.
- **Wrong conversion scope** — session-scoped key events in GA4 misattribute conversions to the session's last source, not the conversion event's source (a hidden last-click bias even in "data-driven").
- **Cross-domain tracking not configured** — subdomains or separate checkout domains breaking the session and resetting the attribution path.
- **Bot/spam traffic in conversions** — inflates volume, corrupts DDM training data.
- **Attribution window too short for the funnel** — a 7-day window on a 45-day SaaS sales cycle drops most touches.

Gate: if the gotcha-checker surfaces P0 issues (self-referral, UTM gap >20% of sessions, wrong conversion scope), surface those as blockers and recommend fixing before configuring the model. The model cannot be trusted until the data can be.

---

### Step 3 — Model recommendation (Shapley-informed framework)

**The framework: Shapley Value logic applied to available data.**

Shapley values (from cooperative game theory) assign credit to each "player" (channel/touchpoint) based on its marginal contribution across all possible orderings of the customer journey. Google's data-driven attribution (DDM) approximates this. Rules-based models are fixed heuristics that distort Shapley logic in predictable ways.

#### Model selection decision tree

```
Enough conversion volume for DDM?  (GA4 ≥ ~400 key events / 30 days; Google Ads ≥ 50/month)
├─ YES → Is the funnel > 14 days AND multi-channel (3+ channels contributing)?
│         ├─ YES → Data-driven (GA4 + Google Ads). Cross-check with linear for sanity.
│         └─ NO  → Data-driven still preferred; last-click as a comparison baseline.
└─ NO  → Rules-based only:
          ├─ Brand-new business / single-channel → Last-touch (least misleading with sparse data)
          ├─ Awareness-heavy mix (display, video, content) → Linear or time-decay
          ├─ Long considered-purchase funnel → Position-based (40/20/40) or time-decay
          └─ Direct-response only (paid search + email) → Last-click (acceptable; these channels
                                                            genuinely capture demand more than create it)
```

#### Model tradeoff reference (encode in output)

| Model | Credits | Inflates | Deflates | Best for |
|---|---|---|---|---|
| Last-click | Final touch | Branded search, direct | Awareness, email | Short funnels, demand capture only |
| First-click | Initial touch | Paid social, display | Nurture, retargeting | Pure acquisition measurement |
| Linear | All touches equally | Mid-funnel noise | High-intent closers | Baseline / sanity check |
| Time-decay | Recent touches more | Retargeting | Upper funnel | Short-window campaigns |
| Position-based (U-shape) | First + last 40% each | First/last touches | Mid-funnel | Balanced lead-gen |
| Data-driven (Shapley approx.) | Marginal contribution | Nothing systematically | Channels with thin data | Most mature accounts |

**Always note:** GA4's "data-driven" model is session-scoped by default. If the GA4 conversion key event is also session-scoped, there is a hidden last-click bias regardless of model name. Flag this explicitly.

---

### Step 4 — Configuration spec

Produce a concrete, copy-pasteable spec. No advice — actual settings.

#### GA4 Reporting Attribution (Admin → Attribution Settings)

```
Reporting attribution model:   [recommended model]
Lookback window — acquisitions: [30d / 60d / 90d — justified by funnel length from Step 1]
Lookback window — other events: [7d / 30d]
Credit views in reporting:      [Yes / No — Yes only if display/video are in the mix]
Engaged-view lookback:          [1d / 3d / 7d]
```

Note: this setting changes *all* historical data in GA4 Explorations and standard reports retroactively — no undo, no versioning. Document the change date as a GA4 annotation.

#### Google Ads Conversion Attribution (Tools → Conversions → [conversion action] → Attribution model)

```
Attribution model:      [DDM if eligible; else position-based or time-decay]
Click-through window:   [30d / 60d / 90d]
View-through window:    [1d if display/video; else off]
```

Note: Google Ads and GA4 use independent attribution calculations. Reporting will differ. Document the expected delta.

#### CRM / offline-conversion alignment (if applicable)

- Import offline conversions (Google Ads Offline Conversions / GA4 Measurement Protocol) to close the closed-won loop for B2B.
- Match `gclid` lifetime to the CRM deal cycle; default 90 days often too short for enterprise.
- Hubspot: map lifecycle stage changes to GA4 key events via webhook + Measurement Protocol for full-funnel attribution.

#### UTM governance note (gate)

If UTM coverage gaps were flagged in Step 2, output a UTM enforcement spec. Compose `utm-parameter-bulk-builder` if available and installed; else specify the required parameters and naming convention inline.

---

### Step 5 — Review pass

Invoke `analytics-report-reviewer` on the full recommendation. It checks for: unsupported claims, missing caveats on model limitations, weak assumptions in window choices, and whether the stated tradeoffs match the client's channel mix. Incorporate any flagged issues before delivery.

---

### Step 6 — Deliver + save

Save the output to `./attribution/[brand-slug]-attribution-spec.md`. Contents:

1. **Recommendation summary** — model, windows, one-paragraph rationale
2. **Data-quality gate results** — P0/P1 issues from Step 2, status (blocked/cleared)
3. **Tradeoff table** — what this model over/under-credits for this specific channel mix
4. **GA4 configuration spec** — copy-pasteable settings
5. **Google Ads configuration spec** — copy-pasteable settings
6. **CRM alignment notes** — if applicable
7. **Revenue delta (optional)** — channel-roi-scorecard delta if Step 4 ran
8. **QA checklist** — verification steps before trusting the new numbers

---

## The classic gotchas (encode these in every output)

These are the attribution errors that corrupt growth decisions. Surface the relevant ones even when the user didn't ask:

1. **Attribution window shorter than the funnel.** A 7-day window on a 30-day SaaS trial means most influenced conversions are credited to "direct" or the last re-touch. Ask for the median time-to-convert before setting any window.
2. **GA4 data-driven ≠ neutral.** DDM requires sufficient volume. Under the threshold GA4 silently falls back to last-click without notifying you.
3. **Session-scoped key events create hidden last-click bias.** Even with DDM enabled, if the key event fires at session level (not event level), the source attributed is the session's entry source — the last channel. Always verify key-event scope in DebugView or the event schema.
4. **Self-referral breaks paths.** Stripe, PayPal, SSO, and subdomain checkouts reset the session and set the source to the processor domain. Fix: add all payment/checkout domains to the referral exclusion list AND configure cross-domain measurement.
5. **Direct/(none) is a catch-all for broken attribution.** A high direct share (>20% on a non-brand-dominant business) is not a channel — it's a measurement failure. Investigate before attributing budget to it.
6. **View-through attribution double-counts.** GA4 and Google Ads can both claim the same conversion through view-through credit. If display/video CPMs look unusually efficient, check view-through window settings.
7. **Cross-device collapse.** GA4 uses User-ID and Google Signals to stitch cross-device journeys; without User-ID implemented, mobile/desktop paths are severed and upper-funnel mobile credit is lost.
8. **SRM in concurrent A/B tests.** Running a CRO test while changing the attribution model creates a sample-ratio mismatch — the test and the attribution change contaminate each other. Stage them.
9. **(not set) in GA4 channel groupings.** Often caused by UTM values that don't match the default channel group rules. Fix: audit channel grouping rules in Admin → Data Settings → Channel Groups.
10. **GA4 vs Google Ads reporting divergence is expected and normal** — different attribution models, different conversion-event definitions, different deduplication logic. Document the expected delta rather than "fixing" one to match the other.

---

## Principles (Non-Negotiable)

- **Data quality gates model choice.** No model recommendation until the gotcha-checker clears or explicitly documents the blockers.
- **Compose, don't re-derive.** Revenue math goes through `channel-roi-scorecard` and `ltv-cac-payback-calculator`; data-quality checks go through `data-qa-measurement-gotcha-checker`; spec review goes through `analytics-report-reviewer`. Don't rebuild shared logic here.
- **Deliver specs, not advice.** The output is copy-pasteable GA4 and Google Ads settings — not a blog post about attribution theory.
- **Name the tradeoffs per channel mix.** "Data-driven is better" is not a recommendation. "Data-driven will shift ~15–25% of credited conversions from branded search to paid social and email, reflecting their assist role in your funnel" is.
- **Truth only.** Real numbers or `[verify]`. Never invent conversion volumes, window benchmarks, or channel credit percentages.
- **Document the annotation.** Any change to GA4 reporting attribution must be recorded as a GA4 annotation on the change date — or six months later nobody knows why the numbers shifted.

## What Not to Do

- Don't recommend data-driven attribution to accounts under the conversion volume threshold — GA4 will silently fall back to last-click and the operator won't know.
- Don't skip the data-quality gate. Attribution config on broken UTMs or self-referral is confetti math.
- Don't conflate GA4 reporting attribution (changes the Explorations/standard reports retroactively) with Google Ads conversion attribution (changes Smart Bidding signals forward-only). They are independent and both need to be set.
- Don't treat direct/(none) as a channel. Investigate it.
- Don't recommend "just use data-driven" as a shortcut. Name the volume requirement, the session-scope caveat, and the window it uses.
- Don't produce a recommendation without noting what this model will over- and under-credit for this specific channel mix.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand (or explicit fallback inputs) loaded before any work?
- `data-qa-measurement-gotcha-checker` run; P0 issues surfaced before any model recommendation?
- Model selected via the Shapley/DDM decision tree, not by name-preference?
- Tradeoff table populated with this brand's actual channel mix (not generic filler)?
- GA4 reporting-attribution spec includes model, both lookback windows, view-credit setting, and the retroactive-change warning?
- Google Ads spec includes model, click-through window, and view-through window?
- All 10 classic gotchas checked; relevant ones surfaced in the output?
- `analytics-report-reviewer` called; any flagged issues resolved before delivery?
- Output saved to `./attribution/[brand-slug]-attribution-spec.md`?
- GA4 annotation step included as a reminder?
