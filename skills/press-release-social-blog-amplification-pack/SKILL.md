---
name: press-release-social-blog-amplification-pack
description: >
  Turns a published press release into a complete owned-media amplification kit: a LinkedIn exec
  post (first-person narrative, ~150–200 words), an X/Twitter thread (hook + 4–6 reply tweets +
  CTA), and a 500–700 word owned-media blog post with an SEO headline, intro, body, and internal
  links. All three assets share one story spine but are reframed for their channel and audience —
  not copy-pasted across platforms. Brand voice, banned words, and ICP come from the shared
  brand-brain; the exec's attribution style and any linked proof come from the press release itself.
  Saves to ./pr/<slug>/. Use when the user says "amplify this press release," "turn our PR into
  content," "social posts from our announcement," "blog post from our press release," "exec post
  for the announcement," "LinkedIn from our PR," "X thread for the news," or pastes a press release
  and asks what to do with it.
---

# Press Release + Social/Blog Amplification Pack

A press release is a legal notice dressed up as news. It is not a story. This skill extracts the
story buried in it and reshapes it into three assets that people actually read — an exec-voice
LinkedIn post that sounds human, an X thread built for the scroll-stopping hook, and an
owned-media blog post that earns organic traffic long after the news cycle dies.

One press release in. Three ready-to-publish assets out. No brand context lost in translation.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads voice, ICP, banned words, offer mechanics, proof, and
  positioning. Every asset obeys the returned voice overrides and uses only the real proof points.
- **`content-repurposer-atomizer`** (Step 2, optional) — if the press release is dense or multi-topic,
  call this to atomize the source into a prioritized message map before drafting. Skip for short,
  single-angle releases.
- **`de-slop-humanize-pass`** (Step 4, optional) — run the LinkedIn post through this after drafting
  if the exec voice reads too corporate. Skip when voice is already clean.
- **`on-page-seo-optimizer`** (Step 4, optional) — run the blog post through this if the user wants a
  full keyword and on-page pass before publishing. Inline optimization is sufficient for the default
  run.
- **`linkedin-post-writer`** — not called directly here; its exec-post conventions are baked into
  Step 3a below. Call it instead if the user wants LinkedIn as a standalone deliverable.

---

## How a run works

```
Step 0  Load the brand       ──► brand-brain skill (required before any drafting)
Step 1  Extract the story    ──► parse PR for angle, news hook, facts, quotes, proof
Step 2  Build the story spine ──► one sentence: what changed and why it matters to the ICP
Step 3  Draft the three assets
        3a  LinkedIn exec post  (first-person, 150–200 words, no markdown)
        3b  X thread           (hook + 4–6 tweets + CTA)
        3c  Blog post          (500–700 words, SEO headline, internal links)
Step 4  Self-review + optional sibling calls
Step 5  Deliver and save to ./pr/<slug>/
```

---

## Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (`skill: brand-brain`). It returns the active brand's digest —
voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning,
ICP + awareness tendency. Obey the voice and banned-words as hard overrides. Use only the real
proof returned (mark anything unconfirmed `[verify]`).

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's
`brand.md` directly. If neither exists, ask the user for: brand name, one-sentence positioning,
3 voice adjectives, any banned words, and the ICP. Prefer the call.

---

## Step 1 — Extract the story (PR parsing)

Read the full press release and pull these fields — the rest is boilerplate:

| Field | What to pull |
|---|---|
| **News hook** | The single most newsworthy fact (funding amount, product launched, partnership name, milestone number) |
| **Who it helps** | Which customer segment or persona benefits |
| **Why now** | The market condition or trend making this timely |
| **Best exec quote** | The one sentence from the release that sounds most human (mark `[verify]` if paraphrased) |
| **Hard proof** | Numbers, named customers, dates, amounts — only what is in the release |
| **Destination URL** | Where readers should go (the release URL, a product page, or landing page) |

If any of these are absent from the release, note `[missing — ask user]` inline; do not invent.

---

## Step 2 — Build the story spine

Write one sentence (not published — internal only):

> "[Brand] [did X] so that [ICP] can [outcome] — at a time when [market condition]."

Every asset in Step 3 is a retelling of this sentence for its channel. If you cannot write this
sentence clearly from the release, the release is either too thin or multi-story. If thin, flag it
and ask for a quote or stat to strengthen it. If multi-story, call `content-repurposer-atomizer`
to prioritize the angles, then pick one per run.

---

## Step 3a — LinkedIn exec post (150–200 words, no markdown)

**Framework: Problem → Turn → Proof → Invitation**

The executive voice is first-person, direct, and slightly vulnerable — not a re-read of the press
release. The post earns the share; it does not announce at the reader.

| Section | Job | Approx. length |
|---|---|---|
| **Hook line** | One punchy sentence that earns the "see more." Start with the tension or outcome, never "I'm thrilled to announce." | 1 sentence |
| **Problem/context** | Why this moment matters to the reader's world. Make the ICP recognize themselves. | 2–3 sentences |
| **The turn** | What changed, what was built, what was decided — and what the exec believes about it. | 2–3 sentences |
| **Proof anchor** | One hard fact or named customer from the release. Mark `[verify]` if unconfirmed. | 1 sentence |
| **Invitation** | A low-commitment ask: read the story, DM for context, tell me if you've seen this. Link at the end only. | 1–2 sentences |

Voice constraints: plain prose, no bullet points, no headers, no em-dash clusters, no emoji
unless brand allows. LinkedIn renders markdown as literal characters — deliver as plain text.
Do not start with "I'm excited/proud/thrilled/honored."

---

## Step 3b — X thread (hook tweet + 4–6 reply tweets + CTA tweet)

**Framework: Hook → Stakes → Reveal → Evidence → Implication → CTA**

Each tweet is self-contained (can be screenshot-shared alone) but builds the narrative.

| Tweet | Job | Char guidance |
|---|---|---|
| **1 — Hook** | The single most surprising or specific fact. Pattern: "[Number/Name] [unexpected verb]. Here's what that means:" | ≤240, ends with a thread signal |
| **2 — Stakes** | Why this matters to the audience right now. Set the market context. | ≤240 |
| **3 — Reveal** | What was announced, in plain language, stripped of PR-speak. | ≤240 |
| **4 — Evidence** | The hardest number or the most credible proof from the release. | ≤240 |
| **5 — Implication** | What this means for the reader's workflow, business, or world. | ≤240 |
| **6 — CTA** | One link, one ask. "Full story:" / "Thread summary:" / "If this resonates, RT tweet 1." | ≤240 |

Vary sentence length across tweets. No corporate stacking of adjectives. Use the brand's voice
register — the thread should sound like the same person writing the LinkedIn post, not a different
department.

---

## Step 3c — Owned-media blog post (500–700 words)

**Framework: ABCD — Angle, Bridge, Case, Direction**

This is not a press-release reprint. It is a thought-leadership or product-education article
triggered by the announcement. The announcement is the news hook; the value is the insight.

| Section | Job | Word target |
|---|---|---|
| **SEO headline** | Target keyword + news hook. Pattern: "[Outcome/Category] — [Brand] [News Verb]" | 55–65 chars |
| **Meta description** | One-sentence benefit + keyword. No truncation past 155 chars. | ≤155 chars |
| **Intro** | Lead with the reader's problem or the industry tension. Name the news in sentence 2–3. | 60–80 words |
| **Bridge** | Why this announcement is the response to that tension. Context, not repetition. | 80–100 words |
| **Case / body** | 2–3 short sections. Use the best exec quote as a pull-quote. Anchor every claim in the release; `[verify]` anything else. | 200–300 words |
| **Direction / CTA** | What the reader should do next. Tie back to the offer mechanics from brand-brain. | 60–80 words |

**Internal links:** add 2–3 internal links minimum — to the product page, a related blog post, and
a case study or proof asset if one exists in the brand's `brand.md` or `proof.md`. Mark
`[internal link — insert URL]` if the target URL is unknown.

**SEO note:** target the keyword that a potential customer would search *when they first hear
about this category of news* — not the brand name alone. If the user has a target keyword, use it;
otherwise derive from the ICP and the news hook. For a full keyword pass, call
`on-page-seo-optimizer` after drafting.

---

## Step 5 — Deliver and save

Output all three assets in sequence with clear H2 separators. Then save:

```
./pr/<release-slug>/linkedin-exec-post.md
./pr/<release-slug>/x-thread.md
./pr/<release-slug>/blog-post.md
```

`<release-slug>` = kebab-cased version of the announcement headline. Confirm save paths to the
user. Do not save into the skill folder.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No asset drafted before the brand returns. Voice + banned-words are hard
  overrides, not suggestions.
- **One story, three reframings.** The spine is constant; the channel, register, and hook differ.
  Never paste the same copy across all three.
- **No invented proof.** Every stat, customer name, and claim must appear in the press release or
  brand-brain's verified proof. Everything else is `[verify]`.
- **Exec voice is human, not corporate.** The LinkedIn post especially must not read like a
  lightly edited press release. If it does, run `de-slop-humanize-pass`.
- **SEO serves the ICP, not the headline.** Blog headline and meta target a search query a real
  buyer types, not the brand's internal framing of the announcement.
- **Internal links are mandatory.** The blog post creates no orphan content. Minimum 2 links.

---

## What Not to Do

- Don't start the LinkedIn post with "I'm thrilled/excited/proud/honored to announce."
- Don't reprint the press release boilerplate in the blog post body.
- Don't use markdown (bullets, headers, bold) inside the LinkedIn post.
- Don't invent quotes, customers, or metrics absent from the release.
- Don't tweet the same sentence twice across the thread.
- Don't skip the story-spine sentence (Step 2) — it is the quality gate.
- Don't produce anything before `brand-brain` returns the active brand.

---

## Quality Checklist (self-review before delivering)

- [ ] `brand-brain` called; voice + banned-words honored across all three assets?
- [ ] Story-spine sentence written and all three assets are retellings of it?
- [ ] LinkedIn: first-person, 150–200 words, no markdown, no "thrilled/excited" opener?
- [ ] X thread: hook tweet is specific and surprising; each tweet standalone-shareable; ≤240 chars each?
- [ ] Blog post: 500–700 words; SEO headline ≤65 chars; meta ≤155 chars; ≥2 internal links; no press-release rephrasing?
- [ ] Every number and claim traceable to the release or brand-brain's verified proof (else `[verify]`)?
- [ ] All three files saved to `./pr/<release-slug>/`?
