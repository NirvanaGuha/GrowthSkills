---
name: competitor-ad-library-spy
description: >
  Pulls and analyzes live competitor creatives, angles, and offers from Meta Ad Library,
  Google Ads Transparency Center, and TikTok Creative Center — then synthesizes a structured
  intelligence brief showing what's running, what's been running the longest (proven creative),
  and what creative angles your competitor is betting on. Input is just a competitor name (or URL).
  Output is a ranked angle map, proven-asset highlights, offer/CTA inventory, and a gap list of
  angles your competitors have not claimed that you could own. Compose with competitive-intelligence-dossier
  for full market context, ad-to-landing-page-message-match-auditor to verify their funnels, and
  social-ad-copy-writer to write counter-creatives. Use whenever the user says "spy on competitor ads,"
  "what ads is [brand] running," "find competitor ad angles," "check their Meta/Google/TikTok library,"
  "competitive ad research," or "what creative is working for [competitor]."
---

# Competitor Ad Library Spy

Pull what competitors are running. Learn what's working. Find the angles they haven't claimed yet.

Every paid media team should know what their competitors are actually betting on — not by guessing from their homepage, but by reading their live and long-running ads directly. This skill structures the three-platform ad library pull (Meta, Google, TikTok) into a single intelligence brief: proven creatives, active angles, offer mechanics, and the gaps you can move into.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's voice, ICP, positioning, and real proof so the gap analysis and counter-angle recommendations are specific to your brand, not generic. Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their brand's positioning, ICP, and key differentiators before proceeding.
- **`competitive-intelligence-dossier`** (compose, optional) — if a full competitive profile doesn't already exist for this competitor, route there to build the broader context (messaging, pricing, review themes); ad intel feeds into it as the paid-signals section.
- **`ad-to-landing-page-message-match-auditor`** (compose, optional) — after surfacing competitor ad copy, pass a flagged ad + its destination URL to this skill to audit how tight their funnel is; reveals where their message degrades on click.
- **`social-ad-copy-writer`** (compose, optional) — after the gap analysis, route to this skill to draft counter-creatives targeting the identified open angles.
- **`data-qa-measurement-gotcha-checker`** (gate) — invoke before drawing conclusions from library data to flag platform-level gotchas: Meta library only shows ads active within 7 days and omits spend data; Google Transparency shows verified advertisers only; TikTok Creative Center is sampled, not exhaustive. Mark any inference that depends on volume/spend as `[verify]`.

---

## How a run works

```
Step 0  Load brand context      ──► call brand-brain (voice, ICP, positioning, proof)
Step 1  Identify target(s)      ──► resolve competitor slug(s) and platforms to pull
Step 2  Pull library data        ──► Meta → Google → TikTok, structured per platform
Step 3  Score and synthesize    ──► proven assets, active angles, offer inventory
Step 4  Gap analysis             ──► angles competitors haven't claimed; your openings
Step 5  Deliver brief + options  ──► present brief; offer to route downstream
```

---

## Step 0 — Load brand context (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). Use the returned digest to orient the gap analysis: angles unclaimed by competitors are only worth flagging if they are credible for *your* brand's ICP, positioning, and proof. Obey voice and banned-words if counter-creative copy is produced here.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for their brand's positioning, ICP, and key differentiators before proceeding.

---

## Step 1 — Identify targets and scope

Resolve:
- **Competitor slug(s):** exact name, domain, or Meta/TikTok page name. If multiple competitors are named, run each separately and synthesize in the gap analysis.
- **Platforms:** default is all three (Meta, Google, TikTok). If the user's category skews B2B (low TikTok probability) or D2C (low Google Transparency volume), note it and deprioritize — don't skip entirely.
- **Recency window:** default is 90 days of active ads. For proven-creative detection, flag ads running >90 days as "evergreen tested."

---

## Step 2 — Platform pulls (the Ad Intelligence Framework)

Treat the three libraries as complementary, not redundant. Each signals different intent.

### Meta Ad Library (`facebook.com/ads/library`)
- Search by advertiser name (page name), filter by country and ad category.
- Pull: ad copy (primary text, headline, description), creative format (image/video/carousel), CTA button, destination URL, and start date (or "Active since" where shown).
- **Proven creative signal:** ads with the earliest start date still running = the highest-confidence winner in their portfolio. Meta does not show spend, so duration is the proxy.
- Flag data limits per `data-qa-measurement-gotcha-checker`: library only surfaces ads active in the last 7 days; paused ads disappear; spend and impression counts are not shown; political/social issue ads have extended archive.

### Google Ads Transparency Center (`adstransparency.google.com`)
- Search by advertiser domain or name. Pull: headlines, descriptions, extensions, formats, and "Last seen" dates.
- **Proven creative signal:** ads seen across many dates or with long "First seen → Last seen" spans.
- Flag: only verified advertisers appear; Performance Max asset groups may not be individually surfaced; local/shopping inventory is often absent.

### TikTok Creative Center / Ad Library
- Use Top Ads search filtered by competitor name (where available) or industry + keyword. Pull: hook text, on-screen copy, CTA, product demo style (talking head, UGC, animation).
- **Proven creative signal:** high-interaction ads surfaced by TikTok's own ranking = social proof of performance.
- Flag: the Creative Center shows a sampled set; direct competitor lookup may be incomplete if the brand doesn't run TikTok aggressively.

---

## Step 3 — Score and synthesize

### Proven Asset Highlights
Flag the top 3–5 creatives across all platforms using this criterion: **long runtime (>90 days) or platform-ranked as top-performing.** These are the creative bets the competitor has already validated. For each, record:

| Asset | Platform | Format | Duration / Signal | Primary angle | Offer / CTA |
|---|---|---|---|---|---|

### Active Angle Map (the Persuasion Layer)
Tag every ad with one primary angle from the Creative Angle Taxonomy (below). Then count frequency per angle. High frequency = heavy investment. Low frequency from a capable competitor = either untested or deliberately avoided.

**Creative Angle Taxonomy** (name the angle, not the format):
1. **Value/price** — savings, ROI, plan comparison, free tier
2. **Problem/pain** — frustrated state, before, cost of inaction
3. **Social proof** — customer count, logos, testimonials, reviews, ratings
4. **Feature/capability** — specific feature demo, integration, workflow
5. **Transformation** — before/after, identity shift, future self
6. **Urgency/scarcity** — limited offer, countdown, event-gated
7. **Curiosity/hook** — question, surprising stat, counterintuitive claim
8. **Comparison/competitive** — direct vs., switching cost, migration
9. **Trust/authority** — press logos, certifications, award, founder credibility
10. **Use case/vertical** — industry-specific outcome, role-specific story

### Offer Inventory
List every distinct CTA / offer type seen: free trial, demo request, discount code, lead magnet, direct purchase, waitlist, etc. Note which platforms run which offers. This reveals their conversion strategy by funnel stage.

---

## Step 4 — Gap analysis (the output that drives decisions)

The gap analysis is the brief's most actionable section. It answers: **what creative angles are competitors not claiming, and are those angles credible for your brand?**

Cross-reference the Active Angle Map against your brand's proven proof points (from `brand-brain`) and ICP. Flag:

1. **Claimed angles** — competitor is heavy here; entering requires differentiation, not just imitation.
2. **Thin angles** — competitor has a few ads but no sustained investment; a coordinated push could gain share of attention.
3. **Unclaimed angles** — not present in their library at all; assess whether your brand has the proof to credibly own this angle. If yes, flag as HIGH priority. If no proof yet, mark `[verify before using]`.
4. **Your differentiation windows** — specific claims or proof points visible in your `brand.md` that the competitor is not advertising. These are asymmetric advantages.

```
## Gap Analysis — [Your Brand] vs. [Competitor]
| Angle | Competitor coverage | Your credibility | Priority |
```

---

## Step 5 — Brief delivery and downstream routing

Deliver the full brief inline or save to `./competitive/[competitor-slug]-ad-spy-[YYYY-MM-DD].md` if the user asks to save.

At the end, offer three downstream paths:
- "Route to `competitive-intelligence-dossier` to fold this into a full competitive profile."
- "Route to `ad-to-landing-page-message-match-auditor` to audit the competitor's funnel on their top ad."
- "Route to `social-ad-copy-writer` to draft counter-creatives on the highest-priority unclaimed angle."

---

## Principles (Non-Negotiable)

- **Brand-brain first.** The gap analysis is meaningless without knowing your brand's positioning and proof. Never produce it before `brand-brain` returns.
- **Duration = proxy for performance.** Ad libraries don't show spend. Long runtime is the best available signal for a winner. Say so explicitly — don't imply precision that isn't there.
- **Platform gotchas declared upfront.** Each library has coverage gaps. Call them before drawing conclusions, not in a footnote. Invoke `data-qa-measurement-gotcha-checker` before finalizing inferences.
- **Angles, not ad count.** Reporting "they have 47 Meta ads" is noise. Reporting "they're 80% invested in social proof and feature demo, with nothing on pain/problem" is intelligence.
- **Only what's verifiable.** Never infer spend, reach, or ROAS from library data. Mark any performance claim that isn't directly observable as `[verify]`.
- **Gaps must clear the proof bar.** An unclaimed angle is only an opportunity if your brand can back it up. If you can't, say so — don't hand over a list of claims you can't substantiate.

---

## What Not to Do

- Don't produce the gap analysis before `brand-brain` loads the active brand's context.
- Don't conflate "many ads" with "high spend" — volume in the library reflects creative variety, not budget.
- Don't invent performance data; the libraries don't expose it.
- Don't recommend a competitor's exact angle verbatim — the goal is to own an unclaimed position, not copy.
- Don't skip the data-quality flags because they're uncomfortable — a misinformed media decision based on incomplete library data costs real budget.
- Don't treat an unclaimed angle as automatically winnable; check the proof requirement first.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (positioning, ICP, proof in hand)?
- All three platforms attempted; coverage gaps flagged with explicit `data-qa` caveats?
- Proven assets identified by duration/platform signal, not ad count?
- Angle map uses the 10-category taxonomy; frequency tallied per angle?
- Gap analysis cross-references competitor coverage against your brand's proof — every HIGH priority gap has a stated proof basis or `[verify]`?
- Offer inventory complete; downstream routing options offered?
- No invented spend, reach, or ROAS figures; all performance inferences qualified?
