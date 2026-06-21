---
name: webinar-event-campaign-planner
description: >
  Webinar and event campaign planner — turns event goals, audience, budget, and date into a
  complete campaign plan with a phased promo timeline, channel mix, team role assignments,
  logistics checklist, and a sequenced pre/post activity calendar. Covers virtual webinars,
  hybrid events, in-person field events, and conference sponsorships. Calls brand-brain for
  voice and positioning, delegates email copy to webinar-email-sequence-writer, landing-page
  copy to event-registration-on-demand-page-copywriter, social posts to
  event-social-promotion-pack, and recap content to event-recap-blog-post-writer so the full
  event-content stack is covered without duplication. Produces a structured, reusable plan
  artifact saved to ./events/. Use when the user says "plan a webinar," "help me promote an
  event," "build a campaign around my conference session," "webinar promotion timeline,"
  "event marketing plan," "field event playbook," or hands over an event brief and asks what
  to do next.
---

# Webinar & Event Campaign Planner

An event without a campaign is a meeting. This skill builds the campaign: the promotion timeline, the channel mix, the copy brief stack, the logistics gate list, and the post-event follow-up calendar — all sized to the event type, audience, and resources available.

It does not write email copy, landing-page copy, or social posts itself. It orchestrates the siblings that do — so the output is a coordinated plan, not a pile of unconnected assets.

---

## Skills this calls

| Skill | When |
|---|---|
| **`brand-brain`** | Step 0 — always. Loads voice, ICP, offer/destinations, and proof before a single word is written. |
| **`icp-persona-builder`** | When the attendee persona is missing or underdeveloped; returns the registrant profile the campaign targets. |
| **`campaign-brief-builder`** | Produces the internal campaign brief (objective, KPIs, channel spec) from the event goals — call before allocating budget. |
| **`subject-line-preview-text-optimizer`** | Runs on every promotional email subject line before they go to the sequence writer. |
| **`webinar-email-sequence-writer`** | Writes all pre-event reminder emails, replay announcement, and post-event nurture; receives the plan's timeline as input. |
| **`event-registration-on-demand-page-copywriter`** | Writes reg-page and replay-page copy; receives the headline hook and benefit bullets from this plan. |
| **`event-social-promotion-pack`** | Generates pre-event, day-of, and post-event social posts across LinkedIn / X / Instagram. |
| **`event-recap-blog-post-writer`** | Produces the 600–900 word recap post from agenda, quotes, and Q&A data after the event. |
| **`cta-variant-generator`** | Called for the registration CTA, replay CTA, and any paid/organic CTA that needs a battery of variants. |
| **`headline-hook-generator`** | Used to craft the event's positioning headline and email subject line foundations. |
| **`lead-nurture-drip-builder`** | Hands off attendee and no-show segments to a structured nurture flow after the event closes. |
| **`gtm-launch-planner`** | Optional — if the event is tied to a product or feature launch, this runs first to align messaging. |
| **`content-repurposer-atomizer`** | Post-event: turns the recording and Q&A into clips, quote graphics, and secondary content. |
| **`a-b-multivariate-test-designer`** | When the team wants to test reg-page variants or subject lines during the promo window. |
| **`channel-roi-scorecard`** | Post-event scorecard: cost per registrant and attendee by channel, ROAS on sponsored placements. |

---

## How a run works

```
Step 0  Load brand          ──► brand-brain (voice, ICP, positioning, offer destinations)
Step 1  Intake & classify   ──► event type + audience + budget + date → scope the plan
Step 2  Build the timeline  ──► T-minus framework anchored to event date
Step 3  Assign channels     ──► channel mix + team roles + budget split
Step 4  Delegate copy work  ──► call sibling skills for each content asset
Step 5  Logistics gate list ──► registration, tech, legal, speaker, run-of-show
Step 6  Post-event calendar ──► replay, follow-up segments, nurture handoff, recap content
Step 7  Save the plan       ──► ./events/<slug>-campaign-plan.md
```

---

## Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`) before producing any plan output. Use the returned digest: voice adjectives and banned words override all copy decisions; the ICP shapes the registrant persona brief; offer destinations anchor the registration and replay CTA destinations; real proof informs the "why attend" value case.

**Fallback if brand-brain is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`. If none exists, ask the user for: brand voice in three adjectives, banned words/phrases, ICP one-liner, and registration destination URL — then proceed. Prefer the call.

---

## Step 1 — Intake and classify

Collect (or infer from what the user provides):

| Input | Why it matters |
|---|---|
| Event type | Webinar / virtual summit / roundtable / in-person / hybrid / conference sponsorship — drives template choice |
| Audience | Existing customers / new prospects / mix; awareness stage |
| Event date | Anchors every T-minus milestone |
| Registration goal | Target headcount; paid or free; gating logic |
| Budget | $0 (owned only) / small ($500–$5K) / medium ($5K–$25K) / large (>$25K) — drives channel mix |
| Speakers/hosts | Internal only / external guest — affects outreach timeline |
| Promo window | Days until event; if < 10 days, invoke compressed track |

Ask only for what the user has not already provided. If the event date is missing, ask for it first — nothing can be scheduled without it.

---

## Step 2 — T-Minus promotion timeline (the core framework)

The **T-Minus Event Promotion Framework** divides the campaign into four phases anchored to the event date. Compress phases proportionally when the promo window is short (< 3 weeks: collapse Awareness + Consideration into one phase).

### Phase 1 — Awareness (T-6 to T-4 weeks)
Goal: create demand for the registration. Target cold and warm audiences; hook with the problem, not the format.
- Announce on owned social; publish blog teaser or thought-leadership primer
- Send first email to list segments most likely to convert (call `subject-line-preview-text-optimizer` on each subject line)
- Launch paid social awareness (LinkedIn / Meta) if budget allows
- Speaker or co-host begins personal promotion

### Phase 2 — Consideration (T-4 to T-2 weeks)
Goal: convert interest into registrations. Lean into social proof, speaker credibility, and specificity of takeaways.
- Registration launch email to full eligible list
- Social posts featuring speaker headshots, agenda details, and a "what you'll learn" hook
- Organic SEO: reg-page live, schema markup, internal links from relevant posts
- Partner / co-sponsor amplification if applicable
- Paid retargeting: site visitors, engaged email subscribers

### Phase 3 — Urgency & Reminder (T-2 weeks to event day)
Goal: close the fence-sitters and reduce day-of no-shows.
- 2-week reminder email to non-registrants (new hook — don't resend the same email)
- 48-hour and 24-hour reminder to registered attendees (logistics + link)
- 1-hour countdown email/push on event day
- Day-of social post with live-join link
- Final paid push to highest-intent audience (video view retargeting, email-list upload)

### Phase 4 — Post-Event (Day 0 through T+4 weeks)
Goal: extend shelf-life; convert no-shows; nurture all segments into pipeline.
- Same-day replay announcement to all registrants
- Segmented follow-up by attendance: attended / no-show / replay-only (call `lead-nurture-drip-builder`)
- Recap blog post published within 72 hours (call `event-recap-blog-post-writer`)
- Content atomization: clips, quote graphics, carousel posts (call `content-repurposer-atomizer`)
- ROI scorecard: cost per registrant and attendee by channel (call `channel-roi-scorecard`)

---

## Step 3 — Channel mix and team roles

Allocate channels to budget tier. Assign a named role (not a job title) to each stream.

| Budget tier | Owned | Earned | Paid |
|---|---|---|---|
| $0 | Email, organic social, blog, in-product | Speaker/partner amplification, community posts | — |
| Small ($500–$5K) | + push notification, SEO landing page | + 1 podcast pitch | LinkedIn lead-gen form; retargeting only |
| Medium ($5K–$25K) | + in-app banners, dedicated send | + media mention, co-host audience | LinkedIn + Meta awareness + retargeting |
| Large (>$25K) | + paid newsletter sponsorship, dedicated send series | + PR push, influencer/analyst partnership | Full funnel: awareness + consideration + retargeting across channels |

Assign three roles minimum:
- **Campaign driver** — owns timeline, briefs, and cross-functional coordination
- **Content owner** — produces or reviews all copy assets
- **Technical owner** — owns reg platform, UTMs, tracking, run-of-show logistics

---

## Step 4 — Delegate copy work

After the plan structure is confirmed, call sibling skills as needed. Pass the plan's event details, brand digest, and timeline as context to each:

- **`headline-hook-generator`** → primary event headline and email subject-line foundation
- **`event-registration-on-demand-page-copywriter`** → reg-page and replay-page copy
- **`webinar-email-sequence-writer`** → full pre/post email sequence
- **`event-social-promotion-pack`** → social posts per phase
- **`cta-variant-generator`** → registration CTA battery (Quick mode default; Battery on request)

Do not write email copy, landing-page copy, or social copy directly in this plan — delegate and reference the output artifacts.

---

## Step 5 — Logistics gate list

Flag anything not confirmed. Each item needs an owner and a deadline before the plan is "green."

- [ ] Registration platform configured and tested (form, confirmation email, calendar invite)
- [ ] Webinar tech stack confirmed (host platform, backup dial-in, recording enabled)
- [ ] Speakers confirmed in writing with slide deadline
- [ ] Run-of-show document drafted and shared with all speakers
- [ ] UTM naming convention set across all promo links (call `utm-parameter-bulk-builder` if needed)
- [ ] CRM/ESP segment list built and suppression list applied
- [ ] Legal/compliance review done for any gated offer or sweepstakes element
- [ ] Replay hosting and access link confirmed
- [ ] Attendee data handling compliant with GDPR/CAN-SPAM/applicable jurisdiction

---

## Step 6 — Post-event segment calendar

On event day, the audience splits into four segments. Map each to a follow-up action.

| Segment | Timing | Action |
|---|---|---|
| **Attended live** | Same day | Thank-you + replay link + next-step CTA |
| **Registered / no-show** | Day 1 | Replay announcement with "what you missed" subject line |
| **Replay watched** | Day 2–3 | Engagement follow-up: resource link, offer, or CTA |
| **Registered / no engagement** | Day 5 | Final nurture touch; route to `lead-nurture-drip-builder` |

All four feed the post-event nurture flow. Hand off to `lead-nurture-drip-builder` with segment tag, event topic, and offer/CTA for the next step.

---

## Step 7 — Save the plan

Save the full plan to `./events/<event-slug>-campaign-plan.md`. Include: event brief summary, T-minus timeline with dates filled in, channel assignments, team roles, logistics checklist status, post-event segment calendar, and links to any delegated copy artifacts once generated.

Never save to the skill folder. Never overwrite without diffing.

---

## Principles

- **Timeline is the spine.** Every channel action, copy brief, and logistics item anchors to a specific T-minus date. Vague "post a few weeks before" plans do not get executed.
- **Delegate copy; own the plan.** This skill coordinates. It does not write emails or landing pages — it briefs the skills that do.
- **Phase compression is explicit.** A 7-day promo window is a different plan than a 6-week one. Name the compression; don't silently skip phases.
- **Segment the post-event immediately.** The follow-up is where most event ROI is won or lost. Attended vs. no-show vs. replay-only are different conversations from day one.
- **Logistics gates are blockers.** An unchecked gate item (no speaker confirmation, broken UTM, untested reg flow) can zero out an event's ROI regardless of promotional spend. Surface them early, assign owners.
- **Brand-brain first, always.** No campaign output before the active brand is loaded.

---

## What not to do

- Don't write email copy, social posts, or landing-page copy in this plan — call the sibling skills.
- Don't skip the T-minus framework because the event is "small" — a compressed version still needs all phases.
- Don't invent speaker credentials, customer proof, or attendance numbers — use only confirmed facts or mark `[verify]`.
- Don't produce a "generic webinar checklist" — every output should be scoped to the specific event type, audience, and budget.
- Don't assume the post-event plan is out of scope — it is the highest-leverage phase and belongs in every plan.
- Don't save to the skill folder; use `./events/`.

---

## Quality checklist

- [ ] `brand-brain` called and active brand loaded before any plan output?
- [ ] All four T-minus phases present (or explicitly compressed with rationale)?
- [ ] Channel mix matched to budget tier with named role owners?
- [ ] Sibling skills called for each copy asset (email, reg page, social, recap) — not written inline?
- [ ] Logistics gate list generated with owner fields?
- [ ] Post-event segments defined with follow-up action per segment?
- [ ] Plan saved to `./events/<event-slug>-campaign-plan.md`?
- [ ] No invented proof, credentials, or numbers — everything confirmed or `[verify]`?
