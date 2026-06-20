# Brand Brain — `brand.md` template

The canonical brand-context file every copy skill reads. One per brand at `<data-root>/brands/<slug>/brand.md`. Copy this, fill what's known, leave `[unknown]` where not — skills degrade gracefully on missing fields and sharpen as it fills in. Fields marked **(core)** are the minimum for good copy work; Bootstrap shouldn't finish without them.

```markdown
---
slug: <kebab-case-brand-id>
name: <Brand display name>
status: ACTIVE            # DRAFT until user-confirmed, then ACTIVE
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
sources:                  # where derived from, for trust + refresh
  - <e.g. ~/.thoth/personas/<slug>/persona.md>
  - <e.g. auto-memory: icp_q4_2025.md>
  - <e.g. user interview YYYY-MM-DD>
confidence: <high | medium | low>
---

# <Brand> — Brand Brain

## What it is  (core)
One or two plain sentences: what it is and the single job it does.

## Positioning / core frame  (core)
The competitive frame every piece of copy reinforces. One short paragraph + the positioning line if there is one.

## ICP(s)  (core)
Per ideal customer: **Who** (titles, company shape, scale) · **Pains / triggers** · **Awareness tendency** (ladder position → default CTA commitment) · **Metrics they own**.

## Value proposition & differentiators  (core)
Headline value prop (one line) + the 3–7 REAL, defensible differentiators. Copy skills must never invent these.

## Offer & pricing essentials  (core for CTAs)
Free plan? (limits) · Trial? (length, card?) · Demo? · Guarantee / "cancel anytime"? · **Primary conversion action(s) + destination URL(s)** · price points only if copy may reference them.

## Proof assets  (core for CTAs)
Real usable numbers/names: stats, ratings, logos, awards. Mark unconfirmed `[verify]`.

## Voice & tone  (core)
3–5 adjectives · person/POV · sentence style · a one-line "sounds like…".

## Banned words / phrases  (core)
The hard no-list (hype words, clichés, punctuation tics). Copy skills treat this as an override.

## Preferred lexicon
Verbs to favor · domain/metric terms to use precisely · framings/possessives that put the reader in the scene.

## Common CTAs & destinations
Known buttons + where they point (signup, trial, demo, pricing, contact) — the message-match anchors.

## Visual identity  (optional, reference only)
Colors, font, tagline — for commissioned assets, not body copy.

## Notes / do-not
Idiosyncrasies a skill must know: claims rules, legal lines, competitor-linking policy, segment nuances.
```

## Minimum viable brand
If the scan finds nothing, the interview must still capture: what it is + ICP + awareness tendency; the core differentiator(s); offer mechanics gating CTAs + primary destination; 3 voice adjectives + banned words; one real proof point (or "none yet"). That's a usable `brand.md`.
