---
name: event-end-to-end-orchestrator
description: >
  The flagship "event team in a box." Takes one event brief (virtual webinar or live event) and runs the
  ENTIRE lifecycle — plan → assets → run → recap → ROI — by chaining the library's specialist event skills
  end to end into one coordinated kit a buyer can run cold. It does NOT re-implement any stage: it invokes
  `webinar-event-campaign-planner` for the master plan, then fans out to
  `event-registration-on-demand-page-copywriter`, `webinar-email-sequence-writer`, `event-social-promotion-pack`,
  and `webinar-script-run-of-show-generator` for the asset wave, then `event-recap-blog-post-writer` for the
  recap, then `event-roi-calculator` to close the loop — each via the Skill tool, with explicit handoffs and
  human-approval gates between waves. Loads brand context once via `brand-brain` so every asset is on-voice,
  ICP-aligned, and uses real proof. Output is a single dated kit folder under the user's CWD. Use when the user
  says "run my whole webinar," "plan and build my event," "event end to end," "I have a webinar in 3 weeks,"
  "give me the full event kit," "save-the-date through follow-up," "everything for my virtual event," or hands
  over an event brief and wants the campaign, the assets, the run-of-show, the recap, and the ROI in one pass.
  Orchestrates and assembles — it does not invent attendance, pipeline, or quotes.
---

# Event End-to-End Orchestrator

One brief in, a whole event out. Point this at a webinar or live-event brief and it produces what a marketer would otherwise spend a week assembling: the campaign plan, the registration + on-demand page, the full email sequence, the social promo pack, the host's run-of-show script, the recap blog post, and the ROI model — all on one brand voice, all cross-referenced, all saved to one dated folder.

This is an **orchestrator**, not a writer. It owns sequencing, handoffs, gates, and assembly. Every actual deliverable is produced by the specialist skill that owns it, invoked through the Skill tool. The orchestrator never re-drafts a page or a sequence itself — if a stage's output is weak, it loops that stage, it does not patch the output by hand.

---

## Skills this calls

Brand context (always first):
- **`brand-brain`** (required) — resolves + loads the active brand's voice, ICP, offer, proof, banned words. The orchestrator never writes `brand.md`; it reads what `brand-brain` serves.

The event pipeline (the spine — invoke in this order):
1. **`webinar-event-campaign-planner`** — the master plan: goals, ICP, promo timeline, channel mix, asset checklist, owners, dates.
2. **`event-registration-on-demand-page-copywriter`** — registration page copy now + on-demand/replay page copy for after.
3. **`webinar-email-sequence-writer`** — the full lifecycle: invite → reminders → live-now → no-show replay → attendee follow-up.
4. **`event-social-promotion-pack`** — speaker cards, countdown posts, quote graphics, live-tweet beats, post-event clips brief.
5. **`webinar-script-run-of-show-generator`** — minute-by-minute host script: intro, segments, poll/Q&A prompts, CTA moments, cues.
6. **`event-recap-blog-post-writer`** — post-event recap article with key takeaways and a replay/on-demand CTA.
7. **`event-roi-calculator`** — costs + registrants + attendance + pipeline → cost-per-lead, cost-per-opp, ROI vs benchmark.

Optional helpers (call only when present and the stage needs them; synthesize inline if absent):
- `cta-variant-generator` for the registration + replay CTAs · `proof-vault` for real proof/logos · `subject-line-preview-text-optimizer` for email subjects · `utm-parameter-bulk-builder` for tagged promo links · `bulk-scheduling-csv-builder` to turn the social pack into a scheduler upload · `lifecycle-email-push-copy-reviewer` as the email-wave reviewer · `campaign-qa-launch-checklist-generator` for the pre-launch gate.

---

## How a run works

```
Stage 0  Brand          ── brand-brain ─────────────► brand digest + brand.md path
Stage 1  PLAN           ── webinar-event-campaign-planner ─► master plan  ──[HUMAN GATE A]──►
Stage 2  ASSETS (∥)     ── reg/on-demand page · email seq · social pack · run-of-show ──[REVIEW + HUMAN GATE B]──►
  ░░░ EVENT HAPPENS — operator runs the show; returns attendance + recording/notes ░░░
Stage 3  RECAP          ── event-recap-blog-post-writer ─► recap post + on-demand page activated
Stage 4  ROI            ── event-roi-calculator ──────► ROI summary  ──► assemble kit ──[HUMAN GATE C]
```

### Stage 0 — Load the brand (always first)
**Invoke `brand-brain`** (Skill tool, `skill: brand-brain`), passing the request + any named brand. Use the returned digest — voice, banned words, ICP + awareness, offer mechanics + destination URLs, real proof — as a hard override on every downstream stage, and pass it forward in every handoff so no specialist re-derives it. Do not produce any asset until it returns.

Also branch on **event type** here, because it changes the asset set:
- **Virtual (webinar / livestream / on-demand)** — the full spine applies, including the on-demand/replay page and the no-show "watch the replay" path. This is the default.
- **Live / field event (in-person, hybrid, dinner, booth)** — keep all seven stages, but the run-of-show becomes an on-site agenda (registration desk, sessions, networking blocks), the "live-now" email becomes a "see you there / logistics" send, and the on-demand page becomes a "photos + slides + recording" recap page. Same skills, retargeted inputs — never a different pipeline.

**Fallback if `brand-brain` is absent or returns no brand:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; else ask the user for the few essentials (what the brand is · ICP + awareness · offer + destination URL · 3 voice adjectives + banned words), then proceed. This thin read is the correct fallback — do not reimplement brand scanning, interviewing, or storage here.

### Stage 1 — PLAN (gate-critical)
Normalize the brief into a planner input: event type (webinar vs live), topic + promise, date/time + timezone, speakers, goal (regs / pipeline / activation), audience segment, promo window, channels, owners. **Invoke `webinar-event-campaign-planner`** with the brief + brand digest.
- **Handoff out:** the master plan becomes the single source of truth for every later stage — the exact event title/promise, the date/timezone, the speaker list, the goal metric + target, the promo timeline, and the asset checklist. Every Stage-2 skill is fed these so titles, dates, and the offer match across all assets.
- **[HUMAN GATE A]** Show the plan and the proposed asset list. Ask the operator to confirm date/title/goal and approve the build. **Do not start Stage 2 until confirmed** — a wrong date or title here propagates into every asset.

### Stage 2 — ASSETS (parallel, then reviewed)
Run the four asset skills, each fed the approved plan + brand digest. The page, email, social, and script jobs are independent → **launch them in parallel** (one Skill call each, in a single batch where the harness allows). Cross-wire the shared facts so nothing drifts:
- **`event-registration-on-demand-page-copywriter`** ← plan (title, promise, speakers, date, what-you'll-learn, CTA destination). Returns registration-page copy + on-demand/replay-page copy. → its headline + value bullets feed the social pack and the email invite so message-match holds; if `cta-variant-generator` is present, route the register/replay CTA through it.
- **`webinar-email-sequence-writer`** ← plan (timeline, segments) + the registration page's promise + CTA. Returns invite → reminder(s) → live-now → no-show → follow-up. → subjects routed through `subject-line-preview-text-optimizer` when present; the follow-up email reuses the recap/on-demand link slot reserved in Stage 3.
- **`event-social-promotion-pack`** ← plan (speakers, dates, hook) + the page headline. Returns countdown/teaser/speaker/quote posts + live beats. → optionally piped to `bulk-scheduling-csv-builder` for a scheduler-ready CSV; links UTM-tagged via `utm-parameter-bulk-builder` when present.
- **`webinar-script-run-of-show-generator`** ← plan (duration, speakers, segments, the ONE registration promise + CTA moment). Returns the minute-by-minute host script with poll/Q&A/CTA cues. → the CTA moment must pay off the same promise the page + emails made.
- **REVIEW (separate lane — never self-approve):** route the email wave through `lifecycle-email-push-copy-reviewer` (or, if absent, a fresh reviewer pass) and the whole pack through `campaign-qa-launch-checklist-generator` when present. If a reviewer returns **Revise**, **re-invoke that specific stage** with the reviewer's notes and the brand digest, then re-review — loop until it passes or two loops elapse, then surface the blocker to the human rather than shipping weak copy.
- **[HUMAN GATE B]** Present the assembled pre-event kit (page + emails + social + script + QA result) for approval to launch promotion. The operator then runs promotion and, after the event, returns **attendance numbers + the recording link or speaker notes**.

### Stage 3 — RECAP (post-event)
With the recording/notes + attendance in hand, **invoke `event-recap-blog-post-writer`** ← plan + actual takeaways + on-demand page URL. Returns the recap post; activate the Stage-2 on-demand/replay page copy and drop its live URL into the email follow-up's reserved link slot and the social "watch the replay" beat. → the recap + on-demand metrics feed Stage 4.

### Stage 4 — ROI (close the loop)
**Invoke `event-roi-calculator`** ← production + promo costs + registrants + attendance + influenced pipeline / closed revenue (operator-supplied). Returns cost-per-lead, cost-per-opp, ROI vs benchmark. Mark any operator-unknown input `[verify]` and compute a range rather than inventing a number. → feeds the kit's executive summary and a "what to change next time" note.

**Assemble + [HUMAN GATE C]:** compile everything into the kit folder (below), write the index + one-paragraph exec summary (goal vs result + ROI), and hand it back for final sign-off.

---

## Bundled deliverable

Save to a **project-relative** path in the user's CWD (never the skill folder), one dated folder per event:

```
./events/<event-slug>-<YYYY-MM-DD>/
  00-INDEX.md                  ← kit map + exec summary (goal vs result, ROI headline) + brand slug + open [verify] items
  01-campaign-plan.md          ← webinar-event-campaign-planner
  02-registration-page.md      ← event-registration-on-demand-page-copywriter (reg + on-demand copy)
  03-email-sequence.md         ← webinar-email-sequence-writer (+ reviewer notes resolved)
  04-social-pack/              ← event-social-promotion-pack (+ optional scheduling CSV)
  05-run-of-show.md            ← webinar-script-run-of-show-generator
  06-recap-post.md             ← event-recap-blog-post-writer (post-event)
  07-roi-summary.md            ← event-roi-calculator (post-event)
```
`00-INDEX.md` records which skill produced each artifact, the date/title/goal locked at Gate A, the approval state of each gate, and every unresolved `[verify]`. Pre-event runs stop after `05` with `06`/`07` stubbed and clearly marked "pending event."

---

## A real run, start to finish (example order)

A buyer hands over: *"Webinar on cutting churn with triggered notifications, 3 weeks out, two speakers, goal = 300 regs and 5 sales conversations."* The orchestrator runs:

1. **`brand-brain`** → loads the brand voice, ICP (mid-market eCommerce), offer + replay destination, real proof. Event type = virtual → full spine.
2. **`webinar-event-campaign-planner`** → master plan: title locked, date + timezone, 3-week promo timeline, channel mix, asset checklist, goal = 300 regs. → **Gate A**: operator confirms title + date.
3. **Parallel asset wave**, all fed the locked plan: **`event-registration-on-demand-page-copywriter`** (reg + replay copy) · **`webinar-email-sequence-writer`** (invite → 2 reminders → live-now → no-show → follow-up) · **`event-social-promotion-pack`** (countdown + speaker + quote posts) · **`webinar-script-run-of-show-generator`** (60-min host script with poll + Q&A + CTA cues). Subjects → `subject-line-preview-text-optimizer`; CTAs → `cta-variant-generator`; links → `utm-parameter-bulk-builder`.
4. **Review lane**: email wave → `lifecycle-email-push-copy-reviewer`; one reminder returns **Revise** ("CTA buried") → re-invoke the email writer with that note → re-review → pass. Whole pack → `campaign-qa-launch-checklist-generator`. → **Gate B**: operator approves, promotes, runs the event, returns "312 regs, 141 attended, recording link, 6 sales convos."
5. **`event-recap-blog-post-writer`** → recap post + activated on-demand page; replay URL dropped into the follow-up email's reserved slot.
6. **`event-roi-calculator`** → cost-per-reg, cost-per-opp, ROI vs benchmark (any unknown cost → `[verify]` range). → **Gate C**: assembled kit + exec summary handed back.

Note the loop in step 4 and the two parallel-then-gated waves — that staging is the orchestrator's actual job.

---

## Principles

- **Orchestrate, don't re-implement.** Each artifact is produced by its owning skill via the Skill tool. The orchestrator sequences, hands off, gates, reviews, and assembles — it never re-drafts a stage's output by hand.
- **Brand first, brand once.** `brand-brain` resolves voice/ICP/offer/proof at Stage 0; its digest is passed into every stage so nothing re-derives it and nothing drifts off-voice.
- **One source of truth.** The approved plan owns the title, date/timezone, promise, speakers, and goal. Every asset inherits them; the same registration promise is paid off on the page, in the emails, in the social, and at the script's CTA moment.
- **Gate the irreversibles.** Human approval before the build (A), before launching promotion (B), and at final assembly (C). Promotion is one-way — confirm before it goes out.
- **Author and review are separate lanes.** The writer skill drafts; a different reviewer pass evaluates. On Revise, loop the owning stage — never self-approve in the same pass.
- **Parallel where independent, sequential where dependent.** Stage-2 assets run in parallel; recap and ROI wait on real post-event data.
- **Truth discipline.** Real proof, real numbers, or `[verify]`. Never invent attendance, pipeline, ROI, or quotes; compute ROI as a range when inputs are unknown.

## What not to do

- Don't produce any asset before `brand-brain` returns the active brand.
- Don't re-implement a stage — no hand-written pages, emails, scripts, recaps, or ROI math; call the owning skill.
- Don't skip Gate A — a wrong date or title there corrupts every downstream asset.
- Don't launch promotion (Gate B) or finalize the kit (Gate C) without explicit human approval.
- Don't self-approve copy; route through a reviewer skill and loop on Revise.
- Don't fabricate registrants, attendance, pipeline, revenue, ROI, or testimonial quotes — mark unknowns `[verify]`.
- Don't write the kit into the skill folder; always use the CWD `./events/...` path.
- Don't let titles/dates/CTAs drift between assets — reconcile against the plan before assembling.

## Quality checklist (self-review before final handoff)

- `brand-brain` called first; its digest passed into every stage; voice + banned-words honored throughout?
- Stage 1 plan approved at Gate A before any asset was built; date/title/goal locked and reused everywhere?
- All four Stage-2 skills invoked (not hand-written), fed the approved plan + brand digest, and run in parallel?
- The single registration promise + CTA matches across page, emails, social, and the run-of-show CTA moment?
- Email wave (and full pack) passed a separate reviewer; any Revise looped back to the owning stage, not patched inline?
- Gate B approval recorded before promotion; recap + ROI run only on real post-event data (or clearly stubbed "pending")?
- ROI uses real inputs or a `[verify]`-flagged range — no invented numbers; no invented quotes in the recap?
- Kit assembled under `./events/<slug>-<date>/` with `00-INDEX.md` mapping every artifact to its source skill + listing open `[verify]` items?
