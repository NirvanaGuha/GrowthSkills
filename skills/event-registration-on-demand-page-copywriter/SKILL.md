---
name: event-registration-on-demand-page-copywriter
description: >
  Writes the full page copy for live-event registration pages and on-demand replay pages —
  headline, subhead, benefit bullets, speaker bios, agenda snapshot, social proof, FAQs,
  and all CTA copy — given a webinar topic, speakers, date, and key takeaways. Handles both
  modes: a live-registration page (urgency-forward, seats/time-limited framing) and an
  on-demand page (value-forward, available-now framing). Pulls brand voice, ICP, offer
  mechanics, and real proof from `brand-brain`. Delegates CTAs to `cta-variant-generator`,
  SEO optimisation to `on-page-seo-optimizer`, and speaker/social proof to `proof-vault`
  when installed. Saves the finished copy to `./events/<slug>/`. Use when the user says
  "write my webinar registration page," "landing page for the event," "on-demand replay
  page," "event sign-up copy," "webinar page copy," "write the registration landing page,"
  or hands over a speaker lineup and asks for page copy.
---

# Event Registration & On-Demand Page Copywriter

Registration pages kill webinar pipeline before a word of the content lands. Most lose attendees to vague headlines, speaker bios that read like LinkedIn about-sections, and a CTA that just says "Register Now." This skill writes page copy that converts a cold skim into a committed registration or replay watch — using the brand's voice, the ICP's actual pains, and a framework that turns "another webinar" into a must-attend event.

Two modes. One framework. All copy on-brand, all proof verified.

---

## Skills this calls

- **`brand-brain`** (required, always first) — loads active brand voice, banned words, ICP, real proof, and offer context.
- **`cta-variant-generator`** — writes the hero CTA label + microcopy and a low-commitment secondary; called after brand context loads.
- **`proof-vault`** — pulls verified speaker credentials, customer proof, and brand authority signals; synthesize inline if absent.
- **`on-page-seo-optimizer`** — optimizes title tag, meta description, and H1/H2 hierarchy for the target keyword; call when the user wants the page indexed.
- **`headline-hook-generator`** — generates alternate headline angles when the user wants A/B options or is unsatisfied with the first pass.
- **`icp-persona-builder`** — surfaces sharper ICP vocabulary and pain language when the brand has no personas yet.
- **`subject-line-preview-text-optimizer`** — if the user also needs the confirmation/reminder emails, hand off the copy inputs to this skill.
- **`content-repurposer-atomizer`** — if the on-demand page copy needs to produce social cards, clips, or email snippets for replay promotion.

---

## How a run works

```
Step 0  Load brand + ICP       ──► call brand-brain; confirm ICP awareness stage
Step 1  Clarify inputs         ──► collect missing event facts (30-sec question pass)
Step 2  Determine mode         ──► Live Registration | On-Demand Replay | Both
Step 3  Run the RSVP Framework ──► build each copy section in order
Step 4  Delegate CTAs          ──► call cta-variant-generator for hero + secondary
Step 5  SEO pass (if needed)   ──► call on-page-seo-optimizer when indexing matters
Step 6  Self-review + deliver  ──► quality checklist, then present with save
```

---

## Step 0 — Load brand (always first)

**Invoke `brand-brain`** before writing a single line. It returns the digest (voice adjectives, banned words, ICP + awareness tendency, real proof, positioning, offer mechanics). Obey voice + banned-words as hard overrides; mark unconfirmed proof `[verify]`.

**Fallback:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if neither exists, ask the user to install `brand-brain` or answer a 5-question inline setup (brand name · ICP role + problem · 3 voice adjectives + banned words · primary proof point · offer/destination) before proceeding.

---

## Step 1 — Minimum inputs needed

Collect anything missing before writing. One focused pass, not a lengthy interview:

| Input | Notes |
|---|---|
| Event topic / working title | The outcome attendees care about — not the tactic |
| Format + date/time | Webinar, panel, workshop; live date or "available now" |
| Speakers | Name, title, company, one notable credential each |
| 3–5 key takeaways | Specific and outcome-framed ("how to X in Y situation") |
| Target attendee role | Disambiguate if the ICP has multiple buyer personas |
| Registration / replay URL | Needed for CTA destination |
| SEO target keyword | Optional; triggers `on-page-seo-optimizer` |

If the user gives a rough brief, extract what's there and ask only for the gaps.

---

## Step 2 — Pick the mode

- **Live Registration** — urgency is a real asset; scarcity framing (seats, date, live Q&A) is honest and available. Written for problem-aware to solution-aware ICP. Conversion goal: form fill.
- **On-Demand Replay** — urgency is gone; value must carry the page. Frame around instant availability and outcome specificity. Conversion goal: email capture or direct watch.
- **Both** — write the live page first, then produce the on-demand variant as a diff (swap headline, remove date/seats, update CTA and FAQs). Don't rewrite from scratch.

---

## RSVP Framework (our working model)

Every high-converting event page hits five gates in order. A reader who clears all five registers. Copy that skips a gate loses them.

**R — Relevance** (above the fold)
State the outcome the attendee will leave with, for whom, and why now. The headline answers "what will I be able to do after this that I can't do today?" The subhead names the audience and the situation. If the reader isn't identified within five seconds, they bounce.

**S — Stakes** (the why-it-matters block)
Make the cost of the status quo vivid without manufactured fear. One short paragraph (or 2–3 punchy bullets) that surfaces the pain the webinar solves. Draw from the ICP's vocabulary (from `brand-brain` or `icp-persona-builder`) — never corporate abstraction.

**V — Value** (the takeaways + agenda block)
Three to five specific, numbered takeaways formatted as outcomes, not topics. "How to reduce cart abandonment by using behavioral triggers" beats "Behavioral triggers for eCommerce." Include an agenda snapshot only if the session has distinct segments worth naming; otherwise skip it to avoid padding.

**P — Proof** (speaker bios + social proof)
Two to four sentences per speaker: relevant credential (what they've done that makes them credible on *this exact topic*), one notable achievement, one line on what they'll cover. Pull from `proof-vault` if installed; mark unconfirmed credentials `[verify]`. Add a social proof strip (attendee count, past event quote, or brand logos) only when real figures exist.

**CTA — The ask** (hero + secondary CTA)
Call `cta-variant-generator` with the brand context, awareness stage (typically solution-aware for a known event), and conversion goal (form fill or email capture). Request: one hero CTA label + friction-reducer microcopy + one low-commitment secondary. For live pages, the microcopy addresses the time/effort objection ("60 minutes · live Q&A included · recording sent if you can't make it"). For on-demand, it addresses the commitment objection ("Watch now · 47 min · pause anytime").

---

## Live Registration vs. On-Demand: key copy differences

| Section | Live Registration | On-Demand Replay |
|---|---|---|
| Headline frame | "Join us [date] to learn…" / outcome | "Watch now: learn…" / instant value |
| Urgency | Real: date, live Q&A, limited seats | None — remove fake scarcity entirely |
| Stakes block | Builds toward the live session | Builds toward the takeaway value |
| CTA label | "Save my seat" / "Reserve my spot" | "Watch the replay" / "Get instant access" |
| FAQ #1 | "Will there be a recording?" | "How long is the replay?" |
| Social proof | Registration count if real | View count, testimonials if real |

---

## Principles

- **RSVP gates in order.** A page that skips to CTA before earning relevance converts at banner-ad rates.
- **Outcomes, not topics.** Every takeaway, every speaker bio line, and the headline itself names what the attendee gains — not what will be discussed.
- **Honest urgency.** Live pages can use the date and live Q&A as urgency. On-demand pages cannot invent scarcity. False countdown timers or "limited replays" on an always-on page are banned.
- **Speaker bios earn credibility, not LinkedIn vanity.** One credential that makes this speaker right for this topic; one line on what they'll cover. Nothing more.
- **Brand-brain first, always.** Voice, banned words, and real proof are overrides — not suggestions. Mark anything unconfirmed `[verify]`.
- **No boilerplate filler.** "Join us for an exciting discussion" and "industry experts" are banned by default. Name the thing.

---

## What not to do

- Don't write anything before `brand-brain` returns the active brand.
- Don't invent speaker credentials, testimonials, attendee counts, or conversion stats — use `[verify]`.
- Don't use fake or manufactured urgency on an on-demand page.
- Don't reuse the live-page headline verbatim for the on-demand page — the value frame is different.
- Don't write a 12-point agenda when three outcome bullets would convert better.
- Don't stack multiple primary CTAs; one hero + one low-commitment secondary is the ceiling.
- Don't write FAQs that exist to pad word count; every FAQ must answer a real objection (use `objection-library-builder` if available).
- Don't re-implement ICP resolution, brand scanning, or voice definition here — `brand-brain` owns that.

---

## Output format

```
## Event Registration Page — [Event Title]
Mode: [Live / On-Demand / Both]
Brand: [slug, via brand-brain]
ICP: [role + awareness stage]

### Page copy

**Title tag:** [60 chars max — if SEO pass requested]
**Meta description:** [155 chars max — if SEO pass requested]

**Eyebrow:** [optional — event type / series label]
**Headline:** [outcome-forward, ICP-specific]
**Subhead:** [audience + situation + date or "available now"]

**Stakes block:** [1 short para or 2–3 bullets]

**Takeaways (3–5):**
1. …
2. …
3. …

**Speakers:**
[Name] — [Title, Company]: [2–4 sentences per RSVP framework]

**CTA block:** [from cta-variant-generator]
  Hero: [label + microcopy]
  Secondary: [low-commitment label]

**FAQs (3–5):** [real objections only]

---
[On-demand diff, if Both mode requested]
```

Save output to `./events/<event-slug>/registration-copy.md` (and `./events/<event-slug>/on-demand-copy.md` if both modes).

---

## Quality checklist (self-review before presenting)

- `brand-brain` called; active brand loaded; voice + banned-words honored; unconfirmed proof marked `[verify]`?
- Headline states an outcome for a named audience — not a topic or a vague promise?
- RSVP gates hit in order: Relevance → Stakes → Value → Proof → CTA?
- Takeaways are outcome-framed, not topic-framed?
- Speaker bios establish credibility for *this topic*, not a general LinkedIn summary?
- `cta-variant-generator` called; hero CTA + friction-reducer microcopy + one secondary delivered?
- Live page uses real urgency only (date, Q&A); on-demand page has no manufactured scarcity?
- FAQs address real objections — none are filler?
- Output saved to `./events/<slug>/`?
