---
name: ai-image-generator-on-brand-assets
description: >
  Turns a brief into a finished, correctly-sized on-brand image — social post, blog hero, display ad,
  OG/featured thumbnail — delivered as a usable, production-ready asset. Pulls brand visual identity
  (palette, typography, motifs, do-not-use rules) from brand-brain, maps the brief to the right
  platform spec, writes a precision prompt engineered for the target generator (DALL-E 3, Midjourney,
  Firefly, Ideogram, Flux), and returns the prompt + a render recipe with size, format, and post-
  processing instructions. On request produces multiple angle variants and a QA checklist. Composes
  cta-variant-generator for overlay text and ad-copy-variant-generator for ad creative briefs — it
  does not rewrite those skills' outputs. Use when the user says "make an image," "create a visual,"
  "generate a social graphic," "I need a hero image," "create a blog thumbnail," "on-brand image for
  my ad," "OG image for this post," "image variants for A/B," or hands over a brief and asks for a
  visual asset.
---

# AI Image Generator (On-Brand Assets)

Brief + brand → a finished, correctly-sized on-brand image ready for social, blog, ad, or OG.

This skill does not wing it. It loads the brand's visual identity from `brand-brain`, maps the brief to the precise platform spec, and engineers a precision prompt the generator can execute — with color hex codes, font directives, composition rules, negative-space instructions, and file-spec metadata baked in. The output is not inspiration — it is a render recipe a marketer can copy-paste and a designer can hand off.

Visual identity is non-negotiable: wrong colors and wrong fonts ship brand debt faster than any copy error.

---

## Skills this calls

- **`brand-brain`** (required, always first) — resolves visual identity: brand palette (hex values), approved fonts, motif/illustration style, on-brand photography direction, and any visual do-not-use rules. Also supplies voice, ICP, and positioning for overlay-copy decisions.
- **`cta-variant-generator`** (optional) — when the asset needs overlay CTA text, call this to get on-brand, on-awareness button copy rather than guessing it inline.
- **`ad-copy-variant-generator`** (optional) — when the image is ad creative, call this for headline/description variants; this skill handles the visual side of the same brief.

---

## How a run works

```
Step 0  Load the brand visual identity  ──► call brand-brain
Step 1  Classify the asset + resolve the spec
Step 2  Write the precision image prompt  (the CRISP framework)
Step 3  Produce the render recipe (format, size, post-processing)
Step 4  Optional: variant set + QA checklist
```

### Step 0 — Brand context (always first)

**Invoke `brand-brain`** (Skill tool, `skill: brand-brain`) before writing a single prompt. Extract from the returned digest:

- **Palette** — primary hex(es), accent hex(es), background/neutral hex(es).
- **Typography** — approved typeface(s); if AI generator can't match exactly, closest approved substitute.
- **Visual style** — photography vs. illustration vs. vector; realistic/editorial vs. flat/graphic; any motifs (icons, patterns, shapes).
- **Do-not-use** — competitor colors, clip-art, watermarked stock, specific visual clichés the brand has banned.
- **ICP** + tone — who sees this and what emotional register fits.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly. If neither exists, ask the user: brand colors (hex) · font(s) · visual style (photo/illustration/flat) · one do-not-use rule. Prefer the call.

Do not write a single image prompt until visual identity is confirmed.

---

## Step 1 — Asset classification + spec resolution

Map the brief to a canonical spec. If the user names a platform, apply the current spec; if ambiguous, ask.

| Asset type | Primary spec | Format | Notes |
|---|---|---|---|
| Social — LinkedIn/Facebook feed | 1200 × 628 px (landscape) or 1080 × 1080 (square) | PNG/JPEG | Text ≤ 20% of area for ads |
| Social — Instagram feed | 1080 × 1080 (square) or 1080 × 1350 (portrait 4:5) | PNG/JPEG | |
| Social — Instagram / LinkedIn Stories | 1080 × 1920 px | PNG/JPEG | Safe zone: 250 px top + bottom |
| Blog hero / OG featured image | 1200 × 630 px | PNG/JPEG | OG standard; ≤ 8 MB |
| Display ad — leaderboard | 728 × 90 px | PNG/JPEG/GIF | Minimal text |
| Display ad — medium rectangle | 300 × 250 px | PNG/JPEG | |
| Display ad — half page | 300 × 600 px | PNG/JPEG | |
| Thumbnail (YouTube/webinar) | 1280 × 720 px | JPEG | High contrast text safe |
| Email header | 600 × 200 px | PNG/JPEG | 72 dpi, < 200 KB |

**Overlay-text zone:** always flag where text will sit and leave appropriate negative space in the prompt. An image with a CTA block embedded but no breathing room is unusable.

---

## Step 2 — The CRISP Prompt Framework

Every prompt this skill writes follows CRISP — the five elements that eliminate ambiguity from AI image generators:

- **C — Composition.** Framing, perspective, subject placement (rule of thirds, center-weight, bleed edge), negative space for text overlay, foreground/background relationship.
- **R — Rendering style.** Photorealistic / editorial / flat-vector / isometric / hand-drawn illustration / cinematic — match the brand's visual style sheet. Name the aesthetic precisely ("editorial product photography," not "nice photo").
- **I — Identity lock.** Inject brand colors as hex values directly. Name the closest real font if the generator supports it; else describe weight + feel ("bold geometric sans-serif, similar to Futura"). Include brand motifs if applicable.
- **S — Subject + Scene.** Who/what is in the frame, what they're doing, environment, props, lighting (soft natural light / dramatic side-lit / studio white / low-key dark).
- **P — Platform parameters.** State the exact resolution / aspect ratio in the prompt. Add negative prompts for anything the brand bans. End with technical directives: format (PNG), quality preset, seed if reproducibility matters.

**Prompt anatomy (CRISP order):**

```
[Composition], [Rendering style], [Subject + Scene], [Identity lock: colors hex, font feel],
[Platform parameters: aspect ratio, resolution], --no [negative terms]
```

Write the full prompt as a code block the user can copy directly into the generator. Label which generator(s) it is tuned for.

**Generator-specific notes:**
- **DALL-E 3 (ChatGPT/API):** prefers natural-language prose; describe the hex as "the exact shade of deep navy #191A35"; works well with negative prompts in parenthetical ("avoid any text or watermarks").
- **Midjourney v6+:** responds to `--ar`, `--style`, `--no`; inject hex colors after `--`; use `--style raw` for less artistic drift.
- **Adobe Firefly:** brand-safe (trained on licensed content); accepts specific font references; strong at product compositions.
- **Ideogram:** best-in-class for text-on-image rendering; use when the asset needs legible overlay text generated in-image.
- **Flux (Black Forest):** high prompt adherence; supports long structured prompts; good for photorealistic editorial.

---

## Step 3 — Render recipe

Deliver alongside the prompt:

```
## Render recipe — [Asset type · Platform · Brand slug]

Generator:       [DALL-E 3 / Midjourney v6 / Firefly / Ideogram / Flux]
Prompt:          [full prompt in code block]
Aspect ratio:    [e.g. 1200 × 628 px / 16:9]
Format:          PNG / JPEG
Quality:         [high / max]
Post-processing: [e.g. "Export at 72 dpi, compress to < 500 KB; add CTA overlay in [brand font] at [hex] via Canva/Figma layer"]
Overlay-text zone: [e.g. "Bottom-right third — leave 400 × 150 px clear"]
Negative prompts: [list]
Save path:       ./assets/images/[brand-slug]-[asset-type]-[date].png
```

Save the recipe file to `./assets/images/[brand-slug]-[asset-type]-[date]-recipe.md`. The image file itself is saved wherever the user's generator exports it; this skill does not call the generator API directly unless an integration is available.

---

## Step 4 — Variant set (on request)

When the user asks for variants, "options," "A/B," or a count ≥ 2:

1. Hold brand identity constant across all variants.
2. Vary one CRISP dimension per variant (composition angle, rendering style, subject/scene, lighting mood) — not random drift.
3. Label each variant with what changed and the hypothesis it tests (e.g., "Variant B — editorial photography vs. flat illustration: tests whether photorealistic converts better for this ICP").
4. Recommend a primary + a deliberate Variant B for A/B, with a one-line hypothesis.
5. If the user wants CTA overlay text on the variants, call `cta-variant-generator` rather than guessing.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No prompt before visual identity is loaded. Wrong hex = off-brand asset at scale.
- **CRISP every time.** A vague prompt is a coin flip. A CRISP prompt is a spec.
- **Platform spec is not optional.** Wrong dimensions waste the generation. Always state the exact output resolution in the prompt.
- **Negative space is a design decision.** Every asset that will carry text overlay must have it engineered into the composition, not cropped in later.
- **Text-on-image → Ideogram.** Any other generator will produce illegible or hallucinated text; flag this and route accordingly.
- **Real brand colors as hex.** Never approximate ("dark blue") — inject the actual hex so the generator has no interpretation room.
- **No invented proof or claims in overlay text.** If the user wants copy on the image, call `cta-variant-generator` or use only confirmed proof from the brand digest.

---

## What Not to Do

- Don't write a prompt before `brand-brain` returns visual identity. One wrong hex at scale = a brand audit.
- Don't produce vague, high-level prompts ("a nice social media image with brand colors") — that is not senior output.
- Don't reimplement brand scanning, voice codification, or proof sourcing — call the appropriate skills.
- Don't exceed the platform's text-safe percentage for ad images (Meta enforces ≤ 20% text area for full delivery).
- Don't call a text-capable generator (DALL-E, Midjourney, Flux) for assets requiring legible overlay text — route to Ideogram or advise Canva/Figma layer.
- Don't hardcode pixel specs from memory for ad platforms — check current platform specs if the brief is for paid placement (specs change quarterly).
- Don't save anything to the skill folder; all output goes to `./assets/images/` relative to the user's CWD.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and visual identity confirmed (hex palette, font, style, do-not-use) before any prompt?
- Asset type identified and correct platform spec applied (dimensions, format, file-size limit)?
- Prompt follows full CRISP order: Composition · Rendering · Identity lock · Subject/Scene · Platform parameters?
- Brand hex values injected as explicit hex codes, not color names?
- Negative-space / overlay-text zone specified in the prompt?
- Generator correctly selected for the asset type (especially: Ideogram for text-on-image)?
- Render recipe written with save path, post-processing note, and overlay-text zone?
- Variant set (if requested): each variant varies one CRISP dimension, labeled with hypothesis; primary + Variant B recommended?
- No invented claims in any overlay-text suggestion; `cta-variant-generator` called for CTA copy on creative?
