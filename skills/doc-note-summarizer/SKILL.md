---
name: doc-note-summarizer
description: >
  Distills any long-form input — article, research paper, deck, PDF, video transcript, voice memo,
  meeting recording, or raw brain dump — into the right-sized output for the job: a 3-sentence
  paste-into-Slack gist, a clean structured bullet summary, or a layered reading guide with key
  takeaways and open questions. Two modes: Express (default) delivers the gist fast for sharing or
  parking; Deep delivers scannable structured notes for reference, decision support, or repurposing.
  Honors the brand's voice and preferred format on any shareable output via brand-brain. Use when
  the user says "summarize this," "tldr," "notes from this article/doc/deck/video," "what are the
  key points," "distill this," "compress this for Slack," "turn these notes into something clean,"
  or pastes a long block and asks what to do with it. Summarizes only — it does not rewrite,
  reformat for a channel, or produce downstream assets; hand off to content-repurposer-atomizer or
  blog-post-drafting-engine for that.
---

# Doc & Note Summarizer

Paste in anything long; get out what you actually need. A 3-sentence Slack gist when you just need to share it. Clean structured bullets when you need to reference it. A layered reading guide when the source has enough depth to warrant one.

This skill reads the input and picks the right compression. It does not repurpose, rewrite for a channel, or produce downstream assets — those are downstream jobs for `content-repurposer-atomizer`, `blog-post-drafting-engine`, and their siblings.

---

## Skills this calls

- **`brand-brain`** (light-touch, for shareable output) — loads voice, banned words, and format preferences so any gist or formatted notes destined for a team or external audience land in the right tone. Skip the brand interview entirely for private working notes; load it silently when the output is Slack/email/public-facing.
- *(compose, don't duplicate)* `content-repurposer-atomizer` — when the user wants to turn the summary into social posts, threads, or newsletter snippets.
- *(compose, don't duplicate)* `blog-post-drafting-engine` — when the user wants to expand the summary into a full post.
- *(compose, don't duplicate)* `deck-presentation-writer` — when the user wants slides from the distilled content.
- *(compose, don't duplicate)* `meeting-prep-follow-up-pack` — when the source is a meeting recording and the user needs a follow-up pack, not just notes.

---

## How a run works

```
Step 0  Load brand (light-touch)  ──► call brand-brain only if output is shareable
Step 1  Classify the input         ──► source type + density + implied use case
Step 2  Pick the mode              ──► Express (default) | Deep (on request or high density)
Step 3  Compress using BLUF+MECE   ──► produce the output
Step 4  Self-review                ──► fidelity, voice, banned words, no invented facts
```

### Step 0 — Brand (always first for shareable output)

If the output is destined for Slack, email, a deck, or anything external, invoke `brand-brain` (Skill tool, `skill: brand-brain`) before producing the summary. Load the returned voice adjectives, banned words, and any format preferences; apply them to tone and phrasing. Keep it light — don't force positioning into a summary.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`; if none exists, ask the user for 2–3 voice adjectives and any banned words, or proceed in a neutral professional voice for private notes.

For private working notes (brain dumps, research captures, personal planning) — skip brand-brain entirely and say so in one word: `[internal]`.

### Step 1 — Classify

Read the input and note:
- **Source type:** article / paper / deck / PDF / video transcript / voice memo / meeting notes / brain dump / other
- **Density:** sparse (lots of filler) · medium · dense (every sentence load-bearing)
- **Implied use case:** share with team · file for reference · inform a decision · repurpose downstream · personal capture

Classification drives mode selection and compression ratio. Don't ask unless the use case is genuinely ambiguous.

### Step 2 — Pick the mode

| Trigger | Mode |
|---|---|
| Default; "tldr"; "Slack gist"; "quick summary" | **Express** |
| "structured notes"; "full notes"; "deep summary"; "reading guide"; dense source; >2 k words of input | **Deep** |
| Unsure | Express, offer Deep at the end |

---

## The compression framework: BLUF + MECE

The named framework underpinning both modes.

**BLUF (Bottom Line Up Front):** the single most important takeaway goes first, not last. Readers scan; give them the answer before the argument.

**MECE (Mutually Exclusive, Collectively Exhaustive):** categories in structured notes don't overlap and together cover the source. Use it to test your bullets: no point should live in two buckets; no important point should be homeless.

Applied together: lead with the verdict, then organize the support so a reader can navigate to exactly what they need and stop.

---

## Express mode (default)

The output is a **3-sentence gist** plus one optional line of metadata. Optimized for pasting into Slack, a quick email, or a Notion card.

**Structure:**
1. **What it is + the bottom line** (1 sentence): source type, core claim or conclusion.
2. **Why it matters / the key evidence** (1 sentence): the strongest supporting point or implication.
3. **What to do with it / the open question** (1 sentence): recommended action, decision trigger, or what's still unresolved.

Optional line 4: `Source: [title / URL / speaker / date]` — include when the user will share it so recipients can go to the original.

**Output format:**
```
[3-sentence gist]

Source: [if shareable]
```

Keep it tight. If the source genuinely requires more than 3 sentences to represent faithfully, note "Dense source — use /doc-note-summarizer deep for full notes" and still deliver the best 3-sentence version.

---

## Deep mode (on request or high-density source)

Structured notes for reference, decision support, or downstream repurposing. Follows the BLUF+MECE framework.

**Output structure:**
```
## [Title or inferred topic] — [source type, ~date if known]

**Bottom line:** [1-sentence verdict or key claim]

### Key points
- [MECE buckets as bold headers; 2–5 bullets each; concrete, no filler]

### Evidence & data
- [Specific numbers, studies, quotes — mark unconfirmed as [verify]]

### Open questions / gaps
- [What the source doesn't resolve; what you'd need to check before acting]

### Suggested next step
[1-line action recommendation — or "file for reference" if no action is implied]
```

Omit sections that are genuinely empty rather than padding them. For voice memos and meeting recordings, add a `### Decisions made` section if any are present.

Save Deep-mode output to `./notes/[slug-or-date].md` when the user says "save" or the session involves a research project. Inline otherwise.

---

## Principles

- **BLUF first, always.** The bottom line goes in sentence one, not the conclusion.
- **MECE discipline.** No redundant bullets; no orphaned points; check coverage before presenting.
- **Fidelity over brevity.** If a source is genuinely complex, say so — don't simplify into inaccuracy.
- **No invented facts.** Only what the source states. Inferred implications get flagged as `[inference]`; unconfirmed numbers get `[verify]`.
- **Brand voice on shareable output only.** Don't impose voice on private working notes.
- **One mode, one output.** Don't blend a gist and structured notes in the same response — pick the right mode and commit.

## What not to do

- Don't produce a summary before checking if the output is shareable (brand-brain call or skip decision).
- Don't pad Deep-mode sections that have no content from the source — omit them.
- Don't invent data, statistics, or conclusions not present in the input; mark inferences clearly.
- Don't turn this into a repurposing tool — hand off to `content-repurposer-atomizer` when the user wants posts, threads, or newsletter snippets.
- Don't ask clarifying questions when the use case is evident from the paste and the phrasing.
- Don't write 5-sentence gists and call them Express — 3 sentences is the constraint.

## Quality checklist

- Brand-brain called for shareable output; skipped with `[internal]` flag for private notes?
- Express: exactly 3 sentences, BLUF-ordered, source line included if shareable?
- Deep: MECE buckets, bottom-line first, empty sections omitted, `[verify]` on unconfirmed data?
- No invented facts; inferences labeled `[inference]`?
- Mode matched to the input density and implied use case?
- Offered handoff to `content-repurposer-atomizer` or `blog-post-drafting-engine` when the content has downstream potential?
