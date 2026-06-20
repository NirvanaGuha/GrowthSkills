---
name: proof-vault
description: >
  Turns scattered testimonials, stats, logos, awards, and case-study quotes into one tagged,
  permission-aware proof vault that every copy skill can draw on — normalized to a consistent shape
  (quote, attribution, use-case tag, metric, permission status) and disciplined about what's confirmed
  vs. `[verify]`. It is a brand-brain COMPONENT: callable standalone, or invoked by `brand-brain` during
  bootstrap/refresh to own the "Proof assets" section. It does NOT manage the rest of brand context — it
  calls `brand-brain` to resolve the active brand, then writes a compact proof block into brand.md and a
  full vault at brands/<slug>/proof.md. Use whenever the user says "proof," "testimonials," "social
  proof," "case study quotes," "stats and logos," "what proof do we have," "normalize our testimonials,"
  or hands over a pile of reviews/quotes/awards to organize. It catalogs and tags proof — it does not
  fabricate it, and it does not write the campaign copy that uses it.
---

# Proof & Social-Proof Vault

Give it a mess of testimonials, screenshots, stats, logos, and award badges. Get back a single tagged vault where every asset is normalized, attributed, tagged by use-case, and flagged with its permission status — so the next time any skill needs "a retention stat for the pricing page" or "an enterprise quote about onboarding," it pulls a real, cleared, on-point asset instead of inventing one.

This skill is the brand's evidence library, not its copywriter. It catalogs, normalizes, and tags proof. It does not write the headline, the case study, or the ad that uses the proof — it makes those skills fast and honest by handing them clean, permission-safe inputs.

---

## Skills this calls

- **`brand-brain`** (required, standalone mode only) — resolves and loads the active brand, returns the data root + current `brand.md`, and bootstraps a brand on first use. Proof-vault does not implement brand resolution, scanning, or storage; that lives in `brand-brain`, once. **When called BY `brand-brain`, do not call it back** — use the context it passed (no recursion).
- *(no other dependencies)* — proof-vault is a leaf. It SUGGESTS proof microcopy for sibling skills (`cta-variant-generator`, headline/landing skills) but never writes their sections.

---

## How a run works

```
Step 0  Detect the mode  ──► CALLED (context passed) | STANDALONE (resolve via brand-brain)
Step 1  Gather raw proof ──► from passed inputs and/or what the user hands over
Step 2  Normalize + tag  ──► one row per asset; permission + [verify] discipline
Step 3  Persist / return ──► STANDALONE: write proof.md + the brand.md block. CALLED: return the block.
```

### Step 0 — Detect the mode

- **CALLED BY `brand-brain`** — the request arrives with the active **slug**, the current **brand.md** content, and the **scanned raw inputs** (reviews, testimonials, ratings, prior copy). Use that context. Do your work. **RETURN** the "Proof assets" block (and, if you built one, the path to `proof.md`) for `brand-brain` to fold in. Do **not** invoke `brand-brain`.
- **STANDALONE** — a user ran you directly with no brand context. **Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`) to resolve the active brand and load its `brand.md` + data root. If the brand is new, `brand-brain` bootstraps it first. Then do your work and **persist** (Step 3).

**Fallback if `brand-brain` is not installed (standalone):** read `~/.brandbrain/brands/.active` and that brand's `brand.md` directly; if none exists, ask the user for the brand slug (or to install `brand-brain`, preferred) before persisting. Always prefer the call.

### Step 1 — Gather

Pull every candidate asset from the passed inputs and from whatever the user hands over: testimonial quotes, review-site ratings, named logos, awards/badges, press mentions, hard stats ("cut churn 22%"), user counts, case-study lines, certifications. Note the **source** of each (review URL, email, deck, screenshot) — provenance drives the permission and `[verify]` calls later. Don't discard a raw quote because it's messy; normalize it in Step 2.

### Step 2 — Normalize + tag

Run every asset through the normalization frame below. One row per asset. Trim quotes to their load-bearing sentence without changing meaning. Assign the use-case tag(s), the proof type, and the permission status. Mark every number you can't trace to a confirmed source `[verify]`.

### Step 3 — Persist / return

- **STANDALONE — persist, touching ONLY what you own:**
  1. **`brand.md` → "Proof assets" section.** Read the file, replace *only* that section block with the compact top-proof view (below), bump `updated`, append `proof-vault (YYYY-MM-DD)` to `sources`. Never touch any other section.
  2. **Companion full vault** at `<data-root>/brands/<slug>/proof.md` — the complete tagged table + the typology coverage map. (Resolve `<data-root>` from `brand-brain`; never store under the skill folder.)
  3. Confirm in one line where both were saved.
- **CALLED — return:** hand `brand-brain` the "Proof assets" block to fold in, plus the `proof.md` path if you wrote one. Do not write `brand.md` yourself in this mode (the caller owns the merge).

---

## The proof normalization frame

Every asset becomes one normalized row. This is the schema that makes a junior operator produce a usable vault:

| Field | What goes here | Rule |
|---|---|---|
| **Quote / claim** | The load-bearing line, trimmed | Verbatim meaning; never reword a customer's words to be punchier |
| **Attribution** | Name · Title · Company | Full name + title + company = strongest; redact down only when permission requires |
| **Type** | The social-proof typology (below) | One primary type per asset |
| **Use-case tag** | What it proves: onboarding, ROI, support, retention, ease, scale, security… | Tag for the job, not the feature; multi-tag is fine |
| **Metric** | The number, with unit + baseline if known | "−22% churn in 60 days" beats "reduced churn"; no number → leave blank, don't invent |
| **Strength** | strong / medium / weak | Named + titled + specific metric = strong; anonymous + vague = weak |
| **Permission** | cleared / implicit / unknown / NDA | Governs where it may appear (below) |
| **Source** | Where it came from + date | Provenance; drives `[verify]` and refresh |

**Strength, fast:** *strong* = real name + title + company + a specific, attributable metric. *Medium* = named OR specific, not both. *Weak* = anonymous and/or generic ("great product!"). Lead the vault with strong; keep weak but flag it.

### Permission status — the part juniors skip

Proof you can't use is a liability, not an asset. Tag it and respect it:

| Permission | Meaning | Where it may appear |
|---|---|---|
| **cleared** | Customer/source explicitly approved public use | Anywhere |
| **implicit** | Public review (G2, Capterra, App Store, Trustpilot) — public but attribute to the platform, don't rename the reviewer | Public copy, cite the platform |
| **unknown** | Private (email, support thread, call) with no clearance | **Do not publish** — mark "needs clearance"; usable internally to shape messaging only |
| **NDA / restricted** | Logo or quote under a usage restriction (e.g. "no logo without sign-off") | Honor the restriction exactly; note it |

When permission is `unknown`, the asset still earns a vault row — flagged "needs clearance" with a one-line ask the operator can send. Never silently promote `unknown` → usable.

## Social-proof typology

Tag each asset by type so callers can pull the *right kind* of proof for the moment — and so you can see what's missing:

| Type | What it is | Strongest for |
|---|---|---|
| **Testimonial** | A customer's words about the outcome | Trust, emotional resonance, objection-killing |
| **Quantified result** | A hard metric tied to a customer | ROI/value claims, pricing-page proof |
| **Logo / customer list** | Recognizable brands using it | Credibility, enterprise reassurance |
| **Rating / volume** | Star rating, review count, user/customer count | At-a-glance trust, "wisdom of crowds" |
| **Award / certification** | Third-party badge (G2 leader, SOC 2, awards) | Authority, security/compliance reassurance |
| **Press / mention** | Named publication coverage | Authority, "as seen in" |
| **Case study** | Full before→after narrative w/ metrics | Solution/product-aware buyers evaluating |
| **Expert / influencer** | A credible third party's endorsement | Borrowed authority for a skeptical audience |

Map coverage in `proof.md`: which types and which use-case tags you have, and where the **gaps** are (e.g. "strong ROI quotes, zero security/compliance proof, no enterprise logos"). The gaps are as valuable as the assets — they tell the operator what to go collect.

---

## Output shapes

**Compact — `brand.md` "Proof assets" section** (what other skills read at a glance; keep it short — the top, cleared, strong assets only):

```markdown
## Proof assets
**Headline proof:** [e.g. 4.8★ on G2 (320 reviews) · 10,000+ teams · SOC 2 Type II]  [verify any unconfirmed]
**Top quotes (cleared):**
- "[load-bearing line]" — Name, Title, Company · proves: [tag] · [metric]
- "[…]" — Name, Title, Company · proves: [tag]
**Hero stats (cleared):** [−22% churn in 60d · 3× faster onboarding]  ([verify] where unconfirmed)
**Logos (cleared/NDA-noted):** [Brand, Brand, Brand]
**Full vault + permission status + gaps →** brands/<slug>/proof.md
```

**Full — companion `brands/<slug>/proof.md`:** the complete normalized table (all fields, every asset including `unknown`/`weak`), grouped by typology, plus the coverage/gap map and a short "needs clearance" list with ready-to-send asks.

---

## Principles (Non-Negotiable)

- **Brand-brain first (standalone).** Resolve the brand through `brand-brain`; don't reimplement brand context. When called by it, use its context and never recurse.
- **Own your lane only.** Write exactly the `brand.md` "Proof assets" section and `proof.md`. Never touch another section or file; SUGGEST microcopy for sibling skills, don't write theirs.
- **Real proof only — never invent.** No fabricated quotes, names, metrics, logos, or awards. If it isn't sourced, it isn't proof.
- **`[verify]` every unconfirmed number.** A stat without traceable provenance is a draft claim, not a fact.
- **Permission is load-bearing.** Tag every asset; never promote `unknown` to usable; honor NDA/restriction exactly.
- **Attribution beats anonymity.** Push toward name + title + company; weakening attribution is a permission decision, not a default.
- **Specific beats vague.** A number with a baseline and timeframe outranks an adjective. Keep both, lead with the specific.
- **Data outside the skill folder.** Proof files live in the resolved data root, never inside the skill.

## What Not to Do

- Don't invent, embellish, or "tighten" a quote, metric, name, or logo into something the source didn't say.
- Don't publish `unknown`-permission proof or strip a real reviewer's platform attribution to pass it off as a direct testimonial.
- Don't write campaign copy, headlines, CTAs, or case-study prose — that's other skills' job; hand them tagged assets.
- Don't overwrite any `brand.md` section other than "Proof assets," and don't write `brand.md` at all in CALLED mode.
- Don't drop messy or weak assets — normalize and flag them; the gap map needs the full picture.
- Don't call `brand-brain` when it already passed you context (no recursion).

## Quality checklist (self-review before presenting)

- Mode detected correctly — context used (CALLED) or `brand-brain` invoked + brand loaded (STANDALONE)?
- Every asset normalized to the full frame (quote · attribution · type · use-case tag · metric · strength · permission · source)?
- Permission status set on every asset; nothing `unknown` leaked into the cleared/compact view; NDA restrictions noted?
- Every unconfirmed number marked `[verify]`; no invented proof, names, or logos?
- Typology coverage + gap map present in `proof.md`; "needs clearance" list with ready-to-send asks?
- STANDALONE: only the "Proof assets" block replaced in `brand.md`, `updated` bumped, `proof-vault` appended to `sources`, `proof.md` written under the data root — and confirmed where? CALLED: block (+ `proof.md` path) returned, `brand.md` left for the caller?
