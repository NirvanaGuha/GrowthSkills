---
name: creative-brief-image-prompt-crafter
description: >
  Turns a plain-English visual description, campaign brief, or concept note into a
  detailed, tool-specific generation prompt for Midjourney, DALL-E 3, or Stable Diffusion —
  fully grounded in the brand's palette, tone, and banned aesthetics, and structured around
  RECIPE (our working scaffold: Reference style · Environment/setting · Character/subject · Image
  parameters · Palette/lighting · Exclusion list). Works in two modes: Single (one hero
  prompt, tuned and ready to paste) or Batch (a grid of prompt variants across angles,
  styles, or crops for fast visual exploration). Also accepts an existing weak prompt and
  returns a strengthened version with a before→after diagnosis. Every prompt is brand-safe:
  no placeholder palette values, no aesthetic directions that conflict with the brand's style
  guide, and no invented proof or product claims embedded in the visual brief. Calls
  brand-brain for live brand context and infographic-data-viz-spec-writer / stock-asset-mood-board-brief
  when the ask is data-driven or stock-hunt adjacent. Use when the user says "write me an
  image prompt," "midjourney prompt for this campaign," "turn this brief into a DALL-E
  prompt," "stable diffusion prompt," "generate AI image prompts," "I need a visual brief,"
  "prompt for our hero image," or "give me prompt variants to explore."
---

# Creative Brief & Image Prompt Crafter

A plain-English brief in; a production-ready generation prompt out. No more half-baked "a photo of a woman smiling" prompts that produce stock-photo mush. This skill applies RECIPE — our own working scaffold, not an external framework — to translate campaign intent, brand identity, and visual direction into the exact prompt syntax each tool expects — so the first generation is already in the right ballpark.

Every prompt is grounded in the brand before a single token is written. If the brief contradicts the brand's visual identity, this skill flags it rather than silently working around it.

---

## Skills this calls

- **`brand-brain`** (required first) — resolves and loads the active brand's palette, typography tone, banned aesthetics, ICP, and any visual style notes. Do not write a single prompt token before this returns.
- **`infographic-data-viz-spec-writer`** (optional) — when the image is data-driven (chart, stat callout, infographic hero), call this first to get the data spec; use that spec as structured input to the prompt.
- **`stock-asset-mood-board-brief`** (optional) — when the user wants a mood-board-style prompt grid or reference images to anchor the generation direction before prompting.
- **`campaign-brief-builder`** (optional) — when no brief exists yet and the user just has a product/goal; build the brief first, then craft the prompt from it.

---

## How a run works

```
Step 0  Load the brand       ──► call brand-brain; capture palette, banned aesthetics, voice tone
Step 1  Classify the ask     ──► Single | Batch | Improve (default: Single)
Step 2  Clarify if needed    ──► gate on subject, intent, tool target; ask ≤3 questions
Step 3  Apply RECIPE         ──► build each section against brand constraints
Step 4  Format for the tool  ──► Midjourney / DALL-E 3 / SD — each has different syntax rules
Step 5  Self-review          ──► brand-safe? prompt-length compliant? exclusions set?
Step 6  Deliver + offer next ──► paste-ready prompt + one round of iteration offered
```

### Step 0 — Brand context (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). Extract from the returned digest:

- **Palette hex values** — use them in the prompt; never guess colors
- **Banned aesthetics** — phrases or visual directions the brand explicitly avoids (e.g. "stock-photo feel," "clip-art," "neon," "grainy")
- **Style adjectives** — the 3–5 voice/visual tone words that map to visual direction
- **ICP** — who the audience is, which shapes model/photography style choices

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if absent, ask for palette (hex), 3 visual tone adjectives, banned aesthetics, and target tool — then proceed. Prefer the call.

### Step 1 — Mode classification

| Mode | Trigger | Output |
|---|---|---|
| **Single** (default) | One hero image, one campaign asset | One tuned prompt + 1–2 quick alternates on different angles |
| **Batch** | "variants," "grid," "explore," "options," "give me multiple" | 4–6 distinct prompts across angles/styles/crops in a scannable table |
| **Improve** | User pastes an existing prompt | Before → after with a one-line diagnosis of each fix |

When unsure, default to Single and offer Batch at the end.

---

## RECIPE — our working scaffold

Every prompt, regardless of tool, is built from six components. Build each one against brand constraints before assembling.

### R — Reference style
The visual genre, photographic or illustrative movement, and quality anchor. Be specific: "editorial flat-lay on textured linen" beats "product photo." Avoid terms the brand bans (e.g. if the brand is minimal, don't reach for "maximalist editorial"). Include artist/photographer/studio references only if unambiguous and brand-appropriate.

Examples of actionable style descriptors:
- `cinematic documentary photography, Magnum-style street realism`
- `3D render, clean product studio, Octane render, subsurface scattering`
- `editorial illustration, flat-vector, Swiss-grid composition`
- `dark-mode UI screenshot mockup, glass morphism, frosted panels`

### E — Environment / setting
Physical or conceptual space. Light source and time of day belong here. Be specific about depth (tight studio vs. wide environment). Match the brand's spatial vocabulary (a brand that runs dark-background hero shots shouldn't suddenly get outdoor-bright lifestyle).

### C — Character / subject
Who or what is in frame. For product shots: exact product descriptor + materials + surface. For people: age range, affect/emotion (not specific ethnicity unless the brand explicitly specifies), action, relationship to the product. Never invent people who imply social proof claims.

### I — Image parameters
Technical specs that control the model:
- **Midjourney:** `--ar 16:9` (or brand's standard ratio), `--v 6`, `--style raw`, `--q 2`, `--stylize 100–750`
- **DALL-E 3:** natural-language quality descriptors ("photorealistic, 4K, sharp focus"); size hint in the API call
- **Stable Diffusion:** CFG scale direction, sampler note (DPM++ 2M Karras), steps hint, LoRA call if relevant

### P — Palette / lighting
Pull exact hex values from brand.md; translate to prompt-language color names if the model doesn't parse hex (most don't). State the dominant and accent. Lighting temperature matters: "warm golden-hour backlight, f/1.8 bokeh" vs. "cool overcast, even diffusion."

Map palette to lighting synergy — dark navy brand colors pair with rim-lit or neon-accent setups, not washed-out noon sun.

### E — Exclusion list
What to suppress. This is not optional for brand-safe outputs. Always include:
- The brand's banned aesthetics (from `brand.md`)
- Generic no-gos: `--no stock photo watermark, text overlay, blurry background, oversaturated, artificial smile, cluttered background` (adapt for DALL-E/SD negative prompt field)

---

## Tool-specific syntax rules (non-negotiable)

### Midjourney
- Prompt in natural English, comma-separated descriptors, then parameters at end
- Parameters block: `--ar X:Y --v 6 --style raw --q 2 --stylize N`
- `--no` is the exclusion operator; stack items with commas: `--no stock photo, artificial smile, text`
- Max practical prompt: ~60 words before parameters; front-load the subject and style
- Do NOT use colon-weight syntax (`::`weight) unless specifically requested

### DALL-E 3
- Single coherent sentence or short paragraph — not comma lists
- Embed style, subject, environment, lighting in flowing prose
- No parameter syntax; quality and size are API-level settings
- Italicize or bracket the elements you want most emphasized via natural emphasis ("the focus should be on…")
- Works well with instructional tone: "Render a…" / "Create a…"

### Stable Diffusion (SDXL / 1.5)
- **Positive prompt:** tag-style, comma-separated, ordered by weight (subject first, then style, then quality boosters)
- **Negative prompt:** separate field; always include `(worst quality:1.4), (low quality:1.4), (blurry:1.2), watermark, text, signature`
- Quality boosters to append: `masterpiece, best quality, highly detailed, sharp focus, 8k`
- SD is LoRA/checkpoint-sensitive — note the intended model or ask

---

## Single mode — delivery format

```
## Image prompt — [brief descriptor, ≤8 words]
Brand: [slug, via brand-brain]
Tool: [Midjourney | DALL-E 3 | Stable Diffusion]
Use: [where this asset will live — hero, social, ad, etc.]

### Prompt (paste-ready)
[Full prompt, formatted for the target tool]

### Alternates
1. [Different angle — same tool, same brand, one changed variable]
2. [Different style register — e.g. illustration vs. photo]

### Brand compliance notes
- Palette: [hex → color name used in prompt]
- Banned aesthetics avoided: [list from brand.md]
- [Any flag if the brief pushed against brand identity]
```

---

## Batch mode — delivery format

```
## Prompt grid — [campaign/brief descriptor]
Brand: [slug] | Tool: [tool]

| # | Angle / use case | Prompt (paste-ready) | Key variable |
|---|---|---|---|
| 1 | [Hero — lifestyle] | … | Wide environment, person |
| 2 | [Product studio] | … | Tight, no person |
| 3 | [Social square] | … | --ar 1:1 or square crop |
| 4 | [Dark mode / editorial] | … | Low-key lighting variant |
| 5 | [Illustration / vector] | … | Style register shift |
| 6 | [Motion / video frame] | … | Cinematic ratio + style |

### Recommended starting pair (A/B)
Primary: #[N] — [one-line why]
Variant: #[N] — [one-line why it's a meaningfully different angle, not just different words]
```

Save batch output to `./creative-prompts/[brand-slug]-[brief-slug].md` when the user confirms.

---

## Improve mode — delivery format

```
## Prompt improvement — [brief descriptor]

### Original prompt
[paste]

### Diagnosis
| Issue | Severity | What it causes |
|---|---|---|
| [e.g. Generic subject] | High | Midjourney defaults to stock-photo archetype |
| [e.g. No palette anchor] | Medium | Brand colors absent from output |
| [e.g. No exclusions] | Medium | Watermarks, smiling-face artifacts common |

### Improved prompt (paste-ready)
[Full rewritten prompt]

### What changed
- [Specific fix 1 — one line]
- [Specific fix 2 — one line]
```

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No prompt token before the active brand's palette, style, and bans are loaded. Visual direction that contradicts brand.md gets flagged, not silently honored.
- **RECIPE is the scaffold, not a checklist.** Every section must be present but need not be equal weight; the subject and style anchor the prompt.
- **Tool syntax is not optional.** Midjourney, DALL-E 3, and SD have genuinely different grammars. Delivering SD syntax to DALL-E produces bad outputs.
- **No invented proof in visual briefs.** Don't embed claims in the brief ("showing 10× revenue increase") that have no factual basis; the image becomes a visual claim.
- **Exclusion list = brand guardrail.** Always build it; never omit the brand's banned aesthetics.
- **Specificity over poetry.** "Warm, editorial, morning-light portrait shot on 85mm with shallow depth, linen texture background" outperforms "beautiful aesthetic photo every time."

## What Not to Do

- Don't write prompts before `brand-brain` returns the active brand palette and bans.
- Don't use placeholder colors ("a blue similar to the brand") — pull the actual hex and translate to a prompt-language color name.
- Don't mix tool syntax (e.g. `--ar` flags inside a DALL-E prompt, or paragraph prose in an SD tag list).
- Don't embed unverified claims or social proof language in the visual brief.
- Don't produce near-identical batch variants by just swapping adjectives — vary the composition, crop, or style register.
- Don't reimplement brand scanning/storage — call `brand-brain`.

## Quality Checklist (self-review before presenting)

- [ ] `brand-brain` called; palette hex and banned aesthetics loaded?
- [ ] All six RECIPE sections present and non-empty?
- [ ] Prompt formatted correctly for the stated target tool (syntax, parameter position, negative-prompt field)?
- [ ] Palette anchor present (real hex → color-name translation)?
- [ ] Exclusion list includes brand's banned aesthetics + generic no-gos?
- [ ] Single: one recommended + 1–2 meaningfully different alternates?
- [ ] Batch: ≥4 distinct angles (not synonyms), a recommended A/B pair?
- [ ] Improve: diagnosis table + before→after + named fixes?
- [ ] No invented claims embedded; no brand-contradicting style direction silently honored?
