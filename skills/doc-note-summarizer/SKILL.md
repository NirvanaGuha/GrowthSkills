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
Step 3  Compress (craft + BLUF)    ──► find the thesis, cut to claims, flag flaws
Step 3b Bucket if Deep (MECE)      ──► organize Key points; produce the output
Step 4  Self-review                ──► fidelity, voice, banned words, no invented facts
```

### Step 0 — Brand (always first for shareable output)

If the output is destined for Slack, email, a deck, or anything external, invoke `brand-brain` (Skill tool, `skill: brand-brain`) before producing the summary. Load the returned voice adjectives, banned words, and any format preferences; apply them to tone and phrasing. Keep it light — don't force positioning into a summary.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md`; if none exists, ask the user for 2–3 voice adjectives and any banned words, or proceed in a neutral professional voice for private notes.

For private working notes (brain dumps, research captures, personal planning) — skip brand-brain entirely and say so in one word: `[internal]`.

### Step 1 — Classify

Read the input and note:
- **Source type:** article / paper / deck / PDF / video transcript / voice memo / meeting notes / brain dump / other
- **Density:** sparse (lots of filler) · medium · dense (every sentence load-bearing) — this sets the compression ratio in [Compression craft](#compression-craft)
- **Implied use case:** share with team · file for reference · inform a decision · repurpose downstream · personal capture

Classification drives mode selection and compression ratio. Don't ask unless the use case is genuinely ambiguous.

### Step 2 — Pick the mode

| Trigger | Mode |
|---|---|
| Default; "tldr"; "Slack gist"; "quick summary" | **Express** |
| "structured notes"; "full notes"; "deep summary"; "reading guide"; dense source; >2 k words of input | **Deep** |
| Unsure | Express, offer Deep at the end |

---

## The two frameworks — and where each applies

These are distinct tools, not one combined framework. Use the right one for the layer you're writing.

**BLUF (Bottom Line Up Front)** — from US Army staff-writing doctrine. The single most important takeaway goes first, not last. Readers scan; give them the answer before the argument. **Applies to both modes:** Express leads sentence one with the verdict; Deep leads with the `Bottom line` line.

**MECE (Mutually Exclusive, Collectively Exhaustive)** — from Barbara Minto's Pyramid Principle (McKinsey). Categories don't overlap and together cover the source. Use it to test your bullet *buckets*: no point lives in two buckets; no important point is homeless. **Applies to Deep mode only** — Express is too short to bucket; forcing MECE on a 3-sentence gist just pads it.

Neither framework tells you *what survives the cut*. That judgment is the craft below.

---

<a id="compression-craft"></a>

## Compression craft

The frameworks order and bucket. This is the harder skill: deciding what survives, finding the real point, and staying honest about a flawed source. A junior who only knows BLUF+MECE produces tidy notes that miss the thesis or launder bias. Don't be that junior.

### 1. Density → compression ratio

A "load-bearing claim" is one the source's conclusion depends on — remove it and the argument changes. Throat-clearing (preamble, transitions, restated context, hedging, recap) is never load-bearing. Compress to the *claims*, not to a fixed word count.

| Source density | Keep | Drop |
|---|---|---|
| **Dense** (every sentence load-bearing — papers, specs, tight memos) | ~1 bullet per load-bearing claim; preserve qualifiers and numbers verbatim | almost nothing — only true repetition |
| **Medium** (mix of claim and connective tissue — most articles, decks) | 1 bullet per distinct claim; merge a claim + its single best example into one bullet | examples 2–n, transitions, restated context |
| **Sparse** (filler-heavy — transcripts, brain dumps, blog SEO padding) | one bullet per *section* capturing its single point; the verdict | intros, sign-offs, anecdotes that don't carry a claim, repeated points |

Rule of thumb: if two bullets would collapse to one without losing a decision-relevant distinction, they were one bullet. When unsure whether a claim is load-bearing, ask: *would a reader make a different decision without it?* If no, cut it.

### 2. Finding the real thesis when the source buries the lede

Rambling sources — most transcripts, many blog posts, founder brain dumps — state their topic early and their actual argument late, or never explicitly. Don't summarize the stated topic; summarize the argument.

Read in this order before writing a word:
1. **Conclusion / last section first** — the real claim usually surfaces where the author finally commits.
2. **First and last paragraph of each section** — claims cluster at section edges; middles are support.
3. **Then test stated-topic vs. actual-argument:** write the topic the source *says* it's about in one phrase, then the position it actually *argues* for. If they diverge, the argument is your bottom line — not the title, not the intro.

For transcripts specifically: the thesis is often an aside the speaker drops once and moves past, or the answer to the last question asked. Scan for "the real point is," "what I'd actually do," "honestly," and hard reversals ("but here's the thing") — these mark where the speaker stops performing and commits.

### 3. Handling biased, one-sided, wrong, or self-contradicting sources

The job is faithful compression, not endorsement. **Never smooth a flaw into a clean claim — that launders the source's bias into your own voice.** Surface it instead, using the existing integrity tags and the `Open questions / gaps` section.

- **One-sided / unsupported assertion:** keep the claim, attribute it to the source, and flag the missing side. `[verify]` on the unconfirmed number; note the absent counter-evidence in `Open questions / gaps` ("Source asserts X with no data; competing view Y unaddressed").
- **Internally self-contradicting:** do not pick a winner silently. Capture both statements and flag the contradiction explicitly: "Source says X in §2 but Y in §5 — unresolved." That tension *is* the finding.
- **Likely-wrong or dubious claim:** report it as the source's claim (`[verify]`), never as fact. If it contradicts well-established knowledge, add a one-line `[inference]` note — don't argue with the source in the body.
- **Loaded / persuasive framing:** compress to the underlying claim in neutral words, dropping the spin. "A game-changing, must-have breakthrough" → "Source claims [specific capability]." Strip the adjectives; keep the assertion.

Litmus test: a reader of your summary should be able to tell *what the source claims* apart from *what is established fact* — and should never be more confident in a shaky claim because you tidied it up.

### 4. Multi-source and speaker attribution

- **Multiple sources:** organize by claim, not by source. When sources agree, state the claim once and note convergence. When they conflict, surface the disagreement in `Open questions / gaps` with each side attributed — don't average them into a mushy middle.
- **Multi-speaker transcripts (meetings, panels, interviews):** attribute decisions and commitments to the named speaker ("Priya: ship by Q3"). Attribute opinions only when who-said-it changes the weight; otherwise compress to the claim. Never merge two speakers' distinct positions into one bullet.

---

## Express mode (default)

The output is a **3-sentence gist** plus one optional line of metadata. Optimized for pasting into Slack, a quick email, or a Notion card.

**Structure:**
1. **What it is + the bottom line** (1 sentence): source type, core claim or conclusion. Use the *actual argument*, not the stated topic — see [finding the real thesis](#compression-craft).
2. **Why it matters / the key evidence** (1 sentence): the strongest supporting point or implication.
3. **What to do with it / the open question** (1 sentence): recommended action, decision trigger, or what's still unresolved — including any major bias or contradiction worth flagging.

Optional line 4: `Source: [title / URL / speaker / date]` — include when the user will share it so recipients can go to the original.

**Output format:**
```
[3-sentence gist]

Source: [if shareable]
```

Keep it tight. If the source genuinely requires more than 3 sentences to represent faithfully, note "Dense source — use /doc-note-summarizer deep for full notes" and still deliver the best 3-sentence version.

---

## Deep mode (on request or high-density source)

Structured notes for reference, decision support, or downstream repurposing. BLUF orders it; MECE buckets the `Key points`; [Compression craft](#compression-craft) decides what fills them.

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

- **BLUF first, always.** The bottom line goes in sentence one (Express) or the `Bottom line` line (Deep), never buried at the end.
- **Compress to claims, not word counts.** Density sets the ratio; cut everything that wouldn't change a reader's decision.
- **Fidelity over brevity.** If a source is genuinely complex, say so — don't simplify into inaccuracy.
- **Surface flaws, don't launder them.** Bias, contradictions, and dubious claims go in `Open questions / gaps` with `[verify]`/`[inference]` tags — never smoothed into clean fact.
- **No invented facts.** Only what the source states.
- **One mode, one output.** Never blend a gist and structured notes in one response — pick the mode and commit.

## What not to do

These are failure modes, not restatements of the principles above:

- Don't summarize the *stated topic* when the source actually argues something else — extract the real thesis.
- Don't pick a winner silently when a source contradicts itself, and don't average conflicting sources into a mushy middle — name the disagreement.
- Don't keep persuasive adjectives ("game-changing," "must-have") — compress to the underlying claim in neutral words.
- Don't turn this into a repurposing tool — hand off to `content-repurposer-atomizer` (posts/threads) or `blog-post-drafting-engine` (full post) when the user wants downstream assets.
- Don't ask clarifying questions when the use case is evident from the paste and phrasing.
- Don't write 5-sentence gists and call them Express — 3 sentences is the constraint.

## Quality checklist

- Brand: called for shareable output, or skipped with `[internal]` flag for private notes?
- Bottom line is the source's *actual argument*, verified against its stated topic?
- Compression ratio matches source density (dense → ~1 bullet/claim; sparse → 1 bullet/section)?
- Express: exactly 3 sentences; source line if shareable. Deep: MECE buckets, empty sections omitted.
- Every claim distinguishable as source-claim vs. established fact; `[verify]`/`[inference]` applied; bias/contradictions surfaced?
- Speakers/sources attributed where it changes the weight; no merged positions?
- Offered the right downstream handoff when the content has further potential?
