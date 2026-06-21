---
name: video-loom-summarizer
description: >
  Converts any video recording — Loom URL, webinar replay, competitor demo, sales call, or internal
  meeting link — into a structured, timestamped text summary with key decisions, action items, notable
  quotes, and a 3-sentence TL;DR. Uses the OARR framework (Outcome, Actions, Rationale, Risks) to
  classify content so readers get navigation-grade structure, not a wall of bullet points. Brand context
  is loaded via brand-brain so summaries honor voice, surface brand-relevant signals (competitor
  mentions, pricing reveals, objection patterns), and feed cleanly into downstream skills. Output lands
  inline for quick reads and, when a batch or archive is needed, saves to a project-relative path.
  Use whenever the user says "summarize this Loom," "digest this webinar," "pull action items from
  this recording," "what did they say in the demo," "turn this meeting into notes," "TL;DR this video,"
  or pastes a video URL expecting structured output instead of a transcript.
---

# Video & Loom Summarizer

Paste a URL, get a usable summary. Not a raw transcript — a navigation-grade digest your team can act on in 60 seconds.

Every content type has a different skeleton: a Loom screenshare front-loads its purpose and ends with a next step; a webinar has a through-line argument; a competitor demo reveals positioning moves; a sales call hides objections. This skill reads the room and structures the output accordingly, not as a generic list of "things that were said."

Brand-brain runs first. That's what turns a summary into an asset — competitor mentions get flagged, objections land in your objection library, pricing reveals get noted, and every line of output honors your voice.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's context: voice, ICP, banned words, offer, proof, competitor set. All summaries are filtered and framed through the returned digest.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their brand name, ICP, primary competitors, and any output voice notes before proceeding.
- **`doc-note-summarizer`** — when the input is a transcript paste or text file rather than a live URL, delegate the raw condensation pass to this skill.
- **`meeting-agenda-action-item-builder`** — for internal meeting recordings where the primary output needed is an action register with owners and due dates.
- **`jtbd-customer-interview-suite`** — when the recording is a customer interview, hand the transcript off here for JTBD tagging and quote banking.
- **`objection-library-builder`** — when objections are found in a sales call or competitor demo, route them here for structured capture.
- **`content-repurposer-atomizer`** — when the user wants the summary to also spawn social posts, a blog section, or a LinkedIn thread.

---

## How a run works

```
Step 0  Load brand          ──► call brand-brain; receive digest + brand.md path
Step 1  Ingest & classify   ──► detect content type; decide output skeleton
Step 2  Extract structure   ──► transcript or caption fetch, or user-pasted text
Step 3  Summarize via OARR  ──► apply framework per content type
Step 4  Flag brand signals  ──► surface competitor mentions, objection patterns, proof gaps
Step 5  Self-review         ──► check completeness, voice, and action-item attribution
Step 6  Deliver             ──► inline (default) or save to ./summaries/
```

### Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns the active brand digest — voice, banned words, ICP, competitors, offer, proof. Do not produce any output before it returns.

Obey the returned voice as a hard override. Use the brand's known competitor set to flag competitive signals in the recording. Use the ICP to decide which moments matter and which are skip-worthy. Mark any numeric claim from the recording as `[verify]` unless the source says it explicitly.

**Fallback if brand-brain is absent or returns no brand:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their brand name, ICP, primary competitors, and any output voice notes before proceeding.

### Step 1 — Classify the content type

| Type | Primary skeleton | Key extraction targets |
|---|---|---|
| **Loom / async update** | Purpose → walkthrough → ask/next step | Decision rationale, named owners, blockers |
| **Webinar / training** | Hook → argument arc → key claims → CTA | Frameworks named, stats cited, slides referenced |
| **Competitor demo** | Product tour → positioning moves → objections handled | Feature reveals, pricing cues, self-comparisons, claims to verify |
| **Sales / discovery call** | Discovery → pain → pitch → objections → close/next | BANT signals, objections, decision-maker cues, next-step commitment |
| **Internal meeting** | Agenda items → decisions → blockers → actions | Decisions with rationale, action owners, open risks |
| **Customer interview** | Opening → story → pains → workarounds → ideal future | JTBD statements, verbatim quotes, moments of frustration |

If the content type can't be determined from the URL or context, ask one question: "What kind of recording is this?" Do not proceed with the wrong skeleton.

---

## The OARR Framework (applied per content type)

OARR stands for **Outcome, Actions, Rationale, Risks**. It replaces the generic "key points" bullet dump with four classified buckets that tell readers exactly what to do with each piece of information.

| Bucket | What goes here | Reader job |
|---|---|---|
| **Outcome** | What was decided, agreed, or concluded | Know what happened |
| **Actions** | Tasks with an owner and a due date or trigger | Execute |
| **Rationale** | Why — the stated reasoning behind a decision | Evaluate or revisit |
| **Risks / open items** | Named blockers, unresolved questions, follow-ups needed | Unblock |

Not every moment maps to OARR. Filler, repetition, and off-topic tangents are cut. Background context with no decision attached is compressed to one line or dropped.

**Timestamps are non-negotiable.** Every OARR item carries a `[MM:SS]` or `[HH:MM:SS]` anchor so a reviewer can jump to the source without scrubbing.

---

## Output structure

### Quick summary (default, inline)

```
## TL;DR
[3 sentences: what happened, the headline decision or insight, the single most important next step]

## [Content-type-appropriate section title] — [Video title or URL domain, duration]
Brand signals flagged: [list competitor mentions, pricing reveals, or objections if found; else omit]

### Outcomes [MM:SS – MM:SS]
- [Outcome 1]
- [Outcome 2]

### Actions
| Action | Owner | Due / Trigger | Timestamp |
|---|---|---|---|
| [task] | [name or role] | [date / event] | [MM:SS] |

### Rationale [MM:SS – MM:SS]
- [Decision A] — because [stated reason] [MM:SS]
- [Decision B] — because [stated reason] [MM:SS]

### Open items / risks
- [Blocker or unresolved question] — owner: [name/TBD] [MM:SS]

### Notable quotes
> "[Verbatim quote]" — [speaker, if identified] [MM:SS]

### Brand signals (for [brand slug])
- **Competitor mention:** [name] — context: [one line] [MM:SS]
- **Objection surface:** [one line] [MM:SS]
- **Pricing / feature reveal:** [claim] `[verify]` [MM:SS]
```

Omit any section with no content (e.g., if no brand signals, drop that section entirely). Keep TL;DR to exactly 3 sentences.

### Batch / archive mode

When the user asks to process multiple recordings, or says "save this" or "archive it," save to `./summaries/[slug]-[YYYY-MM-DD].md`. One file per recording. Index entry appended to `./summaries/index.md` (title, date, type, TL;DR first sentence, path).

---

## Ingestion paths

The skill works regardless of access level:

1. **Live URL with caption/transcript access** — fetch via browser tool (Claude-in-Chrome or WebFetch); extract the transcript or auto-captions. Confirm the URL resolves before proceeding.
2. **User-pasted transcript or notes** — delegate the condensation pass to `doc-note-summarizer`, then apply OARR and brand-signal extraction here.
3. **Recording with no text layer** — note the limitation; ask the user to paste the auto-generated transcript (from Loom's transcript panel, YouTube's "Show transcript," or Zoom's auto-captions). Do not fabricate content.
4. **Partial notes / meeting recap paste** — treat as ingestion path 2.

Never invent timestamps or content. If a timestamp is unavailable, omit the `[MM:SS]` anchor and note once that timestamps could not be extracted.

---

## Brand signal extraction (what to watch for)

While summarizing, run a parallel scan for signals the brand cares about:

- **Competitor mentions** — direct names, product names, "another tool," or thinly veiled references. Flag with competitor slug from brand-brain.
- **Objections** — any hesitation, pushback, price sensitivity, comparison to an alternative, or "we tried X and…" statement. Route to `objection-library-builder` if the user asks.
- **Pricing reveals** — any quoted price, discount, or "starts at" from a competitor demo or sales call. Mark `[verify]`.
- **Proof points** — any stat, case study result, or outcome claim cited in the recording. Mark `[verify]` unless the speaker cites a direct source.
- **Feature gaps or requests** — unmet needs named by the speaker. Flag for product/content use.

Brand signal extraction runs automatically. If zero signals are found, omit the section silently.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No output before brand context is loaded. Voice and banned-words are hard overrides.
- **OARR, not bullets.** Classify content into Outcome / Action / Rationale / Risk. A list of observations with no classification is not a summary — it's a transcript with whitespace.
- **Timestamps anchor everything.** Every OARR item, notable quote, and brand signal carries a timestamp so reviewers can jump to source.
- **No fabrication.** If the recording has no text layer, say so and ask for a paste. Never invent quoted speech or decisions.
- **One skeleton per content type.** A sales call and a webinar produce structurally different outputs. Don't homogenize them.
- **Verify externally sourced numbers.** Any stat cited by a speaker is `[verify]` unless they cite a primary source inline.
- **Cut filler ruthlessly.** Greetings, "um"s, topic drift, and re-statement of what was just said are cut without mention.

---

## What Not to Do

- Don't produce any output before `brand-brain` returns — not even a TL;DR placeholder.
- Don't fabricate speaker attribution, decisions, or quotes if the transcript is ambiguous.
- Don't use the generic "key points" or "highlights" skeleton — always apply OARR.
- Don't surface timestamps you didn't extract — if timestamps are unavailable, note it once and omit.
- Don't write summaries that require watching the recording to understand — the summary is the artifact.
- Don't ignore brand signals — a competitor mention in a sales call recording is a content and competitive intelligence event.
- Don't exceed 3 sentences in the TL;DR. If you can't fit it in 3, the TL;DR is doing too much.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called; active brand loaded and digest applied?
- Content type identified; correct OARR skeleton used (not the generic bullets format)?
- TL;DR is exactly 3 sentences: what happened, the headline, the next step?
- Every OARR item carries a timestamp (or timestamp-unavailability noted once)?
- Action table has owner and trigger/due date columns populated (TBD if genuinely unassigned)?
- Brand signals scanned; competitor mentions, objections, and pricing reveals flagged with `[verify]`?
- Fabricated content: zero?
- Voice + banned-words from brand-brain honored throughout?
- Archive mode: saved to `./summaries/[slug]-[YYYY-MM-DD].md` and index updated?
