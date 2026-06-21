---
name: campaign-qa-launch-checklist-generator
description: >
  Campaign brief + channel/type → channel-specific pre-launch QA checklist with pass/fail fields
  covering copy, creative, UTMs, targeting, tracking, and legal. Accepts any combination of
  channels (paid search, paid social, email, push, organic social, display, affiliate) and any
  campaign type (acquisition, nurture, product launch, promotional, ABM, event). Generates a
  structured checklist with explicit pass/fail/flag columns, ordered by severity, so a junior
  marketer can run a final review without missing a launch-blocker. Composes the brief from
  campaign-brief-builder if one doesn't exist; UTM coverage from utm-parameter-bulk-builder;
  A/B test spec from a-b-multivariate-test-designer if tests are running. Flags tracking gaps
  against the data-qa-measurement-gotcha-checker ruleset. Outputs a signed-off QA sheet ready
  for ops handoff. Trigger phrases: "pre-launch checklist," "campaign QA," "launch checklist,"
  "ready to launch?," "launch review," "QA this campaign," "is this campaign ready," "go/no-go
  on this campaign," "what do I still need to check," "sign off on launch."
---

# Campaign QA & Launch Checklist Generator

Given a campaign brief and one or more channels, produce the exact QA checklist a campaign manager needs to clear before hitting launch — nothing more, nothing less. Checklist items map to real failure modes: ad account billing errors, broken UTMs, missing conversion events, legal copy gaps, undefined negative audiences. No filler. Every item is pass/fail/flag with a clear owner and a remediation path.

This skill gates launches, not blesses them. If blockers are found, it names them explicitly and will not declare a campaign ready until they are resolved or explicitly waived.

---

## Skills this calls

- **`brand-brain`** (required) — loads active brand's voice, banned words, ICP, real proof, and offer mechanics to review creative and copy against brand constraints. **Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand name, voice, banned words, and ICP before running the QA pass.
- **`campaign-brief-builder`** (when no brief exists) — if the user presents raw goals rather than a structured brief, invoke this skill first to create the brief, then QA it.
- **`utm-parameter-bulk-builder`** (when UTM review is needed) — validates naming conventions and completeness of tracking links before launch.
- **`a-b-multivariate-test-designer`** (when tests are running) — ensures the test brief is complete, feature-flag spec is confirmed, and variant targeting won't contaminate results.
- **`data-qa-measurement-gotcha-checker`** (for analytics/tracking items) — applies the standard tracking pitfall ruleset (self-referral, spam hostnames, broken events) to the campaign's measurement setup.
- **`landing-page-heuristic-live-cro-auditor`** (optional) — if a new landing page is the destination, pull its audit scores into the checklist's destination/post-click section.

---

## How a run works

```
Step 0  Load brand           ──► call brand-brain; get voice, ICP, banned words, real proof
Step 1  Confirm the brief    ──► structured brief in hand? Else call campaign-brief-builder
Step 2  Identify scope       ──► channels + campaign type → select checklist modules
Step 3  Run the checklist    ──► fill pass/fail/flag for every item in sequence
Step 4  Triage findings      ──► classify: Launch Blocker | Recommended Fix | FYI Note
Step 5  Render output        ──► signed QA sheet with verdict and next steps
Step 6  Save artifact        ──► ./ops/qa/[campaign-slug]-qa-[YYYY-MM-DD].md
```

---

## The QA Framework — Launch-Gate Severity Triage + Channel Modules

The checklist is structured around two axes:

**Severity triage (every item gets one):**
- `BLOCKER` — campaign must not launch until resolved (broken tracking, missing legal copy, payment method invalid, undefined conversion event)
- `RECOMMEND` — addressable risk that degrades performance but won't break the campaign
- `FYI` — logged observation; owner decides whether to act before launch

**Channel modules (compose only what applies):**
Select all modules that match the campaign's live channels.

---

### Module 0 — Universal (all campaigns)

| # | Item | Pass / Fail / Flag | Owner | Notes |
|---|---|---|---|---|
| U1 | Campaign brief exists and is approved | | | Invoke `campaign-brief-builder` if absent |
| U2 | Campaign goal has a single measurable KPI (not "more traffic") | | | |
| U3 | ICP and audience definition match brand brain ICP | | | |
| U4 | Offer mechanics confirmed (price, guarantee, expiry if any) | | | |
| U5 | All copy and creative cleared against brand voice + banned words | | | |
| U6 | Legal/compliance copy present: T&Cs, asterisks, GDPR/CAN-SPAM notices per channel | | | |
| U7 | All destination URLs resolving and load under 3 s | | | |
| U8 | Mobile render confirmed for all creatives and pages | | | |
| U9 | UTM parameters present on every tracked link (invoke `utm-parameter-bulk-builder`) | | | |
| U10 | Conversion event fires on destination (click through and verify in debug view) | | | |
| U11 | Campaign start/end dates and budget caps set in platform | | | |
| U12 | All stakeholder approvals documented | | | |

---

### Module 1 — Paid Search (Google Ads / Microsoft Ads)

| # | Item | Severity | Pass / Fail / Flag |
|---|---|---|---|
| P1 | Billing method valid; no payment hold on account | BLOCKER | |
| P2 | Conversion actions linked and recording (not "inactive") | BLOCKER | |
| P3 | Negative keyword list applied at campaign or ad-group level | RECOMMEND | |
| P4 | Ad strength ≥ "Good" for RSAs; no single-ad-group setups | RECOMMEND | |
| P5 | Search term report reviewed; branded vs non-branded separation confirmed | RECOMMEND | |
| P6 | Ad extensions (sitelinks, callouts, structured snippets) populated | RECOMMEND | |
| P7 | Audience observation layers set (remarketing, customer match) | FYI | |
| P8 | Smart bidding strategy set; target CPA/ROAS calibrated to historical data | RECOMMEND | |
| P9 | Final URL matches the headline promise (message match) | BLOCKER | |

---

### Module 2 — Paid Social (Meta, LinkedIn, TikTok)

| # | Item | Severity | Pass / Fail / Flag |
|---|---|---|---|
| S1 | Pixel/CAPI/SDK firing on destination page and confirmed in Events Manager | BLOCKER | |
| S2 | Audience size above minimum threshold (≥ 1 K for custom, ≥ 50 K for prospecting) | BLOCKER | |
| S3 | Lookalike or custom audience seed list freshness confirmed (≤ 90 days) | RECOMMEND | |
| S4 | Ad creative dimensions match placement specs; no text exceeding 20% [verify per platform] | BLOCKER | |
| S5 | Attribution window set and documented (e.g. 7-day click, 1-day view) | RECOMMEND | |
| S6 | Exclusion audiences applied (existing customers, bounced visitors, competitors if available) | RECOMMEND | |
| S7 | Frequency cap set for retargeting ad sets | RECOMMEND | |
| S8 | Budget split between ad sets reviewed; no single ad set ≥ 80% of budget | RECOMMEND | |
| S9 | A/B test running? Invoke `a-b-multivariate-test-designer` for complete spec | FYI | |

---

### Module 3 — Email

| # | Item | Severity | Pass / Fail / Flag |
|---|---|---|---|
| E1 | Unsubscribe link present and functional | BLOCKER | |
| E2 | Sender domain authenticated: SPF, DKIM, DMARC all passing | BLOCKER | |
| E3 | Physical mailing address in footer (CAN-SPAM / CASL requirement) | BLOCKER | |
| E4 | List segmentation logic correct; suppression list applied | BLOCKER | |
| E5 | Subject line + preview text render correctly across Gmail, Outlook, Apple Mail | RECOMMEND | |
| E6 | All links tested (including UTMs); no broken hrefs | BLOCKER | |
| E7 | Plain-text version exists and renders coherently | RECOMMEND | |
| E8 | Spam score checked (e.g. via Mail Tester or GlockApps) < 2 [verify threshold with tool] | RECOMMEND | |
| E9 | Send time confirmed; timezone logic correct for segmented sends | RECOMMEND | |
| E10 | Transactional vs marketing designation correct in ESP | BLOCKER | |

---

### Module 4 — Push Notifications

| # | Item | Severity | Pass / Fail / Flag |
|---|---|---|---|
| N1 | Subscriber segment defined and confirmed (not "all subscribers") | RECOMMEND | |
| N2 | Quiet-hours/DND rules applied per subscriber timezone | RECOMMEND | |
| N3 | Notification title ≤ 50 chars; body ≤ 125 chars (platform safe zone) [verify] | RECOMMEND | |
| N4 | Deep-link or URL verified for correct UTMs and page resolve | BLOCKER | |
| N5 | Send frequency check: subscriber hasn't received push in last 24 h if non-transactional | RECOMMEND | |
| N6 | A/B variant (if running) has defined winner metric and sample-size target | FYI | |

---

### Module 5 — Display / Programmatic

| # | Item | Severity | Pass / Fail / Flag |
|---|---|---|---|
| D1 | All creative sizes delivered and trafficked (IAB standard set confirmed) | BLOCKER | |
| D2 | Brand-safety and content-category exclusions applied | RECOMMEND | |
| D3 | Viewability standard documented (MRC 50% in-view for 1 s) [verify] | FYI | |
| D4 | Click tracker / impression tracker URLs firing in QA environment | BLOCKER | |
| D5 | Frequency cap set per user per day | RECOMMEND | |

---

### Module 6 — Organic Social / Content Activation

| # | Item | Severity | Pass / Fail / Flag |
|---|---|---|---|
| O1 | Post copy reviewed against brand voice and banned words | RECOMMEND | |
| O2 | UTM or link shortener with tracking applied to all outbound links | RECOMMEND | |
| O3 | Image/video specs meet platform requirements; no cropped or blurry assets | RECOMMEND | |
| O4 | Hashtags reviewed: no hijacked or ambiguous tags | FYI | |
| O5 | Scheduled post timing confirmed in social scheduling tool | FYI | |

---

## Launch Verdict Template

```
## QA Sign-Off — [Campaign Name]
Date: [YYYY-MM-DD]
Reviewer: [name / skill run]
Brand: [slug, from brand-brain]

### BLOCKERS (must resolve before launch)
- [ ] [Item ID] — [description] — Owner: [name]

### RECOMMENDED FIXES (strong advisory)
- [ ] [Item ID] — [description] — Owner: [name]

### FYI NOTES (logged, no action required before launch)
- [Item ID] — [description]

### VERDICT
[ ] READY TO LAUNCH  (zero blockers)
[ ] CONDITIONAL — resolve blockers listed above, then confirm
[ ] DO NOT LAUNCH  (critical blockers require substantial rework)

Signed off by: _______________  Date: _______________
```

---

## Principles

- **Brand-brain loads first.** No checklist items touching copy, creative, or offer are evaluated before the brand context is confirmed.
- **BLOCKER means BLOCKER.** Do not soft-pedal a broken pixel or missing legal line as a "recommendation." If it can prevent attribution, expose legal risk, or misroute a user, it is a BLOCKER.
- **Compose, don't reimplement.** UTM validation is `utm-parameter-bulk-builder`'s job. Tracking pitfall detection is `data-qa-measurement-gotcha-checker`'s job. Invoke them; don't rebuild them.
- **Only real proof.** Any claim in copy that the brand hasn't confirmed gets marked `[verify]` — not quietly passed.
- **Channel-specific is non-negotiable.** A push-notification checklist looks nothing like a paid-search checklist. Generate only the modules that apply; don't hand the user a 90-item generic list for a single-channel email.
- **Artifact persists.** The signed QA sheet saves to `./ops/qa/[slug]-qa-[YYYY-MM-DD].md` so it can be referenced in post-mortems and audit trails.

---

## What Not to Do

- Don't call a campaign "ready to launch" while any BLOCKER is unresolved.
- Don't generate all six channel modules for a single-channel campaign — noise hides real blockers.
- Don't re-derive UTM conventions from scratch; call `utm-parameter-bulk-builder`.
- Don't invent legal requirements for jurisdictions the brand hasn't confirmed; flag as `[verify jurisdiction]`.
- Don't skip the brand-brain call even if the user provides copy — the banned-word and voice check still needs to run.
- Don't treat this as a one-time artifact — campaigns evolve; re-run for any material change to audience, creative, or destination.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called; voice + banned words + real proof loaded before evaluating any copy item?
- Brief confirmed (or `campaign-brief-builder` invoked and output used as input)?
- Correct channel modules selected and only those modules — no irrelevant rows?
- Every item has a severity classification: BLOCKER / RECOMMEND / FYI?
- All BLOCKER items explicitly listed in the verdict section with owner assignments?
- UTM coverage verified (via `utm-parameter-bulk-builder` or explicit confirmation)?
- Conversion/tracking event confirmed firing — not just "should be set up"?
- Verdict section populated with one of: READY / CONDITIONAL / DO NOT LAUNCH?
- QA artifact saved to `./ops/qa/[campaign-slug]-qa-[YYYY-MM-DD].md`?
