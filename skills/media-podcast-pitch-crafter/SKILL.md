---
name: media-podcast-pitch-crafter
description: >
  Writes personalized, editor-ready pitch emails for media placement and podcast bookings. Takes a
  story angle, the journalist's beat or show's audience, and supporting materials (press release,
  bio, past coverage, episode archive) → produces a single tight pitch under 150 words with a
  subject line built to earn an open, not a delete. Adapts the PESO model
  (Paid/Earned/Shared/Owned, coined by Gini Dietrich) as an angle-fit lens, with a three-beat structure: relevance hook,
  credibility proof, clear ask. Two modes: Media Pitch (reporters, editors, newsletters) and
  Podcast Booking Pitch (hosts, producers, booking managers). Does not write press releases,
  amplification packs, or executive quote polish — calls the sibling skills that own those jobs.
  Use when the user says "pitch a journalist," "podcast pitch," "media outreach," "get on a show,"
  "write a pitch email," "PR outreach," "secure coverage," "book a podcast," "media pitch template,"
  or drops a story idea and asks how to get it placed.
---

# Media & Podcast Pitch Crafter

Gets the open. Gets the reply. Gets the booking. A 150-word pitch that earns coverage does one thing well: it makes the journalist or host immediately see why *their* audience cares — not why the company is proud of its announcement. This skill builds that pitch from a story angle and target dossier, not from a press release summary.

Two modes, one discipline: every pitch is researched, not templated. Fabricated flattery is instantly detectable and fatal. Only facts earned by actual research go in; unconfirmed claims get `[verify]`.

---

## Skills this calls

- **`brand-brain`** (required) — loads voice, positioning, real proof, ICP, and banned words before any copy is written.
- **`proof-vault`** *(optional)* — surfaces the strongest validated proof points for credibility hooks; synthesize inline if absent.
- **`competitive-intelligence-dossier`** *(optional)* — useful when a story angle turns on a market trend or category claim; pull inline if absent.
- **`press-release-social-blog-amplification-pack`** — if the user has a press release, call this for the owned/shared amplification; this skill handles the earned pitch only.
- **`quote-polisher`** — if the pitch includes an executive pull-quote, delegate polishing there rather than improvising.
- **`account-dossier-builder`** — for podcast or publication research; call it when the user provides only a show/outlet name and needs a full profile before targeting.
- **`de-slop-humanize-pass`** — final pass on any pitch draft flagged as generic or over-polished.

---

## How a run works

```
Step 0  Load the brand           ──► call brand-brain (always first)
Step 1  Profile the target       ──► journalist beat / show audience / recent coverage
Step 2  Pick the mode            ──► Media Pitch | Podcast Booking Pitch
Step 3  Lock the angle           ──► one specific story hook, not the company announcement
Step 4  Write the pitch          ──► three-beat structure, ≤150 words
Step 5  Write the subject line   ──► beat-matched, curiosity or specificity angle
Step 6  Self-review              ──► quality checklist; flag any unresearched claims
```

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`) before producing any copy. Use the returned digest for voice, banned words, real proof, ICP, and positioning. Never fabricate proof points; mark anything unconfirmed `[verify]`.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`. If neither exists, ask the user for: brand name + URL, three positioning differentiators, two proof points (real customers, numbers, or awards), and voice adjectives + banned words. Then proceed.

### Step 1 — Profile the target

Before writing one word of pitch copy, establish the target's real context. Shallow research produces generic pitches; generic pitches go unread.

**For journalists / editors / newsletters:**
- Recent 3–5 bylines or issues: what angles do they favor? What sources do they cite?
- Beat scope: sector, company size, narrative frames (underdog story vs. data story vs. policy angle)
- Publication's audience: decision-makers, practitioners, consumers?
- Any prior coverage of the company or category?

**For podcast hosts / producers:**
- Last 5–10 episode titles and guests: recurring themes, guest archetypes
- Host background and stated show mission
- Episode length and format (interview / solo / panel)
- Sponsor categories (signals audience buying power and intent)
- Audience size / reach if available; mark `[verify]` if estimated

If the user supplies minimal target data, invoke `account-dossier-builder` with the outlet or show name to pull a structured profile before continuing.

---

## PESO Angle Fit Lens

Every placement opportunity has an optimal *why this, why now, why them* angle. PESO (Gini Dietrich's media model) is a media-integration framework, not a pitching one — here we borrow it as a lens to pressure-test angle fit before drafting:

| PESO tier | What it means for pitching | Angle test |
|---|---|---|
| **Earned** (the job here) | Journalist / host selects it; zero spend | "Is there a story here independent of our launch?" |
| Paid | Sponsored segment / advertorial | Out of scope — do not pitch as earned if it requires spend |
| Shared | Podcast → repurposed by host's audience | Angle should be inherently shareable / tweetable |
| Owned | Press release, company blog | Source material for the pitch — not the pitch itself |

A story angle passes if it survives this filter: **"Would this be interesting if a competitor announced it?"** If yes, it has legs. If no, it is marketing wrapped in a press release — and editors can smell it.

---

## Three-Beat Pitch Structure

Every pitch, regardless of mode, uses three beats in under 150 words:

```
Beat 1 — RELEVANCE HOOK    (1–2 sentences)
  Open with the story, not the company.
  Connect to the target's recent work, their audience's lived problem,
  or a trend the data supports. Name the specific piece/episode.

Beat 2 — CREDIBILITY PROOF  (2–3 sentences)
  Establish why this source / guest is the right one to tell it.
  One or two real proof points (customer number, outcome, credential, award).
  No puffery; no superlatives without a source.
  Unconfirmed claims → [verify].

Beat 3 — CLEAR ASK + LOGISTICS  (1–2 sentences)
  State the exact request: "15-minute call to explore the angle" /
  "happy to provide data under embargo" / "available [dates] for a recording."
  One ask only. No multiple CTAs.
```

### Mode A — Media Pitch (reporters, editors, newsletter writers)

Additional constraints:
- **Subject line:** 6–8 words. Test: would a reporter open this on a crowded inbox day? Avoid "PRESS RELEASE:" or "EXCLUSIVE:" unless you genuinely have an exclusive.
- **Lead with the story angle, not the company name.** The reporter knows you represent someone; they don't care yet.
- **Data point in Beat 1 when possible** — a specific, sourced stat commands more attention than a claim. If the number isn't confirmed, use `[verify]` and note in the self-review.
- **Embargo or exclusive flag** (if applicable): state clearly in the subject and the ask, never buried.
- **Follow-up note** at the end: one line on when you'll follow up (once, max).

### Mode B — Podcast Booking Pitch (hosts, producers, booking managers)

Additional constraints:
- **Subject line:** address the host by name; reference a specific episode in ≤8 words. Example: "Guest for your [Ep. Title] follow-up."
- **Beat 1 = proof you actually listened** — cite a specific moment, guest, or theme from a named episode. Never generic ("I love your show").
- **Angle hook replaces "here's my bio"** — the show already has enough guests with impressive bios. Pitch the *episode idea*, not the person.
- **Three talking-point bullets** (Beat 2 extension) — concrete, episode-ready conversation hooks, each one sentence. These double as the producer's episode outline.
- **Social proof** in Beat 2: previous notable shows, download reach if real, published piece the host's audience would recognize. Mark estimates `[verify]`.
- **No rider demands** in the pitch — episode length preference, recording format, and promotional asks go in the post-booking brief, not here.

---

## Output format

```
## Pitch — [Target Name / Show] — [Story Angle in 5 words]

**Subject:** [subject line]

---
[Pitch body — ≤150 words, three beats]

---
**Angle fit note:** [1 sentence: why this angle is right for this target's audience]
**Proof status:** [list any [verify] items the user must confirm before sending]
**Follow-up timing:** [one line]
```

Save to `./pr/pitches/[slug]-[target-slug]-pitch.md` when the user asks to persist or when a batch of pitches is produced. Never auto-save a single inline draft.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No pitch copy before the active brand is loaded.
- **Researched flattery only.** If you cannot cite a specific episode, byline, or piece of coverage, do not fake it. Generic openers ("I love your work") are worse than none.
- **One story, one ask.** Pitches that try to sell two angles or make two requests get neither.
- **150-word ceiling is a feature.** Discipline in the pitch signals discipline in the story. Never exceed it except in Podcast Mode Beat 2 bullets (three lines, still tight).
- **Proof or `[verify]`.** Every number, ranking, and outcome claim must be real and sourced, or flagged for the user to confirm before sending.
- **Voice obedience.** The brand's voice adjectives and banned words apply even in outreach copy. Journalists respond to authentic voices, not corporate boilerplate.

## What Not to Do

- Do not write a pitch before `brand-brain` returns the active brand.
- Do not open with "My name is / I'm reaching out because / Hope this finds you well."
- Do not pitch the press release — pitch the story the press release is evidence of.
- Do not list every credential; pick the one that earns credibility *for this specific angle*.
- Do not invent listener counts, download numbers, coverage reach, or customer outcomes.
- Do not send the same pitch to every target; angle fit is non-negotiable per recipient.
- Do not put multiple asks in the CTA; one clear next step only.
- Do not use the skill folder (`./skills/`) as a save path for pitch drafts.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or bootstrapped) before any pitch copy written?
- Target profiled with at least one specific, researched reference (byline, episode, beat)?
- Pitch body ≤150 words (Podcast Mode: ≤150 + 3 tight bullets)?
- Three-beat structure intact: relevance hook → credibility proof → clear ask?
- Subject line ≤8 words, beat-matched, no generic openers?
- Zero fabricated proof, customer names, or stat claims; all uncertain items flagged `[verify]`?
- Single ask in the CTA?
- Voice + banned-words honored?
- Angle passes the PESO Earned filter ("interesting if a competitor announced it")?
- Any `[verify]` items surfaced explicitly to the user before sign-off?
