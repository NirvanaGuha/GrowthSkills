---
name: youtube-description-channel-seo-optimizer
description: >
  Takes a video transcript or outline plus channel context and produces a fully
  SEO-optimized YouTube description with timestamped chapters AND channel-level
  recommendations covering tags, playlists, end-screen CTAs, and keyword
  clustering strategy. Runs the YouTube Description Framework (hook paragraph,
  SEO keyword zone, chapters, social links, CTA stack) against YouTube's real
  ranking signals — click-through rate anchors in the first 2 lines, keyword
  density in the first 150 characters, chapter markers that extend session
  time, and tag/playlist signals that feed the recommendation algorithm.
  Brand context comes from the `brand-brain` skill; sibling skills
  `keyword-research-clustering-suite`, `internal-linking-planner`, and
  `content-repurposer-atomizer` are called when available. Use whenever the
  user says "write a YouTube description," "optimize my YouTube channel,"
  "add chapters to my video," "improve my video SEO," "YouTube tags," "end
  screen copy," "video description template," or pastes a transcript and asks
  for help with YouTube.
---

# YouTube Description & Channel SEO Optimizer

Video transcript or outline in. SEO-ready description with chapters, a channel-level tag and playlist strategy, and end-screen CTA copy out. Everything is grounded in the brand's real voice and offer — because YouTube is still a search engine, and generic descriptions leave free organic traffic on the table.

This skill covers two surfaces: the **per-video description** (the primary job) and the **channel-level SEO layer** (tags, playlists, end-screen, keyword clustering) when channel context is available. It does not script the video itself — use `short-form-video-script-writer` or `tiktok-script-hook-generator` for that.

---

## Skills this calls

- **`brand-brain`** (required) — loads voice, banned words, offer mechanics, and real proof. No description is written before it returns.
- **`keyword-research-clustering-suite`** (when available) — supplies keyword volume and intent data for the SEO zone and tags.
- **`internal-linking-planner`** (when available) — surfaces blog/landing-page URLs worth linking in the description.
- **`content-repurposer-atomizer`** (optional) — if the user wants the transcript repurposed as a blog post, LinkedIn post, or short-form clip alongside the description.
- **`cta-variant-generator`** (optional) — for end-screen and description CTA copy when a deeper CTA pass is requested.

---

## How a run works

```
Step 0  Load brand             ──► call brand-brain skill
Step 1  Parse inputs           ──► transcript/outline, target keyword, channel context
Step 2  Keyword triage         ──► primary, secondary, LSI — from clustering skill or inlined
Step 3  Build the description  ──► 5-zone YouTube Description Framework
Step 4  Chapter extraction     ──► timestamps from transcript, rewritten as SEO titles
Step 5  Channel-level layer    ──► tags, playlist, end-screen (when channel context given)
Step 6  Self-review + output
```

---

## Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's voice adjectives, banned words, offer mechanics, real proof, and ICP. Use these as hard overrides in every zone of the description — especially the hook paragraph and CTA stack.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If none exists, ask the user to install `brand-brain` (preferred) or provide a 4-question mini-setup (what the channel is · ICP + their stage · primary offer + destination URL · 3 voice adjectives + banned words), then proceed.

---

## Step 1 — Parse the inputs

Collect before writing:

| Input | Where it comes from |
|---|---|
| Transcript or outline | User-provided (paste or file) |
| Target keyword (primary) | User-named OR inferred from the transcript's first 90 seconds |
| Channel context (optional) | Description of channel niche, top videos, existing playlists, subscriber count |
| Destination URL | From `brand-brain` offer mechanics or user-provided |
| Monetization / CTA priority | Organic sign-up? Product demo? Newsletter? Affiliate link? Ask if ambiguous |

If the transcript is long, skim it for the core argument, the moment of highest information density, and any on-screen demonstrations (these become chapter markers). Do not ask the user to summarize for you.

---

## Step 2 — Keyword triage (YouTube Description Framework: pre-work)

YouTube ranks descriptions by:
1. Primary keyword in the **first 2 lines** (visible without "Show more") and again near the end.
2. Secondary and LSI keywords in the first 150 characters and distributed across the first 300 characters.
3. Tag relevance — tags that mirror description language reinforce ranking signals.
4. Chapter keywords — each chapter title is an independent search anchor.

Call `keyword-research-clustering-suite` if available, passing the transcript topic and niche. Inline the result. If the skill is absent, derive a 3-tier keyword list from the transcript:

- **Primary** (1): highest-volume, clearest match to the video's single thesis — goes in line 1 and the chapter title for the video's main segment.
- **Secondary** (2–4): intent variants and sub-topics that appear in the video.
- **LSI** (5–10): semantic neighbors that YouTube's algorithm associates with the niche. Mark unverified volume estimates `[verify]`.

---

## Step 3 — The 5-Zone YouTube Description Framework

Write each zone in order. Output them as a single copyable block (no zone headers inside the output block — those are for your reference only).

### Zone 1 — Hook paragraph (lines 1–3, ~150 chars visible before "Show more")

- **Line 1:** Primary keyword + the specific outcome or pain this video resolves. This is the click-through anchor — it has to earn the click from search results where only 2–3 lines show.
- **Line 2:** One concrete proof point or stakes sentence (what the viewer walks away knowing/able to do).
- **Line 3:** Soft CTA or curiosity bridge ("In this video you'll see…" / "Watch to the end for…").

Voice: match the brand's register exactly. If the brand is conversational, these lines are conversational. If it's authoritative/technical, cut the filler.

### Zone 2 — SEO keyword zone (lines 4–10, ~200–300 chars)

2–4 sentences that naturally deploy the secondary and LSI keywords. This is not a keyword dump — it reads as a plain-English elaboration of what the video covers. Think of it as the expanded pitch: the searcher who reads this far is deciding whether to watch.

### Zone 3 — Chapter markers (timestamped)

Extract from the transcript or outline. Format exactly as YouTube requires:
```
0:00 Intro
1:45 [Chapter title — secondary keyword where natural]
…
```

Rules:
- First chapter **must** start at `0:00`.
- Minimum 3 chapters to activate the chapters UI; 5–10 is the sweet spot for a 10–20 min video [verify exact chapter minimum with YouTube docs if this changes].
- Each chapter title is an independent search anchor — rewrite bland titles ("Part 2") into keyword-bearing phrases ("How to Set Up [X] in 5 Minutes").
- Chapters extend average view duration by letting viewers jump to relevant segments — this is a direct ranking signal.

### Zone 4 — Links and resources (prose + bullet list)

- Primary destination link (from `brand-brain` offer mechanics, e.g. free trial, lead magnet, product page).
- 1–2 related videos or playlists on the channel (link directly, boosting session time — a top YouTube ranking factor [verify]).
- Blog post or landing page, if `internal-linking-planner` surfaces a match.
- Call out each link with a plain label ("Free trial: [URL]" not "Click here for more").

### Zone 5 — CTA stack + boilerplate (final ~100 chars)

Standard subscribe/like/notify ask, plus the brand's primary conversion CTA. Keep it direct; no exclamation marks unless brand voice explicitly permits them. End with relevant hashtags (3–5 max — YouTube's tag system does the heavy lifting; description hashtags are secondary [verify current best practice]).

---

## Step 4 — Chapter extraction detail

When a transcript is provided:
1. Identify topic shifts and demo moments by paragraph breaks, time cues, or subject changes.
2. Map each to the nearest clean timestamp (round to the nearest 15 seconds for clean UX).
3. Rewrite the chapter title to include a keyword where it reads naturally.
4. Flag any chapter under 60 seconds as a candidate to merge — very short chapters can feel choppy and may not register in the chapters carousel.

When only an outline is provided:
- Use the outline's H2/H3 structure as chapter anchors.
- Estimate durations based on typical speaking pace (~130 words/minute) and flag them as estimates.

---

## Step 5 — Channel-level SEO layer (when channel context is provided)

This layer is additive. Run it when the user provides channel context (niche, existing videos, playlists) or explicitly asks for channel-level help.

### Tag strategy
YouTube tags are a weak direct ranking signal but a strong semantic association signal — they tell the algorithm what category your content belongs to, which feeds the "recommended next" carousel. The real algorithm leverage is through description and title, but tags reinforce it.

Build a 3-tier tag set (deliver as a copyable comma-separated list for YouTube Studio):
- **Tier 1 (5 tags):** exact-match primary and secondary keywords from Step 2.
- **Tier 2 (5–8 tags):** niche/category terms that describe the channel (not just this video).
- **Tier 3 (3–5 tags):** brand name + branded keyword variants (channel name, product name).

Total: 15–18 tags. YouTube shows ~500 characters of tags; do not pad. Mark any estimated-volume tags `[verify]`.

### Playlist recommendations
Group existing videos or planned content into 2–4 thematic playlists. Each playlist:
- Has a keyword-bearing name (playlists are crawled and rank independently [verify]).
- Has a description (50–100 words) with the playlist's primary keyword in the first sentence.
- Surfaces in the description's Zone 4 links for cross-linking.

### End-screen CTA copy
YouTube's end-screen allows 2 elements for the final 5–20 seconds. Write copy for:
- **Subscribe card:** 1 punchy line (on-screen overlay) + voice-over suggestion ("If this was useful, subscribe — I publish [cadence] on [topic]").
- **Featured video/playlist:** the recommended follow-on piece from the playlist strategy above, with a 1-line teaser.

If `cta-variant-generator` is available, call it with the end-screen brief for a deeper pass.

---

## Output format

```
## YouTube Description — [Video title / primary keyword]
[Copyable description block — no zone headers inside]

---

## Chapter markers
[Timestamp list, copyable]

---

## Channel-level SEO (if applicable)
### Tags
[comma-separated list]

### Playlist recommendations
[Table: Playlist name | Theme | Videos to include | Description snippet]

### End-screen copy
[Subscribe card line + VO suggestion]
[Featured video teaser line]
```

Save to `./youtube/[slug]-description.md` if the user asks; otherwise deliver inline.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No description before `brand-brain` returns. Voice + banned words override everything else.
- **First 150 characters earn the click.** Primary keyword and the outcome/pain must both appear before the fold. If they don't fit, the description isn't done.
- **Chapters are SEO units, not just navigation.** Each title is an independent search anchor — rewrite generic chapter names into keyword-bearing phrases.
- **Truth discipline.** Real proof only; unconfirmed statistics or algorithm behavior claims get `[verify]`.
- **Platform-native format.** YouTube's character limits, chapter rules, and tag conventions are real constraints — honor them, don't approximate.
- **Compose, don't duplicate.** Call `keyword-research-clustering-suite`, `internal-linking-planner`, and `cta-variant-generator` when available; don't reimplement their logic inline.
- **Session time is a ranking signal.** Link to related videos and playlists explicitly — every description should keep the viewer on the channel, not send them off-platform prematurely.

---

## What Not to Do

- Don't write the description before `brand-brain` returns the active brand.
- Don't stuff keywords: if a keyword doesn't read naturally in context, drop it from the description and put it in tags instead.
- Don't skip Zone 1 — a clever middle-section description with a weak opening will lose the search click every time.
- Don't invent timestamp values when working from an outline; mark estimates clearly.
- Don't exceed 18 tags or pad with tangentially related keywords — it dilutes tag signal [verify current YouTube tag policy].
- Don't use description hashtags as your primary keyword strategy; they appear at the bottom and have minimal ranking weight compared to the first 150 characters.
- Don't write identical channel boilerplate as Zone 1 — boilerplate belongs in Zone 5.
- Don't promise algorithm behaviors as fact without `[verify]` — YouTube's ranking documentation is partial and the algorithm changes.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded (or fallback executed) before any copy was written?
- Primary keyword in the first visible line AND in Zone 2 AND in at least one chapter title?
- First 150 characters include the primary keyword and a concrete outcome or pain?
- All chapters start at `0:00`, have keyword-bearing titles, and no chapter is under 60 seconds (or flagged for review)?
- Destination URL from `brand-brain` offer mechanics in Zone 4?
- Tags in 3 tiers, total 15–18, no pure filler?
- Any unverified volume data or algorithm claims marked `[verify]`?
- Voice + banned words honored throughout; no invented proof?
- Channel layer (tags, playlists, end-screen) included when channel context was provided?
