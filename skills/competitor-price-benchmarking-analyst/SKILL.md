---
name: competitor-price-benchmarking-analyst
description: >
  Takes your pricing page and a list of competitors and returns a structured comparison matrix
  of tiers, value metrics, price points, and gaps — identifying where you are over-priced,
  under-priced, or simply framing value less clearly than rivals do. Goes beyond a raw price
  table: it surfaces the value metric each competitor charges on (seats, events, MAU, revenue
  share), the tier-boundary logic, the "good enough" free or entry tier that anchors the
  category, and the moments where competitor packaging creates an opening you could exploit.
  Output is a decision-ready matrix plus a written diagnostic with prioritized pricing
  recommendations grounded in the brand's current offer and ICP. Calls `brand-brain` to anchor
  every insight in the brand's actual positioning, voice, and proof — never in generic
  benchmarks. Use when the user says "how do our prices compare," "are we over-priced / under-
  priced," "competitor pricing," "price benchmarking," "what do our rivals charge," "pricing
  page audit vs competitors," "value metric analysis," "tier gaps," or hands over a pricing page
  and asks how it stacks up.
---

# Competitor Price Benchmarking Analyst

Your pricing page is not just a list of numbers — it is a claim about how you slice value and who pays for what. This skill benchmarks that claim against the live market: what your competitors charge, on what value metric, with what tier logic, and where their packaging leaves gaps you could own.

Output is always a structured matrix first, a diagnostic narrative second. The matrix is machine-readable and presentation-ready. The narrative is opinionated — it names the specific over- and under-pricing risks and proposes ranked next moves. It does not hedge into "you may want to consider."

Brand context comes from `brand-brain`. The offer mechanics come from the brand's `brand.md` (or its companion `offer-pricing-brain` output). This skill does not re-derive what your product costs, what your ICP is, or what your positioning says — it gets that from the library's shared context and puts it to work in the benchmark.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — resolves the active brand and returns the digest (voice, ICP, current offer/tier structure, positioning, proof). Competitor benchmarking must anchor in what the brand actually sells and who it sells to.
  Fallback if brand-brain is absent or returns no brand: do a thin direct read of the already-written brand file — `~/.brandbrain/brands/.active` then that brand's `brand.md`; if no `brand.md` exists, ask the user for: current tier names + prices, primary ICP (job title / company size / industry), and the value metric you charge on (seats, MAUs, events, sites, revenue %).
- **`offer-pricing-brain`** (optional, for deep runs) — if a canonical offer brain exists for the brand, call it to get the full tier/feature-fence map. Synthesize inline from `brand.md` when absent.
- **`competitive-intelligence-dossier`** (optional) — if deeper CI context already exists for a competitor, pull it rather than re-scraping. Synthesize inline otherwise.
- **`positioning-reviewer`** (optional, final step) — after the matrix is built, optionally run a positioning review on the recommended re-framing to pressure-test clarity and differentiation.

---

## How a run works

```
Step 0  Load brand context   ──► brand-brain (always first)
Step 1  Gather inputs        ──► your pricing + competitor list + optional scrape
Step 2  Build the matrix     ──► Van Westendorp / value-metric framework
Step 3  Write the diagnostic ──► over/under-pricing, framing gaps, tier logic
Step 4  Output + save        ──► inline matrix + ./pricing/[slug]-price-benchmark.md
```

---

### Step 0 — Load brand context (always first)

Invoke `brand-brain` (Skill tool, `skill: brand-brain`). Use the returned digest to anchor:
- **Your current tier structure** — names, price points, value metric, key feature fences.
- **ICP** — the buyer persona whose willingness-to-pay defines the right price ceiling.
- **Positioning** — what you claim to be better at; gaps in competitor positioning this skill should surface.
- **Proof** — real numbers to use in the diagnostic; unconfirmed numbers get `[verify]`.

Do not proceed to Step 1 before brand context is loaded.

---

### Step 1 — Gather inputs

Minimum inputs needed to run:

| Input | Source |
|---|---|
| Your pricing page URL or copy | User-supplied or brand.md |
| Competitor list (1–5 names, URLs) | User-supplied |

Ask for missing inputs concisely. For a deep run, also accept: feature-comparison CSV, G2/Capterra pricing pages, or a previously built `competitive-intelligence-dossier`.

If the user hands only a competitor name with no URL, note the URL you'll target for each and flag that pricing data scraped from live pages should be verified before being used in external-facing materials.

---

### Step 2 — Build the matrix (Van Westendorp + value-metric framework)

Borrow the spirit of the **Van Westendorp Price Sensitivity Meter** as the diagnostic backbone — not the actual method (no survey here, and these are not VW's derived intersection points). These are loosely VW-inspired sensitivity bands we use here to reason about where a price sits for the ICP:

- **Too cheap (quality signal risk):** below the floor where the ICP would question product quality or support quality.
- **Acceptable range:** where most ICP buyers expect pricing to land.
- **Too expensive:** above the ceiling where the ICP stops and evaluates alternatives.
- **Upper resistance point:** where the ICP hesitates but might still convert with a strong ROI story.

Layer the **value metric analysis** on top:

1. **Identify the value metric** each competitor charges on — per seat, per site, per MAU, per event, per message, per revenue %, per domain, flat fee, or hybrid.
2. **Map the tier-boundary logic** — where each competitor draws the line between tiers (the fence) and what feature or limit drives upgrade.
3. **Identify the anchor tier** — the entry/free plan or cheapest paid plan that sets the ICP's baseline expectation.
4. **Spot packaging asymmetries** — features one competitor puts behind a high tier that another ships at a lower tier, creating a perceived value gap.

Output the matrix in this structure:

```
## Price Benchmark Matrix — [your brand] vs [competitors]
Date: [today]  |  ICP anchor: [from brand.md]  |  Your value metric: [from brand.md]

| Competitor | Tier Name | Monthly Price | Annual Price | Value Metric | Tier Ceiling | Key Fence Features | Free Plan? | Entry CTA |
|---|---|---|---|---|---|---|---|---|
| [your brand] | … | … | … | … | … | … | … | … |
| Competitor A | … | … | … | … | … | … | … | … |
| …            | … | … | … | … | … | … | … | … |

Notes: all prices in USD; [verify] on any number not confirmed from a live page today.
```

If annual vs monthly pricing varies significantly, flag the discount percentage — this is a retention mechanic, not just a billing preference.

---

### Step 3 — Write the diagnostic

After the matrix, write a structured diagnostic in four sections. Be opinionated.

**3a. Value metric verdict**
Is your value metric aligned with how the ICP experiences value? If competitors charge per seat but you charge per site (or vice versa), name the friction this creates for the buyer's mental model. Recommend whether to hold or shift value metrics, with one-line rationale.

**3b. Over- and under-pricing risks**
For each tier in your structure, assess the risk:
- *Over-priced:* your price exceeds the acceptable range for this ICP, competitors undercut you on the same fence features, or the value proof doesn't match the ask. Name the tier and the gap.
- *Under-priced:* you offer more than competitors at a lower price (or equal price), leaving CAC/expansion revenue on the table. Framing fix is often cheaper than a price change.
- *Correctly positioned:* say so. Don't manufacture problems.

**3c. Tier-logic gaps**
Where does competitor packaging create an opening?
- A competitor's middle tier has a ceiling so low that the ICP outgrows it quickly → buyer churn risk for them, acquisition opportunity for you if your tier is stickier.
- A feature you ship at entry-level that competitors gate behind a higher tier → use this aggressively in comparison copy.
- A gap in the competitive ladder where no one serves the $X–$Y buyer well.

**3d. Prioritized recommendations**
Three to five ranked actions, each with the specific change (not "consider revising"), the effort level (copy-only / tier-restructure / value-metric shift), and the win you'd expect.

---

### Step 4 — Output

Always output the matrix inline in the conversation so the user can read it immediately.

Save the full benchmark (matrix + diagnostic) to `./pricing/[slug]-price-benchmark.md` and confirm the path. Tell the user to re-run this skill (or refresh `competitive-intelligence-dossier`) any time a competitor runs a pricing page change — price benchmarks have a 60–90 day shelf life in most SaaS categories [verify for your market].

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No benchmarking before the active brand context loads. The ICP and current offer anchor every verdict; generic "market average" analysis without ICP grounding is noise.
- **Value metric over price point.** The unit of comparison is not just the number — it is what the number is attached to. Two $99/mo products on different value metrics are incomparable without this.
- **Opinionated verdicts.** Over-priced, under-priced, or correctly positioned: pick one per tier. "It depends" is not a deliverable.
- **Live data, dated and flagged.** Any price number used should be noted as "scraped/confirmed [date]" or marked `[verify]`. Pricing pages change. Don't let stale data drive a pricing decision.
- **Framing is often the fix.** Many perceived pricing problems are actually framing problems — you charge the same or less but communicate value worse. The diagnostic surfaces both; the recommended fix matches the actual root cause.
- **No invented proof.** If you can't confirm a feature is in a tier from the live pricing page, mark it `[verify]` rather than assuming.

---

## What Not to Do

- Don't benchmark without loading brand-brain first — you need the ICP and current offer to assess whether a price is too high or too low for *your* buyer.
- Don't compare prices without comparing value metrics — $49/seat/mo vs $49/site/mo is not the same comparison.
- Don't produce a table with no diagnostic — a table of numbers without a verdict is not analysis.
- Don't recommend a price change when the real problem is framing — call it correctly.
- Don't use pricing data older than 90 days without a `[verify]` flag; pricing pages change.
- Don't include more than 5 competitors in one run — above that, the matrix loses signal in noise. Prioritize the ones the ICP actually cross-shops.
- Don't reimplement brand context derivation — call brand-brain.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called; ICP and current tier structure loaded before any analysis?
- Matrix covers: tier name, price (monthly + annual), value metric, tier ceiling, key fence features, free plan presence, entry CTA — for every competitor including the brand itself?
- All prices marked with scrape date or `[verify]`?
- Diagnostic covers: value metric verdict, over/under-pricing by tier, tier-logic gaps, prioritized recommendations?
- Recommendations are specific (not "consider") and matched to effort level?
- Framing problems called out separately from structural pricing problems?
- Output saved to `./pricing/[slug]-price-benchmark.md` and path confirmed to user?
