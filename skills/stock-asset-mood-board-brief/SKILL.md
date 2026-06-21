---
name: stock-asset-mood-board-brief
description: >
  Turns a campaign concept, tone notes, and a brand palette into two ready-to-hand-to-a-designer
  deliverables: (1) a prioritized stock-asset search brief — platform-specific search strings for
  photos, video clips, and illustrations, ranked by creative priority and annotated with what to
  accept or reject in each result — and (2) a structured mood board brief with 5–8 reference
  image/video directions, each described in enough visual detail that a designer can source them
  without a briefing call. Anchors every direction to the brand's real voice, palette, and ICP
  before writing a single search string. Does NOT generate images — it briefs the search and the
  board so humans and AI image tools work from a deliberate creative direction rather than vibes.
  Use when the user says "put together a mood board," "find me stock for this campaign," "brief the
  designer on visuals," "stock photo brief," "mood board brief," "visual direction for [campaign],"
  "what should the creative look like," or hands over a campaign concept and asks for visual
  references or search guidance. Pairs tightly with creative-brief-image-prompt-crafter (for
  AI-generation prompts) and brand-consistency-auditor (for checking sourced assets against the
  brand guide).
---

# Stock Asset & Mood Board Brief

Campaign concept in, two designer-ready deliverables out: a prioritized stock-search brief and a structured mood board brief. Built on the brand's real voice, palette, and ICP — not on the designer's mood that morning.

This skill briefs. It does not search, scrape, or generate images. It makes the search smarter and the mood board deliberate.

---

## Skills this calls

- **`brand-brain`** (required) — loads the active brand's palette, voice adjectives, banned words, ICP, and positioning. No creative direction before this returns.
- *(optional, when installed)* **`campaign-brief-builder`** — if no campaign brief exists yet, call it first so this skill has a concept to brief against. Synthesize inline if absent.
- *(optional, when installed)* **`creative-brief-image-prompt-crafter`** — pass this skill's mood-board directions to it when AI-generated images (Midjourney/DALL-E/Firefly) are also wanted alongside licensed stock.
- *(optional, when installed)* **`brand-consistency-auditor`** — run selected assets through it before final sign-off.

---

## How a run works

```
Step 0  Load the brand     ──► call `brand-brain`; extract palette, voice, ICP
Step 1  Establish the brief ──► confirm or draft the campaign concept + tone
Step 2  Run the framework  ──► VASTE analysis (below) → creative directions
Step 3  Write the stock-search brief  ──► per platform, per priority
Step 4  Write the mood board brief    ──► 5–8 visual directions, each fully specified
Step 5  Self-review and present
```

---

## Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's digest: palette (hex codes), voice adjectives, banned words, ICP, offer, and positioning. Wait for it before writing any creative direction.

Pull specifically:
- **Palette:** exact hex codes; note warm/cool temperature and any secondary/accent swings
- **Voice adjectives:** these govern the emotional register of every visual direction
- **Banned words/phrases:** check if any map to visual clichés to avoid (e.g. a brand that bans "corporate" should flag handshakes and boardrooms)
- **ICP:** who must see themselves (or aspire to) in these visuals

**Fallback if brand-brain is not installed:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If none exists, ask for: hex palette, 3 voice adjectives + banned words, ICP one-liner, and campaign concept. Prefer the call.

---

## Step 1 — Establish the campaign brief

Confirm or derive: campaign name/concept, primary message, conversion goal, channel(s) where assets will run (hero image, social ad, email header, OOH, etc.), and whether the need is photo, video, illustration, or mixed.

If the user hasn't supplied a concept, ask for it or call `campaign-brief-builder` first. Do not invent the concept.

---

## Step 2 — VASTE Framework (the core methodology)

Every visual direction is built through five lenses. Work through each before writing output.

| Lens | Question to answer |
|---|---|
| **V — Visual register** | What's the lighting mood, depth of field, and color temperature? (golden-hour warmth, flat studio, high-contrast editorial, desaturated muted, etc.) |
| **A — Action / energy** | What is happening? Still/static vs. in-motion; candid vs. staged; close-up detail vs. environmental wide shot |
| **S — Subject** | Who or what is the hero: people, product, environment, abstract texture, data visualization? If people: age, apparent ethnicity composition (ICP-representative), setting |
| **T — Tone & emotion** | The feeling the image should create in the viewer — maps directly to the brand's voice adjectives; any emotional register the brand's banned words implicitly prohibit |
| **E — Exclusion filter** | What to *reject* in search results: stock clichés, off-palette colors, demographic mismatches, compositional styles that clash with the brand's energy |

Running VASTE produces the raw material for both deliverables.

---

## Step 3 — Stock-Asset Search Brief

Produce a prioritized table of search strings per platform. Lead with the highest-priority creative direction.

### Platform conventions (real, not invented)

| Platform | Best for | Search-string tips |
|---|---|---|
| **Unsplash** (free) | Editorial, lifestyle, architecture | Descriptive nouns + adjectives; exclude "business" for authentic results |
| **Pexels** (free) | Diverse lifestyle, product on surface, food | Strong with "aesthetic," mood words, ethnicity descriptors |
| **Getty / iStock** | Premium, commercial-safe, model-released | Use `(concept) AND (emotion) -clip-art -illustration` for photos |
| **Shutterstock** | Broadest catalog; strong for data/tech/B2B | Filter: Authentic collection reduces staged look |
| **Adobe Stock** | Seamless Creative Cloud; strong motion/video | Use `mood:cinematic` or `mood:minimal` in advanced search |
| **Pond5 / Artgrid** | B-roll video, motion backgrounds | Search by shot type: drone, handheld, timelapse |
| **Noun Project** | Icons and simple illustrations | Exact noun searches; Style filter: line vs. solid |

For each creative direction, provide:
- **Direction name** (concise label matching the mood board)
- **Priority** (P1 / P2 / P3)
- **Platform(s)**
- **Search string(s)** — 2–3 variants per platform; specific enough to narrow, not so narrow they return 0 results
- **Accept:** visual signals that confirm the result is right
- **Reject:** red flags to skip immediately

```
## Stock-Search Brief — [Campaign Name]
Brand: [slug, via brand-brain]
Channels: [list]
Asset types needed: [photo / video / illustration]

### Direction 1 — [Name] (P1)
Platform: [e.g. Unsplash, Getty]
Search strings:
  - "[string A]"
  - "[string B]"
Accept: [2–3 visual cues]
Reject: [2–3 red flags]
```

Save to `./creative/[campaign-slug]-stock-brief.md` if the user asks; otherwise present inline.

---

## Step 4 — Mood Board Brief

5–8 visual directions. Each is fully specified so a designer can source assets without a briefing call. Senior art directors write directions this precise; junior marketers rarely do.

Each direction gets:

1. **Direction name** — a short evocative label (not "Image 1")
2. **VASTE spec** — one sentence per lens (≤ 25 words each); total ≤ 125 words per direction
3. **Palette anchor** — which brand hex(es) this direction foregrounds; acceptable neutrals
4. **Composition note** — aspect ratio and safe-zone guidance for the primary channel
5. **Reference description** — describe a specific publicly known image type or visual convention clearly enough to be unambiguous. Do NOT reproduce or name copyrighted images; describe the *type* (e.g., "the overhead flat-lay product photo popularized by DTC beauty brands: white marble surface, soft shadows, one hero product flanked by complementary botanicals")
6. **What this direction does** — one-line strategic rationale: which ICP segment it speaks to and what feeling it's meant to land

```
## Mood Board Brief — [Campaign Name]
Brand: [slug] · Palette: [hex codes] · Voice: [3 adjectives]
ICP: [one-liner from brand-brain]

### Direction 1 — [Name]
Visual register: ...
Action/energy: ...
Subject: ...
Tone/emotion: ...
Exclusion filter: ...
Palette anchor: [hex]
Composition: [aspect ratio, safe zone]
Reference type: ...
Strategic rationale: ...
```

Save to `./creative/[campaign-slug]-mood-board-brief.md` if the user asks.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** Palette, voice, and ICP come from the brand, not from creative instinct. Call it before writing one direction.
- **VASTE every direction.** A direction without all five lenses is incomplete; a designer will fill the gaps with their own assumptions.
- **ICP representation is not optional.** If the brand's ICP is mid-market US eCommerce operators, the people in the visuals should reflect that. Name it.
- **Exclusion filters are as important as search strings.** Stock platforms return thousands of results; the reject list cuts search time by half.
- **No image invention.** Describe visual directions; do not claim specific images exist. Search strings are directional hypotheses, not guarantees.
- **Honest proof only.** Platform conventions stated here are real at time of writing; mark anything platform-specific as `[verify current]` if there's reason to believe it may have changed.

## What Not to Do

- Don't produce visual directions before `brand-brain` returns palette + voice.
- Don't reimplement brand scanning or voice derivation — call `brand-brain`.
- Don't generate images — brief the search and the board; route AI generation to `creative-brief-image-prompt-crafter`.
- Don't write generic directions like "professional woman at laptop" — they return 800,000 results and communicate nothing.
- Don't skip the exclusion filter — it's what separates a useful brief from a stock-photo treasure hunt.
- Don't use off-palette hero colors in any direction unless the brand explicitly allows it; note deviations.
- Don't name or reproduce copyrighted reference images; describe the visual type and convention.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and palette, voice adjectives, banned words, and ICP loaded?
- VASTE run for every direction — all five lenses present?
- Stock-search brief: ≥2 search-string variants per direction, accept + reject specified, platforms matched to asset type?
- Mood board brief: 5–8 directions, each with composition note + palette anchor + strategic rationale?
- ICP representation addressed explicitly in at least one "Subject" lens?
- No invented proof; platform conventions marked `[verify current]` where relevant?
- Palette hex codes used, not color names alone?
- Exclusion filters would actually narrow a search (specific, not "avoid bad photos")?
