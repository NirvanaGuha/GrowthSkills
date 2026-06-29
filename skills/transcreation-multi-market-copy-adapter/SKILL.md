---
name: transcreation-multi-market-copy-adapter
description: >
  Takes a single English (source) marketing asset — ad, email, landing-page section, push
  notification, social post, or CTA — and produces culturally adapted locale variants for one
  or more target markets, including RTL languages. Goes beyond word-for-word translation:
  applies a per-locale transcreation brief (intent, emotion, constraints, cultural notes) per
  locale, shifts tone where the market demands it, rewrites idioms and metaphors, substitutes
  local proof/examples where the brand allows, and provides mirrored-layout guidance for RTL
  outputs (Arabic, Hebrew). Every variant is grounded in the active brand's voice, banned
  words, and real proof — loaded from brand-brain — so nothing contradicts your brand system.
  Outputs a per-locale variant set plus a Transcreation Brief (copy intent notes) the team or
  a human translator can use for QA or further refinement. Use when the user says "localize
  this," "adapt for [country/language]," "multi-market copy," "transcreation brief,"
  "RTL version," "translate and adapt," "French/Spanish/German version," "culturally adapt,"
  or hands over English copy and asks for it in another market.
---

# Transcreation & Multi-Market Copy Adapter

Translation is a word swap. Transcreation is a brand-intent swap. This skill takes English marketing copy and rebuilds it for each target locale — same strategic job, culturally correct execution, on-brand voice — not a dictionary pass.

Every run produces: a **Transcreation Brief** (the creative passport for each locale) and the **adapted copy variants** ready to hand to a reviewer, translator, or direct to publish.

---

## Skills this calls

- **`brand-brain`** (required) — resolves and loads the active brand's voice, banned words, offer mechanics, ICP, and real proof before any locale work begins. Do not start adapting until it returns.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no `brand.md` exists, ask the user for: brand name, voice adjectives (3), banned words/phrases, offer mechanics, and any locale-specific proof or restrictions.
- **`brand-voice-codifier`** *(optional)* — if the brand has a per-locale voice extension (e.g. "our French tone is warmer than English"), call it to load that layer; synthesize inline when absent.
- **`proof-vault`** *(optional)* — pulls locale-relevant proof points (local customer logos, regional case studies); synthesize from brand-brain's proof digest when absent.
- **`consent-privacy-compliance-auditor`** *(optional)* — flag any copy claims or data references that differ in legality between source and target market (GDPR vs. CAN-SPAM, PECR, PIPL, etc.); note risks inline when absent.
- **`cta-variant-generator`** *(optional)* — if the adapted CTA needs a full battery across locale-specific angles, delegate there; write the primary locale CTA inline when absent.
- **`advertising-claims-ftc-disclosure-reviewer`** *(optional)* — if the asset includes performance claims, check claim legality applies per locale (ASA in UK, ARPP in France, etc.); flag `[verify with local counsel]` when absent.

---

## How a run works

```
Step 0  Load the brand        ──► call brand-brain; get voice + proof + banned words
Step 1  Parse the brief        ──► source copy + target locales + asset type + constraints
Step 2  Write Transcreation Briefs  ──► one per locale (intent, emotion, metaphors to change)
Step 3  Produce adapted copy   ──► locale variants, RTL guidance if needed
Step 4  Self-review             ──► brand, voice, cultural, compliance gates
Step 5  Present                 ──► brief + variants + reviewer notes
```

---

## Step 0 — Load the brand (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). It returns the active brand's digest: voice adjectives, banned words, offer mechanics and destination URLs, real proof points, positioning, ICP. Do not write a single locale word until this returns.

Obey returned voice and banned words as hard overrides. Use only returned real proof — mark anything unconfirmed `[verify]`. Note any existing locale-specific brand guidance in `brand.md` (competitor restrictions, regional pricing, local proof).

---

## Step 1 — Parse the brief

Collect from the user (ask if not provided):

| Input | Why it matters |
|---|---|
| Source copy (English) | The asset to adapt |
| Target locales | ISO 639-1 language + ISO 3166-1 country (e.g. `fr-FR`, `es-MX`, `ar-SA`) — country matters for tone, not just language |
| Asset type | Ad / email / landing page / push / social / CTA — sets character limits and register |
| Adaptation depth | **Linguistic** (word swap, brand voice only) · **Cultural** (full transcreation — idioms, metaphors, local examples) · **Market-specific** (legal, pricing, regional proof changes) |
| Hard constraints | Character/word limits per locale, elements that must stay verbatim (legal lines, product names, URLs), glossary |
| Approved local proof | Customer names, logos, regional stats the brand allows per locale |

Default adaptation depth: **Cultural** — do not default to linguistic-only without flagging it will feel translated, not written.

---

## Step 2 — Transcreation Brief (one per locale)

The Transcreation Brief is the creative passport. It documents intent so QA reviewers and human translators know *why* a phrase changed — not just *what* changed.

Write one brief per locale before writing copy. Each brief covers:

```
## Transcreation Brief — [Locale: fr-FR]

**Source intent:** [What the English is trying to make the reader feel/do — 1 sentence]
**Emotional register:** [e.g. professional confidence → in French: warmer, less hyperbolic]
**Idioms/metaphors to replace:** [Source phrase → Locale equivalent or direction]
**Cultural landmines:** [References, humour, taboos, colours, numbers, or imagery that land wrong]
**Local proof substitution:** [If brand allows: replace [US logo] with [FR logo] / keep original / omit]
**RTL layout notes:** [Required if locale is RTL — see RTL section]
**Character/length delta:** [Expected expansion/contraction vs. English, e.g. "German typically runs ~30% longer"]
**Compliance flags:** [Any locale-specific legal constraints — or [verify with local counsel]]
```

Do not skip the brief for "easy" locales. A Japanese B2B email and a Mexican eCommerce SMS both need explicit intent documentation, or QA has nothing to check against.

---

## Step 3 — Adapted copy variants

Work through our house transcreation checklist: **Intent → Emotion → Register → Form → Constraints**.

### Intent
Preserve the strategic job of each copy element (hook, offer, CTA, risk-reducer). If the source hook uses a cultural reference that doesn't land, find the locale-native equivalent that does the *same persuasive job*.

### Emotion
Map the emotional arc. High-pressure urgency that works in US direct-response often reads as aggressive in DE/JP. Warm conversational US brand voice often reads as unprofessional in formal markets (DE formal "Sie" vs. casual "du", LATAM vs. Spain register). Explicit emotional register from the brand-brain digest governs — don't infer from stereotypes.

### Register
- **Formal/informal second person:** honor the brand's stated preference per locale (e.g. German "Sie" vs. "du" is a brand decision, not a copywriter call — flag if brand-brain doesn't specify).
- **Active vs. nominal constructions:** many European languages default to nominal; match what feels natural in the locale, not a calque of English syntax.
- **Sentence rhythm:** English marketing copy is short-punchy; French and German often prefer a subordinate-clause build. Adapt rhythm, not just words.

### Form — RTL outputs (Arabic, Hebrew, Persian, Urdu)

When a target locale is RTL, provide layout guidance alongside copy:

```
RTL Guidance — [ar-SA / he-IL / fa-IR / ur-PK]
- Text direction: right-to-left; all paragraph alignments flip
- UI element mirroring: CTA button moves to right-aligned; icons/arrows reverse direction
- Number/date formats: [locale-specific — e.g. Arabic-Indic numerals vs. Western in Gulf markets]
- Font recommendation: [e.g. Noto Sans Arabic / Scheherazade New for body; verify with design team]
- Line-length note: Arabic script is typically more compact than English; expect ~20% shorter body
- Image mirroring: directional images (person looking right → should look left in RTL context) [verify with design]
- Bidirectional (BiDi) segments: product names / URLs / model numbers stay LTR within RTL flow — flag each
```

Never output RTL copy without this guidance block. A designer who skips it will mirror text without mirroring layout.

### Constraints
Apply character limits per channel and locale. German body copy in an email subject line that's fine in English often blows past 50 characters — flag it and offer a compressed variant. Never silently truncate meaning to hit a limit; surface the tradeoff.

### Output format per locale

```
---
## [Locale: fr-FR] — [Asset Type]

**Transcreation note:** [1-line summary of biggest adaptation made and why]

[HEADLINE / SUBJECT LINE]
[Body copy / email body]
[CTA button label]
[Microcopy / disclaimer]

⚠ Reviewer flags:
- [Any unresolved [verify] items, legal flags, or decisions that need native-speaker QA]
```

---

## Step 4 — Self-review gates

Before presenting, check all four gates:

**Brand gate:** Voice adjectives honored? Banned words absent (in English-origin phrases that carried through)? Only real proof used (else `[verify]`)?

**Cultural gate:** Idioms replaced (not calqued)? Emotional register correct for locale? No cultural landmines from the brief?

**Compliance gate:** Claims that differ in legality between source and target locale flagged? RTL guidance present for every RTL locale?

**Constraint gate:** Character/length limits met? Hard-verbatim elements preserved? Any truncation of meaning surfaced as a flag, not silently applied?

---

## Persistence

Save the full output (briefs + variants) to `./transcreation/[brand-slug]-[locale-pair]-[asset-slug].md` when the user asks for it or when producing 3+ locales. Inline output for 1–2 locales by default.

---

## Principles

- **Transcreation, not translation.** Same intent, native execution. A calque (word-for-word structural copy) that reads as foreign is a failed output.
- **Brief first, copy second.** The brief is the QA contract. Don't skip it because the locale feels familiar.
- **Brand-brain governs, locale adapts.** Voice adjectives and banned words travel with the brand globally — adapt register and idiom, not core identity.
- **RTL is a design problem, not just a language problem.** Never output Arabic/Hebrew/Farsi/Urdu copy without the layout guidance block.
- **Flag, don't fix, legal decisions.** Compliance is `[verify with local counsel]` — not a copywriter call.
- **Honest proof only.** If a US case study doesn't apply to the FR market, say so and use an approved local alternative or omit — don't port a name that has no brand permission.
- **Surface tradeoffs.** If a limit forces a meaning cut, say so. Don't resolve the tension silently.

---

## What not to do

- Don't start any locale work before `brand-brain` returns (or the fallback brand read completes).
- Don't default to linguistic-only depth without flagging the quality downgrade.
- Don't skip the Transcreation Brief for "obvious" locales — the brief is the QA artifact.
- Don't output RTL copy without the layout guidance block.
- Don't port US social proof to a locale without explicit brand permission.
- Don't resolve character-limit tension by silently truncating meaning — surface it.
- Don't apply locale stereotypes as register rules; use only what the brand has stated or what the brand-brain digest supports.
- Don't make legal compliance calls — flag and `[verify]`.

---

## Quality checklist

- `brand-brain` called and digest loaded before any locale copy written?
- Source intent parsed correctly — the *job* of the copy (not just the words) documented?
- Transcreation Brief written for every locale before copy produced?
- RTL guidance block present for every RTL-script locale?
- Emotional register and formal/informal second person explicitly handled (or flagged for brand decision)?
- Only real proof used; unconfirmed proof `[verify]`?
- Character/length constraints met; tradeoffs surfaced not silently applied?
- Legal/compliance differences between source and target locale flagged with `[verify with local counsel]`?
- Output saved to `./transcreation/` path when 3+ locales or explicitly requested?
