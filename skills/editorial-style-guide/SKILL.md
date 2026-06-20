---
name: editorial-style-guide
description: >
  Produces and maintains a brand's house style guide — the mechanical and terminological rulebook every
  writer and copy skill defers to: grammar conventions, capitalization/casing, number and date formats,
  formatting standards, preferred terminology, and the banned-words/phrases list. Built as house style over
  an AP base with brand overrides, so a junior writer renders the brand's name, product features, numbers,
  and dates the same way every time. It is a `brand-brain` COMPONENT (Layer-1): callable standalone (it
  invokes `brand-brain` to resolve the active brand) or called BY `brand-brain` during bootstrap/refresh
  (it uses the passed context and returns its section). It OWNS the companion file
  brands/<slug>/style-guide.md and COORDINATES the banned-words list with the brand's Voice section by
  SUGGESTING additions — it never overwrites the voice section or any other brand.md block. Use when the
  user says "style guide," "editorial standards," "house style," "grammar rules," "how do we write X,"
  "what's our capitalization for X," "is it e-commerce or ecommerce," "banned words list," or hands over
  copy and asks whether it follows the brand's conventions.
---

# Editorial Style Guide Generator

Builds the brand's house style — the rulebook for mechanics, terminology, and banned words — so every piece of copy is consistent down to the comma, the casing, and the way the product is named. Voice answers *how the brand should sound*; this answers *how the brand spells, capitalizes, formats, and refers to things*. They are different jobs and this skill owns the second one.

It writes rules, not prose. It does not draft articles, rewrite headlines, or invent the brand voice. When it touches the banned-words list it **suggests** additions for the Voice section to adopt — it never reaches into that section and overwrites it.

---

## Skills this calls

- **`brand-brain`** (required, standalone mode only) — resolves and loads the active brand: its voice adjectives, banned words, preferred lexicon, common CTAs/destinations, and the path to `brand.md`. This skill never re-implements brand scanning, interviewing, or storage; that lives in `brand-brain`, once. **When `brand-brain` called *this* skill, do not call it back** (no recursion) — use the context it passed.

---

## How a run works

```
Step 0  Detect the mode  ──► Called-by-brand-brain | Standalone
Step 1  Load context     ──► use passed context  OR  call brand-brain
Step 2  Build the guide  ──► AP base → brand overrides → terminology → banned words
Step 3  Coordinate       ──► suggest banned-word additions for the Voice section (don't write it)
Step 4  Persist / return  ──► write style-guide.md (+ return suggestions)  OR  return section to caller
```

### Step 0 — Detect the mode

- **Called by `brand-brain`** — the request passes the active **slug**, current `brand.md` content, and scanned raw inputs. Use that context. Do your work. **Return** your contribution (a pointer to the companion file + the banned-word suggestions for the Voice section) for `brand-brain` to fold in. **Do not call `brand-brain` back.**
- **Standalone** — a user ran you directly with no brand context. **Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`) to resolve the active brand and load `brand.md` (it bootstraps on first use). Then do your work and **persist** (Step 4).

### Step 4 — Persist (standalone) or return (called)

- **Write your companion file** at `<data-root>/brands/<slug>/style-guide.md` — the full house style. This is the file you own.
- **Coordinate, don't clobber.** If you derived new banned words, surface them as a **suggested-additions block** for `brand-brain` to merge into the *Banned words / phrases* section. In standalone mode, present the suggestions and ask the user (or hand them to `brand-brain refresh`) — **never edit the Voice section yourself.**
- Confirm where you saved: *"House style saved to `<root>/brands/<slug>/style-guide.md`. Suggested N banned-word additions for the brand's Voice section — run `brand-brain refresh <slug>` to fold them in."*

**Fallback if `brand-brain` is not installed (standalone):** read `<root>/brands/.active` and that brand's `brand.md` directly (`<root>` = `./.brandbrain/` if present, else `~/.brandbrain/`); if none exists, ask the user for brand name + 3 voice adjectives + any existing conventions, then proceed. Always prefer the call.

---

## The house style framework (AP base → brand overrides)

Start from a known base so you never argue first principles, then layer brand decisions on top. Default base: **AP style** (the lingua franca of marketing/editorial). Record the base explicitly, then every override is a deliberate, documented exception. The guide has six sections; fill each.

### 1. Grammar & punctuation
| Decision | Default (AP base) | Capture the brand's call |
|---|---|---|
| Serial (Oxford) comma | AP omits it | Most brands keep it — pick one, apply everywhere |
| Em dash spacing | spaced — like this | spaced vs unspaced; pick one |
| Sentence vs title case (headings) | — | the single biggest consistency lever; decide per surface |
| Contractions | informal: yes | match voice formality |
| Exclamation marks / emoji | sparing | usually a hard cap; cross-check the banned list |
| One space after periods | one | one, always |

### 2. Capitalization & casing
- **Product/feature names** — list the brand's products and features with their *exact* casing (e.g. is it "Web Push," "web push," or "WebPush"?). This is the highest-error-rate area; enumerate, don't generalize.
- **The brand name itself** — exact casing, any required ® / ™, never auto-capitalized mid-sentence unless the brand does.
- **Job titles, headings, UI labels, generic nouns** — title vs sentence case, each settled once.
- **Acronyms** — expand-on-first-use rule; which ones never need expansion.

### 3. Numbers, dates & units
| Item | Recommended house rule |
|---|---|
| Numbers | spell out one–nine, numerals for 10+ (AP); **always numerals for stats, %, money, versions** |
| Percent | "20%" in marketing copy; "20 percent" only if the brand insists on AP prose |
| Money | "$1,200"; large rounds as "$1.2M" / "$3B" — define the threshold |
| Dates | one format, stated (e.g. "June 20, 2026" or "2026-06-20") — never mixed |
| Time | "8 a.m." vs "8 AM" vs "08:00" — pick one |
| Ranges | en dash "10–20" or "10 to 20" — pick one |
| Phone / units | one format each |

### 4. Formatting standards
- Headings hierarchy and case; list style (sentence fragments vs full sentences; terminal punctuation).
- Links: descriptive anchor text, never "click here"; UI elements in **bold**; user input in `code`.
- Quotation marks (curly vs straight), ellipses, trademark placement, file/version naming.

### 5. Preferred terminology
A two-column **Use / Don't use** table — the single most useful page for a junior writer. Source it from the brand's preferred lexicon and from real usage in the scan. Examples of the shape:

| Use | Not | Why |
|---|---|---|
| customers | users | brand frames buyers as customers |
| sign up (verb) / signup (noun) | sign-up everywhere | part-of-speech split |
| set up (verb) / setup (noun) | setup as verb | same pattern |
| ecommerce | e-commerce, eCommerce | one spelling, brand-chosen |

Also fix: industry terms the brand spells a specific way, competitor names (exact casing + linking policy), and the canonical spelling of every recurring domain term.

### 6. Banned words & phrases (coordinated with Voice)
The hard no-list: hype words ("revolutionary," "game-changer," "seamless," "cutting-edge"), filler ("very," "really," "just," "actually"), clichés, AI-slop tells ("delve," "in today's fast-paced world," "unlock the power of"), and any brand-specific tics. Pull the brand's existing banned list from `brand.md` first; **add** to it, don't replace it. Anything you newly propose goes into a clearly labeled **suggested additions** block for the Voice section — you flag the reason for each, and `brand-brain` (or the user) decides.

---

## Companion file shape (`brands/<slug>/style-guide.md`)

```markdown
---
slug: <slug>
updated: <YYYY-MM-DD>
base: AP style
sources: [brand.md, <scan sources>, user interview <date>]
---
# <Brand> — House Style Guide
## 1. Grammar & punctuation        (table of resolved decisions)
## 2. Capitalization & casing      (product/feature name list + rules)
## 3. Numbers, dates & units       (table)
## 4. Formatting standards
## 5. Preferred terminology         (Use / Don't / Why table)
## 6. Banned words & phrases         (the list + a note that Voice owns the canonical set)
## Suggested additions → Voice section   (for brand-brain to merge; reason per item)
```

---

## Principles (Non-Negotiable)

- **One base, documented overrides.** Name the base (AP) and treat every deviation as a deliberate, recorded exception — not an undocumented habit.
- **Decide once, apply everywhere.** A style guide's only value is consistency; every rule resolves to a single answer.
- **Enumerate names, don't generalize.** List actual product/feature names with exact casing — that is where consistency dies.
- **Mechanics, not voice.** This guide governs spelling, casing, numbers, formatting, terminology. It does not set tone — Voice does.
- **Suggest, never overwrite the banned list.** Add proposed banned words as suggestions for the Voice section; `brand-brain` owns the merge.
- **Truth only.** Don't fabricate a brand convention — if it's unsettled, mark it `[verify]` and ask. Don't invent product names.
- **Brand-brain first (standalone).** No guide before `brand-brain` returns the active brand.

## What Not to Do

- Don't write the guide before brand context is loaded (called: from the passed context; standalone: from `brand-brain`).
- Don't call `brand-brain` when `brand-brain` called you (no recursion).
- Don't write or overwrite the Voice & tone / Banned words section, or any other `brand.md` section or file — you own only `style-guide.md`.
- Don't re-implement brand scanning, interviewing, or storage — that's `brand-brain`.
- Don't rewrite the brand's copy, draft articles, or set tone — flag conflicts, don't fix them.
- Don't impose generic AP rules over an explicit brand decision; the brand override always wins.
- Don't invent product names, statistics, or conventions the brand hasn't set.

## Quality Checklist (self-review before presenting)

- Mode detected correctly? (Called → used passed context and did NOT call `brand-brain` back. Standalone → called `brand-brain`, or fell back, before writing.)
- Base named (AP) and every override recorded as a deliberate exception?
- All six sections filled: grammar, casing, numbers/dates, formatting, terminology, banned words?
- Product/feature names enumerated with exact casing — not described in the abstract?
- Each rule resolves to one answer (no "either is fine")? Unsettled items marked `[verify]`?
- Banned-word additions surfaced as **suggestions** for the Voice section, with a reason each — Voice section left untouched?
- Persisted correctly: companion file written to `<root>/brands/<slug>/style-guide.md` (standalone), or section returned to the caller (called) — and the save location confirmed?
- No other `brand.md` section or file written; no fabricated conventions or names?
