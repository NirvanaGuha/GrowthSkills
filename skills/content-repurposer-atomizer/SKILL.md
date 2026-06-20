---
name: content-repurposer-atomizer
description: >
  Turns one long-form asset — a blog post, webinar, podcast, case study, report, or talk — into a
  full set of platform-native versions in one pass: LinkedIn post, X thread, Instagram carousel,
  Threads, Facebook post, a nurture email, and a short-video script. It does NOT summarize seven
  times; it EXTRACTS the source's atomic ideas once, then rewrites each for the channel's format,
  length, hook style, and reader intent — every version on the brand's voice because it calls the
  `brand-brain` skill for voice, banned words, ICP, offer/destinations, and real proof (no invented
  stats). Use whenever the user says "repurpose this," "atomize this post," "turn this into social,"
  "make a LinkedIn/X/Instagram version," "spin this into a thread," "break this into posts," "create
  a content distribution pack," "what can I make from this article," or hands over a long asset and a
  list of channels. It rewrites per channel — it does not change the source's facts or claims.
---

# Content Repurposer & Atomizer

Hand it one long asset and the channels you publish on; get back a distribution pack — each piece written for *that* platform, not the same paragraph reformatted seven ways. The hard part of repurposing isn't shortening; it's knowing which atomic idea earns a post, which format the platform rewards, and how to keep every version on-voice. This skill does all three. Brand voice, banned words, offer, and real proof come from the shared `brand-brain` skill, so the LinkedIn post sounds like the brand and the email points at the right URL.

It rewrites; it does not relitigate the source. It will not invent a stat the source didn't make, change a claim, or fabricate a customer. If the source is too thin to atomize, it says so rather than padding seven weak posts out of one good paragraph.

---

## Skills this calls

- **`brand-brain`** (required) — resolves and loads the active brand's voice adjectives, banned words, ICP + awareness tendency, offer mechanics + destination URLs, positioning, and real proof. This skill does not re-derive brand context; that lives in `brand-brain`, once.
- *(optional, when installed / when a piece naturally needs them)*
  - **`cta-variant-generator`** — for the closing CTA on the email, LinkedIn post, and short-video end card, in the brand's real offer + voice. Synthesize a simple CTA inline if absent.
  - **`proof-vault`** — to pull the brand's real stats/quotes/logos when a piece leans on proof. Use only confirmed proof; mark the rest `[verify]`.
  - **`editorial-style-guide`** — for the brand's mechanical rules (capitalization, product-name casing, number/date format, hashtag policy) so every version renders the brand consistently.
  - **`positioning-messaging-architect`** / **`icp-persona-builder`** — already folded into `brand.md` via brand-brain; call directly only when a channel needs deeper persona language.

Synthesize a section inline only when a needed component is unavailable.

---

## How a run works

```
Step 0  Load the brand   ──► call the `brand-brain` skill (it bootstraps on first use)
Step 1  Intake           ──► get the source asset + the target channels (+ goal/CTA)
Step 2  ATOMIZE          ──► extract the source's atomic ideas once (the Atom Bank)
Step 3  ADAPT            ──► rewrite the right atoms per channel to spec (the Spin Matrix)
Step 4  Self-review, then present / persist the pack
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request (and any named brand). It returns the digest — voice adjectives, banned words, offer mechanics + destination URLs, real proof, positioning, ICP + awareness tendency — and the path to `brand.md`. If the brand is new, `brand-brain` bootstraps it before returning; **do not repurpose anything until it returns.** Obey voice + banned-words as hard overrides; use only real proof (else `[verify]`); point CTAs at the brand's real destinations.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if none exists, ask the user to install `brand-brain` (preferred) or answer a 4-question mini-setup (what it is · ICP + awareness · offer + destination · 3 voice adjectives + banned words), then proceed.

### Step 1 — Intake

Get two things; ask only if missing:
- **The source asset** — paste, file path, or URL. Read it fully. If it's a transcript (webinar/podcast/talk), expect filler and tangents — atomize the *ideas*, not the verbatim.
- **The target channels** — default pack is LinkedIn, X, Instagram, Threads, Facebook, email, short-video. Honor any subset the user names.
- *(optional)* **Goal + CTA** — awareness vs. nurture vs. conversion; the destination URL. Default to the brand's primary CTA + destination.

---

## The Atomic Spin method

Repurposing fails when people **reformat** (same text, new wrapper) instead of **atomize-then-adapt**. The method has two moves, in order — never skip the first.

### Step 2 — ATOMIZE: build the Atom Bank (do this once)

Read the source and pull its **atoms** — the smallest standalone units of value. Each atom is a thing a reader could find useful on its own. Don't write posts yet; just inventory. Tag each atom by type, because type determines which channel it suits.

| Atom type | What it is | Strongest on |
|---|---|---|
| **Thesis / hot take** | The one argument the asset makes | LinkedIn, Threads, X opener |
| **Stat / data point** | A real number from the source | X, Instagram, LinkedIn hook |
| **Framework / steps** | A named model, list, or process | Carousel, X thread, email |
| **Story / example** | A customer, scenario, or anecdote | LinkedIn, video, email |
| **Quote / soundbite** | A line that stands alone | Instagram, X, video B-roll |
| **Contrarian / myth-bust** | "Everyone thinks X, actually Y" | LinkedIn, Threads, X |
| **How-to / tactic** | A concrete do-this | Carousel, email, video |
| **Definition / explainer** | Clarifies a term or concept | Threads, Instagram, X |

Aim for 6–12 atoms from a solid long-form asset. **Truth rule:** every stat/quote/customer atom must trace to the source. If the source didn't say it, it's not an atom — don't manufacture one.

**Thin-source gate.** If you can't pull at least ~4 distinct atoms, the source is too thin to atomize well. Say so, name what's missing (no data? no story? one idea repeated?), and offer to repurpose the *one* strong angle instead of forcing seven weak posts.

### Step 3 — ADAPT: run the Spin Matrix (rewrite per channel)

Take the right atoms for each channel and rewrite to that channel's **format, hook, length, and reader intent** — never paste the same text twice. Lead with the hook that platform rewards; match the brand's voice; respect the channel's mechanics.

| Channel | Best atoms | Format & length | Hook / first line | Mechanics |
|---|---|---|---|---|
| **LinkedIn** | thesis, story, contrarian | 1 idea, ~150–250 words, line breaks, no link in body | Pattern-interrupt or stakes line; payoff after the "see more" fold | CTA/link in first comment or end; 0–3 hashtags; plain prose (no markdown — LinkedIn renders `#`/`**` literally) |
| **X / thread** | stat, framework, hot take | Single post (<280) OR 5–9-tweet thread; one idea per tweet | Tweet 1 = the whole promise; no "a thread 🧵" filler | Numbered or hook-chained; last tweet = recap + CTA |
| **Instagram** | framework, quote, how-to | 6–10-slide carousel script (slide = one beat) OR single caption | Slide 1 = scroll-stopper claim; slide 2 = stakes | Caption ≤~125 chars before fold; CTA "link in bio"; alt-text per slide |
| **Threads** | hot take, definition, contrarian | 1–3 short posts, conversational | A claim or question that invites replies | Looser/chattier than LinkedIn; minimal hashtags |
| **Facebook** | story, how-to, stat | 1 post, ~80–150 words, plain | Relatable framing; benefit-forward | Link OK in body; conversational; 0–1 hashtag |
| **Email** | framework, story, how-to | Subject + preview + ~150–300-word body + 1 CTA | Subject = curiosity/benefit (≤~45 chars); preview extends it | One CTA button → brand's real destination; skimmable |
| **Short-video** | thesis, story, tactic | 30–60s script: HOOK (0–3s) · BODY (3 beats) · CTA | Spoken hook in first 3s or the scroll is lost | Mark [on-screen text] + a caption; end card = CTA |

**Cross-pack discipline:**
- **Vary the hook across channels.** Don't open all seven with the same line — different platforms reward different first moves.
- **Don't atomize the same atom into every channel.** Spread the atoms so the pack feels like a campaign, not an echo. The thesis can anchor LinkedIn; the stat can anchor X; the framework can anchor the carousel.
- **Message-match the CTA** to the goal and the brand's real destination (call `cta-variant-generator` for the conversion pieces).
- **Honor the style guide** — product-name casing, number format, hashtag/emoji policy come from the brand, not from platform defaults.

### Step 4 — Present / persist

Present the pack channel by channel, each labeled and ready to paste, with a one-line "atom used + why this hook." Then **offer to save** the pack to a sensible project path — e.g. `./repurposed/[source-slug]-pack.md` — never inside the skill folder, never overwriting `brand.md`. Offer a posting-order suggestion (which to publish first, what to stagger) if the user wants a mini distribution plan.

```
## Repurposing pack — [source title]
Source: [path/URL]  ·  Brand: [slug, via brand-brain]  ·  Goal: [awareness/nurture/convert]
[⚠ thin-source note, if any]

### Atom Bank
| # | Atom | Type | Source location |

### LinkedIn   (atom #_ · hook: …)
### X / thread (atom #_ · hook: …)
### Instagram  (atom #_ · …)
### Threads · Facebook · Email · Short-video …

### Suggested posting order  (optional)
```

---

## Principles (Non-Negotiable)

- **Brand-brain first.** Nothing is repurposed before `brand-brain` returns. Its voice + banned-words override everything here.
- **Atomize before you adapt.** Inventory the ideas once; never reformat the same paragraph seven times.
- **Native, not ported.** Each version is written for its platform's format, hook, and length — a LinkedIn post is not a tweet with line breaks.
- **One idea per piece.** Each social post carries one atom, fully landed; don't cram the whole article into one post.
- **Truth travels.** Every stat/quote/customer must trace to the source or the proof vault; no invented numbers. Unconfirmed proof is `[verify]`.
- **Vary the hooks.** The pack is a campaign, not an echo — different first lines, different atoms, spread across channels.
- **Source is canonical.** Rewrite for the channel; never change the source's claims or facts.

## What Not to Do

- Don't repurpose before `brand-brain` returns the active brand.
- Don't reimplement brand scanning/interviewing/storage — call `brand-brain`.
- Don't paste the same text into multiple channels or open them all with the same hook.
- Don't invent stats, quotes, customers, or differentiators the source didn't contain.
- Don't put markdown in LinkedIn/Threads output (`#`, `**`, backticks render literally) or exceed a channel's limit.
- Don't use emojis/hashtags the brand bans; don't force seven posts out of a thin source — flag it instead.
- Don't write the pack inside the skill folder or overwrite `brand.md`.

## Quality checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded (or bootstrapped) before any output?
- Atom Bank built first (≥4 atoms, each tagged + traced to the source); thin source flagged if not?
- Every requested channel produced, each on its own format/hook/length spec — not a reformat of one paragraph?
- Hooks vary across channels; atoms spread so the pack reads as a campaign, not an echo?
- Voice + banned-words honored; style-guide mechanics applied; LinkedIn/Threads in plain prose?
- Only real proof used (rest `[verify]`); CTAs point at the brand's real destination?
- Offered to save the pack to a project path (never the skill folder, never `brand.md`)?
