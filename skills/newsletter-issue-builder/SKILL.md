---
name: newsletter-issue-builder
description: >
  Turns a raw content feed — links, internal updates, hot takes, product news, curated
  articles, data points — into a fully structured, on-brand newsletter issue ready to
  paste into your ESP or send through your publishing stack. Built around our house
  Story-Curation-Action (SCA) working model: each issue earns attention with a strong lede
  story, builds trust through curated signal, and converts with a single frictionless CTA.
  Brand voice, banned words, and ICP awareness level come from the shared `brand-brain`
  skill (Layer 0), so every issue sounds like the brand, not a content aggregator.
  Composes `subject-line-preview-text-optimizer` for the envelope, `cta-variant-generator`
  for the issue CTA, `headline-hook-generator` for section headers, `proof-vault` for
  inline social proof, and `lifecycle-email-push-copy-reviewer` for a final copy pass.
  Does NOT write the underlying articles — it drafts issue architecture and copy; for
  full-length article authoring call `blog-post-drafting-engine`. Saves issues to
  `./newsletters/[slug]-[YYYY-MM-DD].md` when asked.
  Trigger phrases: "write my newsletter," "draft this week's issue," "build a newsletter,"
  "newsletter draft," "compile the issue," "newsletter issue," "write the email digest,"
  "newsletter copy," "lay out the newsletter," "write the intro," "curate this into a
  newsletter," "newsletter from these links."
---

# Newsletter Issue Builder

Give it a pile of links, notes, product updates, or a content brief. Get back a structured,
on-brand newsletter issue with a punchy subject line, a lede that earns the open, curated
sections that build authority, and one clear CTA. The work is architecture and craft — not
content aggregation dressed up as copy.

Every issue follows our house **Story-Curation-Action (SCA) working model**: one owned lede that asserts
a point of view, 2–4 curated sections that prove it or extend it, and a single action that
converts the attention earned. The brand voice, audience awareness level, and proof come from
`brand-brain` — this skill does not re-derive them.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — resolves the active brand and returns voice adjectives,
  banned words, ICP + awareness tendency, offer mechanics, and real proof. All copy obeys the
  returned voice; unconfirmed claims are marked `[verify]`.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` +
  that brand's `brand.md` directly; if none exists, ask the user for brand name, ICP + awareness
  level, voice adjectives + banned words, and the newsletter's primary CTA destination before
  proceeding.
- **`subject-line-preview-text-optimizer`** (Step 4) — generates the envelope copy (subject +
  preview text) from the issue theme and lede angle. Synthesize inline if absent.
- **`headline-hook-generator`** (Step 3) — writes the lede headline and section headers.
  Synthesize inline if absent.
- **`cta-variant-generator`** (Step 5) — generates the issue's primary CTA button/link with
  1–2 alternates. Synthesize inline if absent.
- **`proof-vault`** (Step 3) — surfaces relevant proof microcopy for lede or callout blocks.
  Synthesize inline if absent.
- **`lifecycle-email-push-copy-reviewer`** (Step 6, optional) — runs a final voice/flow/CTA
  audit on the completed draft. Invoke when the user asks for a review pass or when quality
  is uncertain.

---

## How a run works

```
Step 0  Load brand        ──► brand-brain (always first; blocks all copy)
Step 1  Triage input      ──► classify, prioritize, reject weak links
Step 2  Choose mode       ──► Standard (default) | Deep (long-form lede on request) | Digest (no lede; pure curation)
Step 3  Build structure   ──► lede + sections + callouts via SCA working model
Step 4  Envelope          ──► subject-line-preview-text-optimizer
Step 5  CTA               ──► cta-variant-generator
Step 6  Review pass       ──► lifecycle-email-push-copy-reviewer (on request)
Step 7  Present / save    ──► inline or ./newsletters/[slug]-[YYYY-MM-DD].md
```

---

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns the active brand's digest:
voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning, ICP
+ awareness tendency. Do not write a single word of issue copy until it returns.

Use only the returned real proof; mark anything else `[verify]`. Honor voice + banned-words as
hard overrides over every other instruction in this skill.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
brand's `brand.md` directly; if none exists, ask the user for brand name, ICP + awareness level,
voice adjectives + banned words, and the newsletter's primary CTA destination before proceeding.

---

### Step 1 — Triage the input

Before architecting the issue, evaluate what the user handed over:

| Input signal | Action |
|---|---|
| URL or article link | Fetch title + 1-line summary; note the source |
| Internal update / product news | Flag as Owned — prioritize for lede |
| Raw notes / brain dump | Extract the assertion, not the list |
| Data point / stat | Verify source before treating as proof; else `[verify]` |
| "Here's everything, sort it out" | Pick the one story most relevant to the brand's ICP this week; lead with that |

Reject content that: contradicts the brand, is older than 3 weeks without a strong angle hook,
or is a press-release shell with no point of view. Name the rejects and why — don't silently drop.

---

### Step 2 — Choose the mode

- **Standard (default):** 200–350-word lede + 2–4 curated sections. The typical weekly/biweekly issue.
- **Deep:** 400–600-word lede with reporting-style structure (hook → context → implication → so-what);
  1–2 curated sections. Triggered by "go deep," "long-form," "I want a meaty issue," or when the
  brand's voice is explicitly editorial/opinionated.
- **Digest:** No lede. 4–8 tight curated items, each 1–3 sentences + a link. Triggered by "digest,"
  "quick links," "just the links," or when the user has no owned content this week.

When unsure, default to Standard. Offer Deep or Digest at the end.

---

## The SCA Working Model (our craft engine)

### Story — the lede

The lede is the issue's reason for existence. It earns the open; it is not a summary or an
announcement. Rules:

1. **One assertion, not a list.** The lede argues or demonstrates one thing: a trend, a
   counterintuitive fact, an earned lesson, a strong opinion. If it could start with "Here are
   N things…" it is not a lede — it is a digest intro.
2. **Open with the news-value sentence.** Bury no lede. First sentence = the most interesting
   fact or claim. (Tested against the Poynter "so what" test: if a reader could answer "so
   what?" in one second, the lede has no hook.)
3. **ICP relevance in the first 10 words.** The brand's ICP must feel addressed before the
   scroll. Use the ICP awareness level from brand-brain to calibrate: problem-aware readers
   need the pain named; solution-aware readers need the angle that reframes the known.
4. **One owned voice.** No hedge words ("interesting," "exciting," "important") unless
   they are earned. The lede should pass the "could any other brand's newsletter send this?"
   test — if yes, it is generic.
5. **Bridge to curation.** Last 1–2 lede sentences set up why the curated links below matter —
   making the curation feel editorially chosen, not bolted on.

### Curation — the sections

Each curated section follows a tight template:

```
[Section header — named concept or angle, not "Resources" / "Links"]
[1–2 sentence editorial frame — what to notice, what the implication is]
[Link label + URL] — [1 sentence on why this specific piece, not a summary]
```

Quality gates for curation:
- Minimum editorial frame per section — if you have nothing to say about it, cut it.
- No more than one link per section in Standard mode; up to two in Digest mode.
- Sources should vary (practitioner post, data report, contrarian take) — never three think-pieces
  from the same category.
- Apply the "so what for the reader" test per item; fail = cut or hold.

Callout block (optional, powerful): one pull-stat or pull-quote per issue, formatted as a standalone
visual block. Use only real proof from `proof-vault` or brand-brain digest; else `[verify]`.

### Action — the CTA

One primary CTA per issue. Every newsletter issue is ultimately in service of one action. Call
`cta-variant-generator` (Skill tool) with the issue theme, brand context, and the destination URL.
Use the returned primary recommendation; include 1 alternate for the sender's choice.

CTA placement: bottom of the issue (default) OR anchored after the lede when the issue is
offer-driven (product launch, promo, event). Never both.

---

## Output format

```markdown
---
Issue: [Issue title / theme]
Date: [YYYY-MM-DD]
Mode: [Standard | Deep | Digest]
Brand: [slug, from brand-brain]
---

**Subject line:** [primary subject]
**Preview text:** [preview text]
(Alt subject: [alternate subject — from subject-line-preview-text-optimizer])

---

[LEDE SECTION HEADER — optional branded section name, e.g. "This week's take"]

[Lede body — full paragraphs, on-brand voice]

---

[SECTION 1 HEADER]

[Editorial frame]
[Link label](URL) — [one-sentence why]

---

[SECTION 2 HEADER]

[Editorial frame]
[Link label](URL) — [one-sentence why]

---

[Optional callout block]
> [pull-stat or pull-quote] — [source]

---

[CTA — primary recommendation from cta-variant-generator]
[CTA microcopy / friction-reducer]

---
_[Optional: Brief sign-off in brand voice — 1–2 sentences max]_
```

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No issue copy before `brand-brain` returns the active brand. Voice +
  banned-words override everything in this file.
- **One assertion per lede.** A lede that tries to cover three things covers nothing.
- **Curation earns its keep.** Every curated link comes with editorial framing. A link without
  framing is a dump, not an issue.
- **One CTA.** A newsletter that asks for three things gets zero clicks. Pick the action that
  matters most this week.
- **Honest proof only.** Real stats, real testimonials, real data. Anything unconfirmed is `[verify]`.
- **Reject before you pad.** Better a tight three-section issue than a bloated five-section one.
  Name the cuts and why — don't silently pad to hit a length target.
- **The reader's time is non-renewable.** Every sentence must justify its line count against the
  ICP's actual attention budget. No filler transitions. No "In today's issue…" preambles unless
  the brand's style guide requires them.

---

## What Not to Do

- Don't write a lede that is a summary of what's in the issue ("In this week's email, we
  cover…"). That is a table of contents, not a story.
- Don't pad thin content with generic commentary — flag the gap, offer to hold until stronger
  content exists.
- Don't re-implement brand voice derivation or proof research here — call `brand-brain` and
  `proof-vault`.
- Don't place two primary CTAs in the same issue. One action wins.
- Don't invent statistics, differentiators, or social proof — mark all unconfirmed claims `[verify]`.
- Don't produce the issue in markdown with bold headers labeled "Section 1," "Section 2" — use
  named editorial section heads that reflect the content.
- Don't start the lede with "This week," "Hey [first name]," or "Exciting news" — those are
  default-mode openers that signal low editorial ambition.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called; active brand loaded; voice + banned-words honored throughout?
- Lede makes one assertion; opens on the most interesting sentence; bridges to curation?
- Every curated section has an editorial frame (not just a link)?
- Callout block (if used) uses only confirmed proof or is marked `[verify]`?
- `subject-line-preview-text-optimizer` called; subject + preview text present?
- `cta-variant-generator` called; single primary CTA with one alternate?
- Rejects named (not silently dropped)?
- Mode correct (Standard / Deep / Digest) for the input and request?
- Ready to save to `./newsletters/[slug]-[YYYY-MM-DD].md` if the user asks?
