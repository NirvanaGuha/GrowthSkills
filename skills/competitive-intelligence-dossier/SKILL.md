---
name: competitive-intelligence-dossier
description: >
  Researches a competitor into a battlecard and a living dossier — their positioning, pricing, ICP,
  strengths, weaknesses, recent moves, the honest "why we win," and the traps/landmines a deal can die
  on. Sources from the website, pricing page, G2/Capterra reviews, job boards, LinkedIn, and the
  changelog, then mines it into something a rep or a writer can act on. It is a `brand-brain` COMPONENT:
  callable standalone (a user runs it directly) or by `brand-brain` during bootstrap/refresh. It OWNS
  the companion file `brands/<slug>/competitors.md` and SURFACES a differentiation summary that FEEDS
  the positioning skill's "Value proposition & differentiators" section — it suggests, it does not
  overwrite. Use when the user says "competitive analysis," "competitor dossier," "build a battlecard,"
  "research competitor X," "how do we beat Y," "what's our wedge against Z," or hands over a rival's URL.
---

# Competitive Intelligence Dossier

Point it at a competitor, get a battlecard a rep could win a call with and a dossier that stays true over time. It researches the rival in *your* brand's frame — what they're strong at, where they're soft, where deals die, and the honest reason a buyer picks you — because your own context comes from the shared `brand-brain`, not from guessing.

This skill researches and writes the competitor file. It does **not** rewrite your positioning. It hands the positioning skill a tight differentiation summary as a suggestion; the positioning section stays owned by positioning.

---

## Skills this calls

- **`brand-brain`** (required, standalone mode) — resolves the active brand and loads its `brand.md` so the battlecard is framed against *your* real ICP, offer, proof, and positioning. This skill does not implement brand resolution, scanning, or storage; that lives in `brand-brain`, once.
- *(optional, when installed)* `proof-vault` for the proof you counter their claims with; `objection-library-builder` for the objections a competitor seeds. Synthesize inline when absent.

---

## How a run works

```
Step 0  Detect the mode  ──► CALLED by brand-brain  |  STANDALONE
Step 1  Load your brand   ──► use passed context (called)  |  call brand-brain (standalone)
Step 2  Research          ──► source → mine → fill the battlecard (one per competitor)
Step 3  Surface           ──► a differentiation summary as a suggestion for positioning
Step 4  Persist / return  ──► write competitors.md (standalone)  |  return your block (called)
```

### Step 0 — Detect the mode
- **Called by `brand-brain`** — the request passes the active **slug + current `brand.md` + scanned raw inputs** (likely a competitor list or URLs). Use that context. Do the work. **Return** your dossier/battlecard block and a differentiation summary for brand-brain to fold in. **Do not call `brand-brain` back** (no recursion).
- **Standalone** — a user runs you directly with a competitor name/URL and no brand context. **Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`) to resolve the active brand and load its `brand.md`. Do the work. Then **persist** (Step 4).

### Step 1 — Load your brand
You can't write a battlecard without knowing who *you* are. From `brand-brain` (or the passed context) take: your positioning line, ICP(s) + the metrics they own, your real differentiators, offer mechanics, and real proof. A "why we win" written without your own proof is fiction.

**Fallback if `brand-brain` is not installed (standalone):** read `<data-root>/brands/.active` and that brand's `brand.md` directly (`~/.brandbrain/` global or `./.brandbrain/` per-project); if none exists, ask the user to install `brand-brain` (preferred) or give you a 3-line brand sketch (what you are · your ICP · your 2–3 differentiators), then proceed. Always prefer the call.

### Step 2 — Research one competitor → one battlecard
Run the sourcing pass, then mine it into the battlecard (below). One competitor = one dossier block in `competitors.md`. Never blur two rivals into one card.

---

## Sourcing — where the truth actually lives

Pull from primary sources first; reviews and job boards are where the real story leaks. Note the source per fact so a refresh knows what to re-check.

| Source | What it reliably tells you | What to extract |
|---|---|---|
| **Homepage + product pages** | Their *claimed* positioning, target persona, headline value prop | Positioning line, primary ICP, feature emphasis, the category they're claiming |
| **Pricing page** | Packaging, tiers, what's gated, who they price *for* | Entry price, model (seat/usage/flat), free/trial terms, the "talk to sales" floor, hidden-cost signals |
| **G2 / Capterra / TrustRadius** | What buyers actually love and hate (the gold) | Recurring praise = real strengths; recurring complaints = exploitable weaknesses; star trend; segment of reviewers |
| **Job boards (their careers page, LinkedIn jobs)** | Where they're *investing* — the roadmap before it ships | Hiring spikes (new market? new product line?), tech stack, ICP shift signals |
| **LinkedIn (company + leaders)** | Headcount trajectory, funding, narrative, leadership churn | Growth/contraction, recent raises, the story they tell investors, exec departures |
| **Changelog / release notes / blog** | Recent moves and velocity | What shipped in the last 1–2 quarters, cadence, where momentum is (and isn't) |
| **Review mining (cross-cut)** | The *pattern* across all of the above | The objection they seed about you; the wedge they can't close |

**Truth discipline:** every number, claim, and quote is real or marked `[verify]`. Never invent a price, a customer name, a star rating, or a weakness. "Couldn't confirm" is a finding, not a failure.

---

## The battlecard (one per competitor)

The framework. Fill every row; an empty row is a research gap to name, not skip.

| Section | What goes here |
|---|---|
| **Snapshot** | Name · URL · category they claim · stage/size · last-checked date |
| **Their positioning** | The frame *they* push (in their words), and the persona it's aimed at |
| **Pricing & packaging** | Model, entry price, what's gated, free/trial terms, the sales-floor — all `[verify]` unless on their page today |
| **Their ICP** | Who they actually win — by segment/size; where it overlaps yours and where it doesn't |
| **Strengths (real)** | What they're genuinely good at — from reviews + product, not their marketing. Respect these; underrating a rival loses deals. |
| **Weaknesses (exploitable)** | Recurring complaints, gaps, friction — each one usable in a sales conversation, not a cheap shot |
| **Recent moves** | Last 1–2 quarters: launches, raises, hires, repositioning, pricing changes |
| **Why we win** | The honest, provable reasons *our* ICP picks us over them — each tied to one of OUR real differentiators + proof. No proof → don't claim it. |
| **Where they win** | Be honest: the segments/use-cases where *they* are the better pick. Knowing this is how you stop chasing wrong-fit deals. |
| **Traps / landmines** | The objection they plant about us, the feature checkbox they bait with, the "but can you do X" — and the one-line counter for each |
| **Talk track** | 2–3 ready lines a rep/writer uses when this competitor comes up — reframe, not trash |

**Rules of the card:** Win on *your* strengths, not their weaknesses (a weakness-only card ages badly and sounds bitter). Every "why we win" maps to a real differentiator + proof you actually hold. The "where they win" row is mandatory — a card with no honest losses is propaganda, and reps stop trusting it.

---

## Surfacing the differentiation summary (the feed)

After the battlecard(s), distill a **differentiation summary**: 3–7 lines stating where *your* brand is genuinely, defensibly different across the competitive set — each grounded in a real differentiator + the proof that backs it. This is the one thing competitive work owes the rest of the library.

**This is a SUGGESTION for the positioning skill's "Value proposition & differentiators" section — you do not write that section.** You hand it over (to `brand-brain` in called mode, or note it for `positioning-messaging-architect` in standalone). Frame it as *"proposed differentiators, with evidence — for positioning to accept/refine,"* never as a finished rewrite. Flag any place your claimed differentiator is actually competitive parity (table stakes) — that's the highest-value thing you can tell positioning.

---

## Persist / return

**Standalone — persist, then confirm where:**
- **Companion file** — write/update `<data-root>/brands/<slug>/competitors.md`: one dossier block per competitor (frontmatter `updated` + `sources`), a "Competitive set" index at top, and the differentiation summary at the bottom. Upsert by competitor name — never duplicate a card; bump that card's `updated` and re-check stale rows.
- **`brand.md`** — this skill owns **no** `brand.md` section. Do **not** edit `brand.md`. Surface the differentiation summary as a suggestion for the "Value proposition & differentiators" section and tell the user to run `positioning-messaging-architect` (or `brand-brain refresh`) to fold it in. Append yourself to `sources` only if positioning/brand-brain writes it — not here.
- Confirm in one line: *"Dossier saved to `<root>/brands/<slug>/competitors.md` — N competitors carded. Differentiation summary ready for positioning."*

**Called by `brand-brain` — return, don't write:**
- Return the dossier/battlecard block(s) **and** the differentiation summary as structured content for brand-brain to fold in (it owns the files; competitors.md may be written by it or delegated to you per its convention). Do not write `brand.md`. Do not call `brand-brain` back.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No battlecard before your own brand context is loaded — "why we win" needs your real proof.
- **Stay in your lane.** You own `competitors.md`. You *suggest* differentiators to positioning; you never overwrite the positioning/value-prop section or any other `brand.md` field or file.
- **Respect the rival.** Map their real strengths honestly; an underrated competitor wins deals you thought you had.
- **Honest losses included.** Every card states where *they* win. A card with no losses is propaganda and reps stop trusting it.
- **Evidence or `[verify]`.** Real prices, ratings, quotes, and moves — or marked unconfirmed. Never invent a number, a customer, or a weakness.
- **Win on your strengths.** Differentiation tied to your proof, not a list of their flaws.
- **Living, not one-shot.** Pricing and changelogs move; the dossier carries last-checked dates and refreshes on real change.

## What Not to Do

- Don't write battlecards before `brand-brain` returns your active brand (standalone).
- Don't reimplement brand resolution/scanning/storage — call `brand-brain`.
- Don't overwrite the positioning "Value proposition & differentiators" section, any other `brand.md` section, or any file but `competitors.md`. Suggest; don't seize.
- Don't call `brand-brain` back when you were called by it (no recursion).
- Don't invent prices, ratings, customer names, funding, or weaknesses — `[verify]` or omit.
- Don't write a trash-talk card (no "where they win" row, weaknesses only) — reps won't use it.
- Don't claim parity features as differentiators — flag table stakes for positioning instead.
- Don't merge two competitors into one card or duplicate an existing card on refresh.

## Quality Checklist (self-review before presenting)

- Mode detected correctly; in standalone, `brand-brain` called and your brand loaded before any card?
- One battlecard per competitor, every row filled (or the gap named), each fact sourced and dated?
- "Why we win" lines each tied to a REAL differentiator + proof you hold; the "where they win" row honestly filled?
- All prices/ratings/quotes/moves real or `[verify]`; nothing invented?
- Differentiation summary surfaced as a *suggestion* for positioning — parity claims flagged — not written into `brand.md`?
- Standalone: `competitors.md` written (upserted, not duplicated) and the save location confirmed? No other file touched?
- Called mode: returned the block + summary, did not write `brand.md`, did not recurse into `brand-brain`?
