---
name: social-proof-screenshot-styler
description: >
  Transforms raw proof — a plain screenshot, raw review text, a tweet/DM grab, an approved
  testimonial — into a social-ready branded card. Produces a render-ready annotation spec
  (border style, brand color fills, watermark placement, blur zones, overlay copy) PLUS an
  optional 30–60s spoken-script for a talking-head or voiceover video that amplifies the same
  proof point. Calls brand-brain for colors, font stack, banned words, and voice; calls
  proof-vault for the canonical proof library so you never surface an unverified or
  superseded number; calls screenshot-annotator-bug-reporter for the technical annotation
  layer. The framework is Cialdini's Social Proof ladder: the output maps each asset to the
  strongest available proof tier (celebrity/authority → user mass → wisdom of friends →
  certification) and styles it accordingly. Outputs a card spec the user can execute in
  Canva, Figma, or any image editor, plus (on request) a spoken script. Use when the user
  says "style this screenshot," "make this review shareable," "brand this testimonial,"
  "turn this DM into a social card," "social proof graphic," "proof card," "screenshot card,"
  "testimonial image," "quote graphic," "review screenshot," or hands over raw proof and asks
  how to make it look good and on-brand.
---

# Social Proof & Screenshot Styler

Raw proof rots on a clipboard. This skill turns it into a branded, trust-signaling asset — a render-ready card spec with every styling decision made and a video spoken-script when you need to go further. It does not redesign your brand; it reads the brand and applies it. It does not invent proof; it checks the proof vault and flags anything unverified.

The governing framework is **Cialdini's Social Proof ladder**. Where a piece of proof sits on that ladder determines how the card is styled, what context copy it carries, and how prominently the proof source is amplified.

---

## Skills this calls

- **`brand-brain`** (required) — loads brand colors, font stack, voice, banned words, and watermark/logo assets. This skill does not implement brand resolution itself.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand colors (hex), font name, logo path, and any banned words before proceeding.
- **`proof-vault`** (required when available) — retrieves the canonical proof library and flags any numbers the user supplied that conflict with or are superseded by the vault. Never style a card with an unverified or stale stat.
  Fallback if proof-vault is absent: use only proof the user explicitly supplies and mark any numeric claims `[verify]`.
- **`screenshot-annotator-bug-reporter`** (optional, when installed) — handles the pixel-level annotation layer (blur zones, redaction boxes, callout arrows). Synthesize annotation instructions inline when absent.
- **`canva-figma-workflow-accelerator`** (optional, for downstream execution) — if the user wants a Canva Bulk Create CSV or a Figma component spec for batch production, route there after this skill produces the card spec.
- **`brand-consistency-auditor`** (optional, for review pass) — invoke after producing a batch of card specs to flag any off-brand color or tone violations before the assets go live.

---

## How a run works

```
Step 0  Load brand + proof  ──► call brand-brain, then proof-vault
Step 1  Classify the proof  ──► place it on Cialdini's ladder
Step 2  Choose the card mode ──► Quick card (default) | Batch spec | + Video script
Step 3  Build the card spec  ──► styling decisions, annotation layer, copy
Step 4  Optional video script ──► 30–60s spoken framing
Step 5  Self-review + present
```

### Step 0 — Load brand and proof (always first)

**Invoke `brand-brain`** (Skill tool, `skill: brand-brain`). Use the returned digest for brand colors, font stack, voice adjectives, banned words, and any available logo/watermark path.

**Fallback if brand-brain is absent or returns no brand:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for brand colors (hex), font name, logo path, and any banned words before proceeding.

Then invoke `proof-vault` if installed. Check every numeric claim in the user's proof against the vault. If a number conflicts or is superseded, flag it and use the vault version. If the vault is absent, accept user-supplied proof and mark any unverified numbers `[verify]`.

Do not produce any card spec until both calls return (or fallbacks are applied).

---

## Cialdini's Social Proof ladder (the classification framework)

Map every piece of incoming proof to its strongest honest tier — never inflate upward.

| Tier | Signal | Styling treatment |
|---|---|---|
| **T1 Celebrity / Authority** | Named expert, recognized brand logo, press mention, verified badge | Full-bleed logo or headshot, high-contrast brand border, masthead-style attribution |
| **T2 Certification / Award** | G2 badge, award seal, compliance cert, platform badge | Badge prominent (no crop), muted background, trust-copy beneath |
| **T3 User mass / volume** | Review count, "X customers," star rating aggregate | Large typographic number, secondary brand color fill, source attribution |
| **T4 Named user / case study** | First-name + job title + outcome, identifiable DM | Quote pull, avatar or initials circle, company logo if cleared |
| **T5 Anonymous / unverified** | Screenshot with no name, paraphrased feedback, raw DM | Minimal styling, add `[name on file]` or `[shared with permission]` attribution line; never fabricate identity |

**Inflate nothing.** If the user's proof is T5, style it as T5 — do not strip attribution lines or imply identity that doesn't exist in the source.

---

## Card spec format (Quick mode, default)

One proof item → one render-ready card spec. Inline output unless the user asks to save.

```
## Card spec — [brief proof description]

Brand: [slug, from brand-brain]
Proof tier: T[n] — [tier name]
Canvas: [recommended dimensions, e.g. 1080×1080 / 1080×1350 / 1200×628]

### Background
[color hex + fill style: solid / gradient / branded texture suggestion]

### Border / frame
[px weight · color hex · radius · shadow spec — or "none"]

### Proof content zone
- Quote / stat text: "[exact text]" — [font: weight size, brand font stack from brand-brain]
- Max characters to display: [n] — truncation rule if source is longer
- Attribution line: "[Name, Title, Company]" — [font: weight size]
- Source badge / logo: [placement · max px · clear space]

### Watermark / brand lock-up
- Logo: [placement — e.g. bottom-right 24px margin · max 120px wide]
- Color: [hex — should not compete with proof content]

### Annotation layer (blur / redaction / callout)
[If screenshot: blur zones (PII, unrelated UI regions) · callout arrows if directing attention]
[If text-only proof: "no annotation needed"]

### Overlay copy (optional trust-builder)
[e.g. "Verified G2 review" · "4.9 ★ on Capterra" · "5,000+ teams" — use only vault-confirmed numbers]

### Accessibility
- Alt text: "[draft alt text for the final image]"
- Contrast check: confirm overlay text meets WCAG AA against the background hex above

### Execution path
[Canva: step-by-step from blank · Figma: component suggestion · or "export spec only"]
```

---

## Batch mode (on request)

Triggered by "multiple," "batch," "bulk," or a list of ≥3 proof items.

1. Classify all items on the ladder first; group by tier.
2. Produce a **shared base spec** (background, border, watermark, font stack) common to all cards.
3. Produce a **per-card delta table** — only the fields that differ per item (quote text, attribution, tier-specific badge, canvas size).
4. Flag any item where proof cannot be verified — do not omit it, but annotate it.
5. If `canva-figma-workflow-accelerator` is available, offer to produce a Bulk Create CSV after the specs are confirmed.

Save batch output to `./social-proof/[brand-slug]-card-specs.md` when asked.

---

## Video spoken-script (on request)

Triggered by "video," "script," "voiceover," "talking head," or "reel."

30–60s. Structure:

```
[0–5s]   Hook — name the proof outcome in plain language ("A SaaS team cut churn by…")
[5–20s]  The proof itself — read or paraphrase the quote/stat; add one sentence of context
[20–45s] Why it matters for the viewer — connect to the ICP's specific pain (use brand-brain ICP)
[45–60s] CTA — one action, brand-voice compliant, no banned words
```

Deliver as labeled sections with estimated timestamps. Keep language conversational — written to be heard, not read. Use only vault-confirmed numbers; mark others `[verify]`.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No card spec before brand-brain returns. Its colors, fonts, and banned words are hard overrides.
- **Proof vault gates the numbers.** Any numeric claim not in the vault or not user-supplied gets `[verify]` — never invent a stat to make a card look more compelling.
- **Ladder honesty.** Place proof at its actual tier. Never style a T5 piece as T1 by removing attribution or implying authority that isn't there.
- **Attribution is not optional.** Every card carries a source attribution line. Anonymous proof carries an explicit permission note.
- **Accessibility is output, not afterthought.** Every spec includes a draft alt-text string and a contrast check note.
- **Spec is the deliverable.** This skill produces render-ready instructions, not the rendered image. The spec must be complete enough that a non-designer can execute it.

## What Not to Do

- Don't produce a card spec before brand-brain returns (or the fallback inputs are collected).
- Don't surface a number that conflicts with the proof vault — use the vault version and flag the discrepancy.
- Don't inflate the proof tier — a DM from an anonymous user is T5, not T1 celebrity endorsement.
- Don't strip the attribution line to make the card look cleaner — attribution is a compliance and trust requirement.
- Don't invent identity: never fabricate a name, company, or photo for an anonymous proof item.
- Don't produce a video script that includes unverified stats — mark them `[verify]` or omit them.
- Don't reimplement brand scanning or proof curation — call `brand-brain` and `proof-vault`.

## Quality Checklist (self-review before presenting)

- `brand-brain` called; active brand loaded; colors, font, banned words applied?
- `proof-vault` checked; all numeric claims vault-confirmed or marked `[verify]`?
- Proof classified at its honest Cialdini tier — no inflation?
- Card spec complete: background, border, content zone, attribution, watermark, annotation layer, alt text, contrast note, execution path?
- Attribution line present on every card; anonymous proof carries a `[shared with permission]` note?
- Batch mode: shared base spec + per-card delta table produced?
- Video script (if requested): hook → proof → ICP context → CTA; branded voice; no unverified stats?
- No banned words; no invented proof; no fabricated identity anywhere in the output?
