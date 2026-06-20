---
name: subject-line-preview-text-optimizer
description: >
  Turns an email body, campaign brief, or a weak subject line into ranked subject line /
  preview text pairs optimized for open rate — applying the six open-rate levers (curiosity,
  urgency, personalization, specificity, self-interest, social proof) with the right lever
  mix for the campaign type and audience stage. Two modes: by default it reads the copy or
  brief and returns a recommended pair plus 2–3 alternates across distinct angles; on request
  it produces a full battery of variants with A/B recommendation and a test-ready hypothesis.
  Calls `brand-brain` first so every pair is on-voice, uses only real proof, and honors banned
  words. Pair-scores every output against open-rate criteria before presenting. Use when the
  user says "write a subject line," "improve my subject line," "subject line options," "preview
  text," "A/B test my subject lines," "why is my open rate low," "optimize this email header,"
  or hands over email copy and asks what the subject should be.
---

# Subject Line & Preview Text Optimizer

Subject lines are the first conversion event in any email or push campaign — before copy,
before design, before offer. This skill writes and ranks subject line / preview text pairs
using a named lever framework so a junior operator produces senior-quality options. It does
not rewrite the email body. It does not redesign the campaign flow. If the offer is the real
problem, it says so.

---

## Skills this calls

- **`brand-brain`** (required) — loads voice, banned words, ICP + awareness tendency, real proof,
  and offer mechanics. Subject lines are not written before this returns.
- *(optional, when installed)* `proof-vault` — for real social-proof snippets usable in the
  subject line. Synthesize inline when absent.
- *(optional, when installed)* `cta-variant-generator` — when the user also needs the email CTA
  aligned to the subject line promise (message-match check). Invoke only if explicitly requested.

---

## How a run works

```
Step 0  Load brand context  ──► call brand-brain (bootstraps on first use)
Step 1  Pick the mode       ──► Quick (default) | Battery (on request)
Step 2  Score open-rate levers for this campaign type + audience stage
Step 3  Generate pairs
Step 4  Self-score each pair → present ranked output
Step 5  Offer to save battery output
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`), passing the user's request
and any named brand. It returns the active brand's digest: voice adjectives, banned words, offer
mechanics + destination URLs, real proof, ICP + awareness tendency. Do not write a single subject
line until it returns.

Obey the returned voice and banned-words as hard overrides; use only returned real proof (mark
anything else `[verify]`); anchor personalization tokens to the brand's known CRM fields.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` + that
brand's `brand.md` directly; if none, ask the user to install `brand-brain` or answer a
4-question mini-setup (brand name · ICP + awareness tendency · offer · 3 voice adjectives +
banned words), then proceed. Always prefer the call.

### Step 1 — Pick the mode

- **Quick mode (default).** Email body or brief → one recommended subject line / preview text
  pair + 2–3 alternates across distinct levers. Always the default unless the user asks for
  options/variants/battery/A/B.
- **Battery mode.** Triggered by "variants," "options," "give me 10," "A/B test," or a count
  → full lever-spread table + A/B recommendation + hypothesis.

When unsure, default to Quick and offer Battery at the end.

---

## Quick mode (default)

1. **Read the email or brief.** Extract: core offer/hook, stage in the funnel
   (acquisition/onboarding/nurture/re-engagement/transactional), any personalization tokens
   available, and the single action the email asks for.
2. **Select the dominant lever** for this campaign type (see *Lever Framework* below) — one
   lever should lead; the others support.
3. **Check the awareness ceiling.** Match subject line specificity to where the reader is in
   their journey. Don't pitch a feature the reader hasn't heard of yet; don't be too vague for
   a high-awareness list.
4. **Write one recommended pair** (subject line + preview text). The preview text extends —
   never repeats — the subject line, adding specificity, urgency, or a curiosity close.
5. **Add 2–3 alternates** each on a *different* leading lever (not synonyms). Label the lever.
6. **One line of why** the recommendation fits this audience + campaign type.

Quick-mode output is short: recommended pair with lever label, alternates with lever labels,
one-line rationale. No table unless Battery is requested.

### Quick-mode format

```
## Subject line options — [campaign name or type]
Brand: [slug, via brand-brain]
Stage: [funnel stage]

RECOMMENDED
Subject:  [subject line text]
Preview:  [preview text]
Lever:    [dominant lever]
Why:      [one sentence]

ALTERNATES
A  Subject: … | Preview: … | Lever: …
B  Subject: … | Preview: … | Lever: …
C  Subject: … | Preview: … | Lever: …
```

---

## Battery mode (on request)

1. **Offer gate.** If the email body is vague, the offer is buried, or no clear hook exists —
   name it. Battery variants can't manufacture a hook that isn't there.
2. **Set the awareness ceiling** from the brand's ICP + funnel stage.
3. **Generate across all six levers** — at least one pair per lever, no two pairs with the
   same dominant lever, minimum 6 pairs total.
4. **Score each pair** (see *Pair-Scoring Rubric* below) and sort the table descending.
5. **Recommend an A/B pair** — the top scorer + a deliberately different-lever variant — with
   a stated test hypothesis and the metric it resolves (open rate, and if measurable, CTOR
   as a downstream signal).
6. **Preview text discipline:** every preview extends the subject; no "View in browser" filler.

### Battery output format

```
## Subject line battery — [campaign name or type]
Brand: [slug] · Stage: [funnel stage] · ICP awareness: [level]
[⚠ offer gate note if any]

| # | Subject line | Preview text | Lever | Score /10 |
|---|---|---|---|---|

### Recommended A/B pair
Primary (test A): #_  Variant (test B): #_
Hypothesis: [what you'll learn from the open-rate difference]
Downstream check: CTOR on the click if ESP supports it.

### Message-match note
[One sentence confirming the subject line pays off what the email body delivers.]
```

Save battery output to `./sequences/[slug]-subject-lines.md` if asked.

---

## Lever Framework (the named craft method)

Subject lines work through six open-rate levers. This skill builds every pair on a conscious
lever choice — not gut feel.

| Lever | What it does | When to lead with it | Watch out for |
|---|---|---|---|
| **Curiosity** | Opens a question or pattern interrupt the reader must resolve | Re-engagement, cold list warm-up, early nurture | Bait-and-switch — the email must answer the curiosity |
| **Urgency / Scarcity** | Creates time or quantity pressure | Promotional, flash sale, expiring trial, event countdown | Fake urgency erodes trust; use only real deadlines |
| **Personalization** | Name, segment, recent behavior, or account signal | Triggered/behavioral, onboarding, win-back | Empty `{{ first_name }}` fallback looks broken; confirm default |
| **Specificity / Number** | Concrete stat, step count, or timeframe beats vague claims | How-to, listicle-led, product education | Invented numbers need `[verify]`; use only real proof |
| **Self-interest** | Leads with the direct benefit to the reader | Offer announcements, feature launch, upgrade nudge | Generic "save time/money" without the brand's specific claim is filler |
| **Social proof** | Peer signal (customer count, rating, story snippet) | Evaluation/decision stage, mid-funnel nurture | Unverified numbers need `[verify]`; check brand-brain proof |

**Lever interaction rules:**
- A subject line can carry two levers; more than two dilutes both.
- Urgency + specificity is the highest-converting combo for promotional sends (real deadline + real number) [verify for your list].
- Curiosity + personalization works well for re-engagement.
- Preview text can introduce a second lever when the subject line leads with one.

---

## Character limits and rendering

| Client / surface | Subject line | Preview text | Notes |
|---|---|---|---|
| Mobile inbox (most common) | 30–40 chars visible | 40–90 chars | Write mobile-first; test desktop second |
| Desktop Gmail / Outlook | ~60–70 chars | 90–140 chars | Full subject more visible; preview supplements |
| Apple Mail (desktop) | ~70 chars | ~140 chars | Highest preview usage — fill it |
| Push notification (web) | ≤50 chars title | ≤90 chars body | Call `push-notification-copy-generator` for full push work |

**Default target:** 40–50 character subject line (reads well on mobile, doesn't truncate on
desktop). Flag when a draft exceeds 60 characters and offer a trimmed version.

**Preview text hygiene:**
- Always populate it explicitly — never let the ESP pull the first body sentence.
- Never repeat the subject line word for word; extend or complement it.
- End with a curiosity hook or benefit close when space allows.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No subject line before `brand-brain` returns. Voice + banned-words
  override everything else here.
- **One opening promise per pair.** Subject line + preview text together make a single
  coherent promise the email must keep.
- **Vary the lever, not the vocabulary.** Six versions of "big sale" are not six options.
- **Honest urgency and honest proof only.** Fake deadlines and invented stats get flagged as
  `[verify]` or removed.
- **Message match is load-bearing.** The subject line promise must be paid off in the first
  screen of the email — no bait-and-switch.
- **Preview text is real estate.** It is never filler, never a repeat, never left to the ESP
  to guess.
- **Awareness ceiling respected.** Don't pitch a cold list like a warm one.

---

## What not to do

- Don't write subject lines before `brand-brain` returns the active brand.
- Don't reimplement brand scanning, interviewing, or storage — that lives in `brand-brain`.
- Don't generate near-synonym variants and call them options — if the lever spread is thin,
  say the offer is too vague to build distinct hooks from.
- Don't use all-caps, excessive punctuation, or emoji unless the brand voice explicitly allows
  them (check `brand-brain` voice adjectives + banned words).
- Don't put urgency language on a non-expiring offer — flag it.
- Don't rewrite the email body or redesign the campaign sequence; flag structural problems
  and stay in your lane.
- Don't invent proof (customer counts, stats, rankings) — only confirmed proof from
  `brand-brain` / `proof-vault`; everything else is `[verify]`.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or bootstrapped) before any pair was written?
- Voice adjectives honored; banned words absent from all options?
- Only real proof used; invented or unconfirmed figures marked `[verify]`?
- Quick: one recommended pair with lever label + 2–3 different-lever alternates + one-line why?
- Battery: ≥6 pairs, one per lever minimum, sorted by score; A/B recommendation includes a
  stated hypothesis and a metric?
- No pair exceeds the character limit for the target client; preview text extends (not
  repeats) the subject line?
- Awareness ceiling respected; urgency is real; message-match note included?
- Offer weakness flagged (not papered over) if the hook is genuinely absent?
