---
name: product-demo-gif-mockup-builder
description: >
  Takes a live URL + click path (or a design file / screenshot sequence) and produces a looping
  product demo GIF under 5 MB and/or a device-framed mockup ready for landing pages, social, and
  email. The render recipe is deterministic: it resolves brand colors and asset constraints from
  brand-brain, builds a frame-by-frame storyboard against the AIDA-motion framework, generates a
  precise shell-level render command block (using gifski + ffmpeg or Puppeteer + gifski), then
  validates the output against platform constraints (LinkedIn <5 MB / <400 frames, OG images,
  email-safe still fallback). Outputs a render-ready recipe file, not hand-wavy instructions.
  Use when the user says "make a product demo GIF," "record my feature as a GIF," "animated
  mockup for the landing page," "device-frame the screenshot," "GIF for LinkedIn/email,"
  "show the click-through flow," "product walkthrough animation," or hands over a URL and says
  "make this visual."
---

# Product Demo GIF & Mockup Builder

A looping product demo GIF that fits under the platform size envelope and actually shows the
right thing — the moment a feature clicks — converts faster than a static screenshot and
cheaper than a full video. This skill produces a deterministic render recipe: brand-governed,
frame-budget-constrained, and command-level complete so you can run it in a terminal or hand
it to any developer without guesswork.

Framework: **AIDA-Motion** — every GIF has an Attention frame (problem or hook), Interest frames
(feature build-up), a Desire moment (the aha payoff), and an Action cue (the branded still that
loops back to the CTA). Four beats. One GIF.

---

## Skills this calls

- **`brand-brain`** (required) — loads active brand's visual identity (colors, font, UI
  accent palette) and voice; drives overlay text, badge copy, and device-frame color.
- **`cta-variant-generator`** (optional) — if the GIF includes an end-card or caption CTA,
  call this for the CTA text rather than writing it ad hoc.
- **`landing-page-heuristic-live-cro-auditor`** (optional) — when the GIF will sit on an
  existing page, check the page first so the GIF reinforces the highest-leverage conversion fix.

---

## How a run works

```
Step 0  Load the brand          ──► call brand-brain; extract visual identity digest
Step 1  Clarify scope           ──► URL + click path, OR file/screenshot sequence + platform
Step 2  Build the storyboard    ──► AIDA-Motion beat map, frame budget, timing
Step 3  Write the render recipe ──► shell commands (ffmpeg / gifski / Puppeteer)
Step 4  Validate constraints    ──► size envelope, frame cap, fallback still
Step 5  Save & confirm          ──► ./assets/[slug]-demo-recipe.md + ./assets/[slug]-still.png spec
```

---

## Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Extract from the returned digest:
- **accent color** — the primary brand color used in UI overlays and device-frame border
- **font stack** — for caption / badge overlays
- **banned visual patterns** — any flagged in the brand voice section (e.g. competitor colors)
- **proof points** — for end-card badge microcopy (real only, else `[verify]`)

**Fallback:** read `~/.brandbrain/brands/.active` + `brand.md` directly; if absent, ask the
user for accent hex, font name, and one proof point before continuing.

---

## Step 1 — Clarify scope

Resolve in order from what the user supplies; ask only for genuinely missing pieces:

| Input mode | What to collect |
|---|---|
| **URL + click path** | URL, ordered list of interactions (click / hover / scroll / fill), viewport size (default 1280×800), target platform(s) |
| **Screenshot sequence** | Ordered file paths or pasted images, frame duration per image, target platform(s) |
| **Design file** | Figma / Sketch artboard URLs or local file paths, which frames are "screens," target platform(s) |

**Target platform defaults** (determine frame budget and output spec):

| Platform | Max size | Max frames | Preferred px | Notes |
|---|---|---|---|---|
| LinkedIn post | 5 MB | ~400 | 1080×1350 (portrait) | Freezes to frame 1 if either limit exceeded |
| Landing page hero | 3 MB | no hard cap | 1200×675 | Use WebP if tooling allows |
| Email | N/A — GIF risky | — | — | Always produce a still fallback PNG |
| OG / social preview | 1 MB | ≤100 | 1200×630 | Keep it snappy |
| Blog / docs embed | 2 MB | no hard cap | 800×500 | |

---

## Step 2 — AIDA-Motion storyboard

Map the click path onto four beats. Be specific: each beat names the exact UI state visible,
the action happening, and the caption overlay (if any).

```
Beat A — Attention (frames 1–N_a, ~0.5–1 s)
  UI state: [e.g., empty dashboard, competitor's clunky UI, the pain-state screen]
  Action: none or slow pan
  Caption overlay: [the problem, in ≤6 words, brand font, accent color]

Beat I — Interest (frames N_a+1 – N_i, ~1–2 s)
  UI state: [feature revealed, step 1–2 of the flow]
  Actions: [click / keystroke / dropdown reveals]
  Caption: optional label ("Set your trigger →")

Beat D — Desire (frames N_i+1 – N_d, ~0.5–1 s)
  UI state: [the aha moment — result loaded, metric changed, notification fired]
  Actions: none — let it breathe
  Caption: [the payoff, 1 proof point if available, brand accent]

Beat A2 — Action cue (final frames, ~1 s, loops to frame 1)
  UI state: [product logo or branded still, no UI noise]
  Caption: [CTA from cta-variant-generator if called, else product tagline]
  Loop behavior: freeze on last frame for 1.5 s before looping
```

**Frame-budget math:** `total_frames = fps × total_duration_s`. Default 12 fps.
For LinkedIn: keep total ≤ 380 frames (20-frame safety margin). If the click path exceeds
budget, cut Beat I (not the Desire moment — that is the conversion driver).

---

## Step 3 — Render recipe (runnable output)

Produce a complete, copy-paste-ready shell block. Choose the toolchain based on what the user
has or can install:

### Path A — URL input (Puppeteer + gifski)

```bash
# 1. Record frames via Puppeteer (save as PNG sequence to ./frames/)
# Adjust DELAY_MS between interactions to match your timing above.
node record-demo.js \
  --url "https://example.com/feature" \
  --interactions '[
    {"type":"click","selector":"#open-modal","delay":800},
    {"type":"fill","selector":"#input","value":"test@example.com","delay":500},
    {"type":"click","selector":"#submit","delay":1200}
  ]' \
  --viewport 1280x800 \
  --fps 12 \
  --out ./frames/

# 2. Stitch frames → GIF with gifski (high quality, palette-optimised)
gifski --fps 12 --quality 85 --width 1080 \
  ./frames/*.png -o ./assets/[slug]-demo.gif

# 3. Verify constraints
python3 -c "
import os, sys
f='./assets/[slug]-demo.gif'
size_mb=os.path.getsize(f)/1024/1024
print(f'Size: {size_mb:.2f} MB  ({'OK' if size_mb < 5 else 'EXCEEDS LIMIT'})')
"
```

> Puppeteer scaffold `record-demo.js` — see reference block in the recipe file output below.

### Path B — Screenshot sequence (ffmpeg + gifski)

```bash
# 1. Build a palette-optimised GIF from an ordered PNG sequence
#    Set -framerate to your target fps; adjust -vf scale to target width.
ffmpeg -framerate 12 -i ./screens/frame-%03d.png \
  -vf "scale=1080:-1:flags=lanczos,split[s0][s1];[s0]palettegen=max_colors=256[p];[s1][p]paletteuse" \
  -loop 0 ./assets/[slug]-draft.gif

# 2. Re-encode with gifski for smaller file at same quality
gifski --fps 12 --quality 90 ./assets/[slug]-draft.gif -o ./assets/[slug]-demo.gif

# 3. Still fallback (frame 1 exported as PNG for email)
ffmpeg -i ./assets/[slug]-demo.gif -vframes 1 ./assets/[slug]-still.png
```

### Path C — Device-frame mockup (no animation)

When the user wants a static device-framed screenshot (for OG, hero, or email):

```bash
# Composite the screenshot onto a device frame using ImageMagick
# Replace DEVICE_FRAME.png with the appropriate frame asset (browser, iPhone, etc.)
convert DEVICE_FRAME.png \
  \( ./screens/screenshot.png -resize 1040x650! \) \
  -geometry +120+80 -composite \
  ./assets/[slug]-mockup.png

# Apply brand accent drop-shadow
convert ./assets/[slug]-mockup.png \
  \( +clone -background "[BRAND_ACCENT_HEX]" -shadow 60x20+0+8 \) \
  +swap -background none -layers merge \
  ./assets/[slug]-mockup-shadow.png
```

---

## Step 4 — Constraint validation

Before handing off, validate every output against the target platform row in the table above:

- [ ] File size within platform envelope (gifski `--quality` tunable; reduce fps from 12→8 first, then cut Beat I frames, never the Desire beat)
- [ ] Frame count within cap (LinkedIn ≤400 — check with `identify -format "%n\n" demo.gif | tail -1` if ImageMagick installed)
- [ ] Still fallback PNG produced for any email placement
- [ ] First frame works as a standalone static image (attention-grabbing; do not start on a blank state)
- [ ] Brand accent color used in overlays; no banned visual patterns; proof microcopy real or `[verify]`

---

## Step 5 — Save artifacts

Write to `./assets/` (project-relative, never inside the skill folder):

| File | Contents |
|---|---|
| `[slug]-demo-recipe.md` | Full storyboard + annotated shell commands + validation checklist |
| `[slug]-demo.gif` | The rendered GIF (if run locally) |
| `[slug]-still.png` | First-frame still for email fallback |
| `[slug]-mockup.png` | Device-framed static (if Path C) |

The recipe file is the primary deliverable — it lets any developer reproduce the GIF from
scratch without re-running the skill.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** Visual identity (color, font, proof) comes from the brand digest —
  never hardcode hex values or invent proof microcopy.
- **Show the Desire moment.** Every GIF must reach the payoff. If the click path is too long
  to fit the frame budget, trim setup beats — never trim the aha moment.
- **Deterministic recipe output.** The skill delivers runnable shell commands, not vague
  "use gifski" instructions. A junior should be able to copy-paste and produce the asset.
- **Platform constraints are hard limits.** LinkedIn's 5 MB / 400-frame envelope is the
  default target; exceed it and the file silently freezes on frame 1. Validate before shipping.
- **First frame sells.** The GIF may not autoplay. Frame 1 must communicate the product value
  on its own — no blank canvases, no loading spinners.
- **Still fallback for email.** Email clients do not reliably animate GIFs; always produce
  a still PNG and note the alt-text copy alongside it.

## What Not to Do

- Don't produce a render recipe before `brand-brain` returns the visual identity digest.
- Don't recommend re-recording the whole click path to save a few frames — cut Beat I
  (the build-up) before cutting the Desire beat.
- Don't hardcode brand hex values or proof points — pull them from the brand digest every run.
- Don't skip the constraint validation step; a 6 MB GIF is a broken asset on LinkedIn.
- Don't treat Path C (static mockup) as a fallback for a failed GIF — surface the size issue
  explicitly and let the user decide.
- Don't write files inside the skill folder; all output goes to `./assets/` in the user's project.

## Quality Checklist (self-review before presenting)

- [ ] `brand-brain` called; visual identity digest (accent color, font, proof) loaded?
- [ ] AIDA-Motion beat map complete — Attention / Interest / Desire / Action cue defined?
- [ ] Frame budget calculated and within platform envelope (default LinkedIn ≤380 frames)?
- [ ] Render recipe is copy-paste runnable (correct toolchain path chosen for input mode)?
- [ ] Constraint validation block included; still fallback PNG specified for email placements?
- [ ] Artifacts saved to `./assets/[slug]-demo-recipe.md` with annotated storyboard?
- [ ] First frame works as a standalone static; no blank/loading state as opening?
- [ ] All proof microcopy real or `[verify]`; brand banned visual patterns avoided?
