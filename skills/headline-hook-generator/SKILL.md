---
name: headline-hook-generator
description: >
  Turns a draft title, topic, or rough idea into ranked headline variants plus matching opening-hook
  sentences across distinct persuasion angles — for blog posts, articles, landing-page heroes, email
  subject lines, ad headlines, and social posts. Two modes: by default it takes a topic (or a weak
  headline) and returns one recommended headline + hook with 2–3 alternates; on request it produces a
  full spread of angle-distinct variants, each scored and ranked, with an A/B recommendation. Every
  variant is on-voice and honest because it calls the `brand-brain` skill to load the active brand's
  voice, banned words, ICP + awareness stage, offer, and real proof — it does not re-derive brand
  context or invent numbers. Use whenever the user says "write a headline," "give me headline
  options," "title ideas," "improve this headline/title," "write a hook," "opening line," "subject
  line," "headline variants," "make this more clickable," or hands over a topic/draft and asks what to
  call it. It writes the title and the first sentence — not the rest of the article.
---

# Headline & Hook Generator

Give it a topic or a limp title; get back the headline that earns the click *and* the first sentence that earns the second one. A headline gets the open; the hook keeps it — most generators write one and forget the other, so the click bounces. This skill always ships them as a **pair**: headline + opening hook, matched in promise so the reader who clicks is the reader who stays.

Every variant is written in the brand's real voice, aimed at the brand's real ICP at the right awareness stage, and backed only by real proof — because brand context comes from the shared `brand-brain` skill, not from guessing. This skill writes the title and the first sentence. It does not write the body, restructure the offer, or fix a piece with nothing to say — a great headline on an empty article is a bounce with extra steps.

---

## Skills this calls

- **`brand-brain`** (required) — resolves and loads the active brand's voice adjectives, banned words, ICP + awareness tendency, offer, positioning, and real proof. Headlines do not implement brand scanning/interviewing/storage; that lives in `brand-brain`, once.
- *(optional, when installed / when the job needs them)*
  - **`proof-vault`** — to pull the real stat, customer, or result that makes a specificity/proof-angle headline true rather than `[verify]`.
  - **`editorial-style-guide`** — for the brand's title-case vs. sentence-case rule, number formatting, and product-name casing so headlines render house-correct.
  - **`cta-variant-generator`** — when the "headline" is really a hero/CTA block (eyebrow + headline + button), hand the action line to the CTA skill rather than improvising it here.
  - **`content-brief-builder`** — if a brief already exists, pull its angle, primary keyword, and intent so the title matches the page's strategy.

Synthesize inline only when a needed component is unavailable.

---

## How a run works

```
Step 0  Load the brand   ──► call the `brand-brain` skill (it bootstraps on first use)
Step 1  Pick the surface ──► blog · landing hero · email subject · ad · social — sets limits + intent
Step 2  Pick the mode    ──► Quick (default) | Spread (on request)
Step 3  Generate + self-score, then present (and offer to persist a Spread)
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the request and any named brand. It returns the digest — voice adjectives, banned words, offer mechanics, real proof, positioning line, ICP + awareness tendency — and the `brand.md` path. Obey voice + banned-words as **hard overrides**, use only the returned real proof (anything else is `[verify]`), and aim every variant at the brand's ICP. **Do not write a headline until brand-brain returns.**

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user to install `brand-brain` (preferred) or answer a 3-question mini-setup (what it is · ICP + awareness stage · 3 voice adjectives + banned words), then proceed.

### Step 1 — Pick the surface

The surface sets the hard constraints. Never write a headline without knowing where it lives.

| Surface | Headline length | What the hook does | Tuning notes |
|---|---|---|---|
| Blog / article title | ~50–60 chars (SEO title); H1 can run longer | First 1–2 sentences pay off the title and create a curiosity gap | Lead with the keyword for SEO; the H1 can be punchier than the SEO title |
| Landing-page hero | 6–12 words, scannable | Subhead names who it's for + the payoff | Message-match the ad/source that sent them; benefit over cleverness |
| Email subject line | ~30–50 chars (mobile-first) | Preview/preheader extends the subject, never repeats it | Subject + preheader are *one* unit; specificity beats hype in the inbox |
| Ad headline | platform limit (Google ~30, Meta varies) | The first line of body copy is the hook | Match the destination's promise exactly; honest claims only |
| Social post | platform-native | Line 1 is the scroll-stopper *before* the "see more" fold | Front-load the hook; no clickbait the post can't cash |

### Step 2 — Pick the mode

- **Quick mode (default).** Topic/draft in → *the* headline + hook out (not a menu). One recommended pair + 2–3 different-angle alternates + a one-line why.
- **Spread mode.** Triggered by "variants," "options," "give me 10," "A/B," "test," or a count → the full angle-spread table, each scored and ranked, with an A/B pick.

When unsure, default to Quick and offer Spread at the end.

---

## HOOKED — our working scoring model (every headline is scored on six dimensions)

HOOKED is a house mnemonic we use here, not an established external framework. A headline that wins is rarely the cleverest — it's the one that scores across all six. Use HOOKED both to *generate* (each letter is a lever to pull) and to *rank* (1–5 per dimension; the top total wins, with judgment).

| Letter | Dimension | The question | Failure mode it catches |
|---|---|---|---|
| **H — Hook the desire** | Does it speak to a real ICP want, fear, or job-to-be-done? | Generic, no clear "this is for me" |
| **O — Obvious benefit** | Is the payoff/transformation clear in one read? | Clever but vague; reader can't tell what they get |
| **O — On-voice** | Does it sound like the brand (and dodge banned words)? | AI-hype voice; off-brand register |
| **K — Keyword & match** | (Blog/ad) keyword present and front-loaded? Message-matches the destination? | Ranks poorly or breaks the scent trail after the click |
| **E — Evidence** | Is any number/claim *real* (a proof point), or is it `[verify]`? | Fabricated specificity that erodes trust |
| **D — Distinct angle** | Is this a different *motivation*, not a reworded twin? | Ten ways to say the same thing |

Score honestly. A 5 on cleverness with a 2 on Obvious benefit loses to a plain headline that's clear. The hook sentence inherits the same promise — score the *pair*, not the headline alone.

### Angles to vary (motivation, not vocabulary)

Pull from these to keep variants genuinely distinct — never ten synonyms of one idea:

- **Benefit / outcome** — the result they get.
- **Specificity / number** — a real figure (from `proof-vault`; else `[verify]`).
- **Curiosity gap** — an open loop the hook closes (honest, not bait).
- **How-to / mechanism** — the method or steps.
- **Contrarian / pattern-interrupt** — challenges the assumed wisdom.
- **Question** — names the reader's exact doubt.
- **Social proof / authority** — who else, real and attributed.
- **Loss aversion / mistake** — the cost of not solving it.
- **Identity / transformation** — who they become.

### Writing the hook (the half everyone skips)

The opening line is a separate craft. It must:
1. **Pay off the headline's promise immediately** — same scent; never a bait-and-switch.
2. **Open a loop** the next sentences close — a stat, a stakes statement, a sharp question, a vivid specific.
3. **Earn the next line** — short, concrete, no warm-up ("In this post we'll explore…" is banned). Cut throat-clearing.

Give one hook per recommended headline; in Spread mode, hooks for the top 2–3 only unless asked for all.

---

## Quick mode (default)

1. **Read the topic/draft** (and brief, if one exists) for the single promise. If improving an existing headline, **diagnose in one line** — vague benefit, off-voice, banned word, no keyword, over limit, fake specificity, clickbait.
2. **Fix the awareness ceiling** from the ICP: an unaware reader needs curiosity/benefit, not a feature; a product-aware reader can take a sharper, more direct claim.
3. **Write the recommended pair** — headline + opening hook, on the strongest angle for this surface and audience.
4. **Add 2–3 alternates on *different* angles** (not synonyms) so there's something to test.
5. **One line of why** it fits this surface + audience. For improvements, show **before → after** and name what changed.

Quick output stays short — the pair, alternates, one-line rationale. No big scoring table unless asked.

---

## Spread mode (on request)

1. **Sanity-check the substance (gate).** A headline can't save a piece with no real promise or proof. If the topic is thin, say so and offer to sharpen the angle first — don't manufacture clickbait over emptiness.
2. **Generate ≥8 headlines across ≥6 distinct angles**, every one inside the surface's char limit and on-voice.
3. **Score each on HOOKED** (1–5 per dimension) and rank by total, using judgment on ties.
4. **Write hooks for the top 2–3.**
5. **Recommend an A/B pair** — the top scorer plus a deliberately *different-angle* Variant B (e.g. benefit vs. curiosity), each with a one-line rationale and the hypothesis the test resolves ("does specificity beat curiosity for this list?").

```
## Headlines — [topic / surface]
Context: [brand · ICP · awareness stage · surface · char limit · keyword if any]
Brand: [slug, via brand-brain]
[⚠ thin-substance note, if any]

| # | Headline | Angle | Chars | H | O | O | K | E | D | Total |
|---|---|---|---|---|---|---|---|---|---|---|

### Hooks (top picks)
#_  [headline]
    Hook: [1–2 sentences that pay off the promise and open a loop]

### Recommendation
Primary #_ · Variant B #_ (different angle) · You'll learn: [hypothesis] · [verify] items: […]
```

---

## Persistence

Quick output is inline. For a Spread, offer to save to `./headlines/[slug]-[topic].md` (create the dir if needed) — **never** inside the skill folder, never touching `brand.md`. Offer; don't assume.

---

## Principles

- **Brand-brain first.** No headline before `brand-brain` returns. Its voice + banned-words override everything here.
- **Always ship the pair.** Headline + hook, matched in promise. A headline without its hook is half a job.
- **Vary motivation, not vocabulary.** Distinct angles, not synonyms of one idea.
- **Specificity beats hype — when it's true.** A real number from `proof-vault` outperforms a superlative. A fake number is fraud; mark unconfirmed claims `[verify]`.
- **Match the surface and the awareness.** Char limits are hard; never ask for more belief than the stage supports.
- **Message match is sacred.** The headline keeps the promise the destination (or ad/subject) must pay off.
- **No clickbait the body can't cash.** The curiosity gap must be honest and closable.

## What not to do

- Don't write headlines before `brand-brain` returns the active brand.
- Don't reimplement brand scanning/interviewing/storage — call `brand-brain`.
- Don't write the article body, the offer, or the CTA button (hand CTA blocks to `cta-variant-generator`).
- Don't invent stats, customers, results, or superlatives; don't present `[verify]` items as confirmed.
- Don't exceed the surface's char limit or use banned words / emojis / exclamation hype unless the brand allows them.
- Don't hand back near-identical variants — if you can't find distinct angles, say the topic is too thin to spread.
- Don't open a hook with throat-clearing ("In this article…", "We all know that…").

## Quality checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded (or bootstrapped) before any headline?
- Surface identified, with its char limit and intent applied?
- Voice + banned-words honored; only real proof used (rest `[verify]`)?
- Every recommended headline shipped **with** a matching opening hook that pays off the promise?
- Quick: one recommended pair + 2–3 different-angle alternates + a one-line why? (Improve: before → after + what changed?)
- Spread: ≥8 headlines, ≥6 distinct angles, each HOOKED-scored and ranked; primary + different-angle Variant B with a stated hypothesis?
- No variant exceeds the char limit; message match addressed; thin substance flagged not hidden?
- Persistence to `./headlines/` offered for a Spread (never the skill folder, never `brand.md`)?
