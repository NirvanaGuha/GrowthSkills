---
name: channel-strategy-selector
description: >
  ICP description + budget range + stage of growth → ranked channel recommendations with
  channel-market-fit scores and rationale. Uses the Channel-Market Fit (CMF) framework to
  score every channel against four dimensions: ICP reachability, unit economics fit, stage
  readiness, and proof of traction — then returns a prioritized shortlist, not a menu.
  Calls brand-brain for ICP, positioning, and voice context; composes with icp-persona-builder,
  competitive-intelligence-dossier, and channel-roi-scorecard when present. Use when the user
  says "what channels should we invest in," "help me pick a channel," "channel strategy,"
  "where should we spend our marketing budget," "channel mix," "which channels for growth,"
  or hands over an ICP and budget and asks where to focus.
---

# Channel Strategy Selector

Give it an ICP and a budget, get a ranked, scored channel shortlist with a clear primary bet and a
tightly reasoned rationale — not a list of everything that might work. The job is to surface
channel-market fit, surface the bets that match the brand's stage and economics, and name the ones
to ignore so the team doesn't spread thin.

This skill ranks and recommends. It does not write ad copy, build campaigns, or model unit
economics in granular detail — it names the right lanes so downstream skills (paid-campaign-spin-up,
lead-nurture-drip-builder, etc.) can execute in them.

---

## Skills this calls

- **`brand-brain`** (required) — loads ICP, positioning, voice, offer mechanics, and real proof
  before any channel is evaluated.
- **`icp-persona-builder`** *(when present)* — if no ICP exists yet, delegate its creation here
  before scoring channels.
- **`competitive-intelligence-dossier`** *(when present)* — call to identify which channels
  competitors have saturated vs. where they have gaps; synthesize inline if absent.
- **`channel-roi-scorecard`** *(when present)* — pull its historical CAC/LTV/payback data for
  existing channels to anchor the CMF scoring; synthesize from user-provided numbers if absent.

---

## How a run works

```
Step 0  Load brand context        ──► call brand-brain (always first)
Step 1  Establish the inputs      ──► confirm ICP, budget tier, stage, existing channels
Step 2  Score every candidate     ──► run the CMF framework (four dimensions, one table)
Step 3  Produce the shortlist      ──► Primary Bet + Supporting Channels + Deprioritized list
Step 4  Rationale + playbook      ──► sequencing, green flags, kill criteria
Step 5  Save and surface          ──► ./channel-strategy/[slug]-channel-strategy.md
```

---

## Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`) before evaluating any channel.
It returns the active brand's ICP, awareness tendency, positioning, offer mechanics, and real proof.
Channels that cannot reach the ICP or cannot communicate the offer's value exchange are scored down
immediately — before the framework runs.

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` and that
brand's `brand.md` directly. If none exists, ask the user: (1) who is the ICP (role, company type,
pain, awareness stage), (2) what is the offer and its primary destination, (3) budget range per
month, (4) current stage. Proceed only after these four answers.

---

## Step 1 — Establish the inputs

Confirm or gather the four required inputs. When brand-brain is loaded, most fields pre-fill.

| Input | What to confirm |
|---|---|
| **ICP** | Role, company type, channel-specific behavior (where they actually spend attention) |
| **Budget tier** | Seed (< $3 k/mo), Early ($3–15 k/mo), Growth ($15–75 k/mo), Scale (> $75 k/mo) |
| **Stage of growth** | Pre-PMF, Post-PMF/traction, Scaling, Mature/defend |
| **Existing channels** | What is already working (even lightly) — preserve optionality |

If the user provides rough inputs, map them to the tiers and state your interpretation before scoring.

---

## Step 2 — The CMF Framework (Channel-Market Fit scoring)

Score every plausible channel on four dimensions, each 1–5. Sum to a CMF score out of 20. Only
channels with CMF ≥ 12 enter the shortlist; 10–11 are "watch list"; ≤ 9 are deprioritized.

### Dimension 1 — ICP Reachability (1–5)
Can you reach a meaningful concentration of this ICP on this channel at a realistic cost?

| Score | Meaning |
|---|---|
| 5 | ICP is dense and actively engaged on this channel; targeting is precise |
| 4 | ICP is present, targeting is workable with some noise |
| 3 | ICP is reachable but dispersed; requires significant waste budget |
| 2 | ICP is present but not the primary demographic; signal-to-noise is poor |
| 1 | ICP is not on this channel in meaningful volume |

### Dimension 2 — Unit Economics Fit (1–5)
Does the channel's cost structure match the ACV / LTV / payback window of this offer?

| Score | Meaning |
|---|---|
| 5 | CAC ceiling comfortably absorbs channel CPCs/CPMs at this budget; payback < 6 mo |
| 4 | Economics are workable; payback 6–12 mo at expected conversion rates |
| 3 | Possible but tight; requires top-of-range conversion to be profitable |
| 2 | Structurally expensive for this ACV; would need 2× better-than-industry conversion |
| 1 | Channel economics are incompatible with the offer's unit economics |

### Dimension 3 — Stage Readiness (1–5)
Is this channel appropriate for the current stage of growth?

| Stage → Channel fit notes |
|---|
| **Pre-PMF** — favor direct, high-signal, low-cost channels (communities, cold outreach, content SEO); avoid paid until conversion rate is validated |
| **Post-PMF/traction** — begin paid on proven ICP segments; SEO compounds; email/lifecycle builds retention |
| **Scaling** — channel diversification; paid at volume; partner/affiliate; brand PR |
| **Mature/defend** — loyalty programs, community, word-of-mouth, ABM for expansion revenue |

Score 5 if the channel is a canonical fit for the stage; 1 if it belongs to a different stage.

### Dimension 4 — Proof of Traction (1–5)
Is there internal or competitor evidence that this channel works for this offer + ICP combination?

| Score | Meaning |
|---|---|
| 5 | Internal data shows it is working or a close competitor has scaled it visibly |
| 4 | One directional signal (early campaign data, a competitor reference, industry benchmark) |
| 3 | No direct evidence but strong analogical fit (same ICP, similar offer in adjacent category) |
| 2 | Limited analogical evidence; would be a greenfield test |
| 1 | No evidence; pure hypothesis; high-risk first bet |

**Score table format:**

```
| Channel            | ICP Reach | Unit Econ | Stage Fit | Traction | CMF /20 | Tier |
|--------------------|-----------|-----------|-----------|----------|---------|------|
| [Channel name]     |    n      |    n      |    n      |    n     |   sum   | P/S/W/D |
```

Tier: **P** = Primary Bet, **S** = Supporting, **W** = Watch List, **D** = Deprioritized.

---

## Step 3 — The shortlist

**Primary Bet (1 channel):** The single highest-CMF channel the brand should concentrate on first.
State explicitly: this is where the primary budget and attention go. If two channels tie closely,
name the one with better stage fit as primary and the other as first Supporting.

**Supporting Channels (1–3):** Channels that compound the Primary Bet or cover funnel stages it
cannot reach (e.g., SEO + email if primary is paid; community + content if primary is SEO). Cap at
three — a team cannot execute five channels well.

**Deprioritized (named list):** Every channel scored ≤ 9, one sentence each on why it fails at
this stage/ICP. This is as important as the shortlist — it ends the "but what about [channel X]?"
distraction.

---

## Step 4 — Rationale and sequencing playbook

For each shortlisted channel deliver:

1. **Why this channel at this stage:** one precise paragraph anchored to the CMF scores.
2. **ICP signal evidence:** where the brand-brain data (or user-provided info) confirms ICP
   concentration; cite real channels/communities/platforms, not generalities.
3. **Budget allocation guidance:** rough split across Primary and Supporting (e.g., 60/30/10);
   flag if budget tier requires sequencing (Primary only until PMF, then add Supporting).
4. **Green flags — when to scale:** 2–3 specific, observable signals that mean this channel is
   working and deserves more budget (e.g., paid CAC < 40% of ACV, organic MoM traffic > 15%,
   reply rate on cold outreach > 8%).
5. **Kill criteria:** 1–2 specific conditions that trigger a deprioritize decision (e.g., after
   $5k spend and 200 clicks, CPA > 3× ACV; after 90 days, no keyword reaching page 2).
6. **First 30-day action:** the single most important thing to do in the first 30 days on this
   channel, stated as an action, not a category (e.g., "Set up a tightly themed Google Search
   campaign targeting 8–12 bottom-funnel keywords at $50/day; pause before expanding to Display").

---

## Step 5 — Save and surface

Save the full output (score table + shortlist + rationale playbook) to:
`./channel-strategy/[brand-slug]-channel-strategy.md`

Create the `./channel-strategy/` directory if it does not exist. Confirm the path at the end.
If the user is in a team context, surface the summary table inline in the response for
asynchronous sharing.

---

## Channel reference (CMF anchors by type)

Use these as scoring anchors, not defaults. The CMF score is always ICP- and stage-specific.

| Channel type | Canonical stage fit | Unit economics floor (ACV guidance) | ICP signal source |
|---|---|---|---|
| SEO / organic content | Post-PMF → Scale; compounds over 6–18 mo | Works at any ACV; slow payback | GSC, Ahrefs, GA4 landing pages |
| Paid Search (Google/Bing) | Post-PMF; needs validated CVR first | ACV > ~$500 to sustain; CPCs vary by category | Keyword intent, competitor terms |
| Paid Social (Meta/LinkedIn/TikTok) | Post-PMF → Scale; Meta for B2C/SMB, LinkedIn B2B | Meta: ACV > $200; LinkedIn: ACV > $3k | ICP job titles, interest clusters |
| Cold Outbound (email/LinkedIn DM) | Pre-PMF → Traction; high-signal, low-cost | Scales with ACV > $2k+ (time cost high) | LinkedIn Sales Nav, Apollo, intent |
| Community / PLG / Product Virality | Pre-PMF → Scale; requires product-loop | Very low CAC; depends on virality coefficient | Subreddits, Slack groups, Discords |
| Email / lifecycle (owned) | All stages; highest ROI at retention | Near-zero marginal cost; list is the asset | ESP open/click segmentation |
| Partner / affiliate / integration | Traction → Scale; requires existing product | Low CAC; revenue-share model | Partner ecosystem, marketplace data |
| Content / thought leadership (social) | Pre-PMF → Traction; trust-building | Low cost; slow payback | LinkedIn, X (Twitter), newsletter |
| PR / earned media | Scale → Mature; credibility multiplier | Unpredictable ROI; not primary acquisition | Press, analyst coverage, podcasts |
| Events / webinars | Post-PMF → Scale; high-touch conversion | High cost per lead; best for ACV > $5k | Conference attendance, intent signals |

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No channel scored before the active brand's ICP and offer are loaded.
- **One primary bet.** The output has one primary channel, not a five-way tie. If the user pushes
  back, explain stage economics — don't cave to FOMO.
- **Score against the ICP, not the channel's general reputation.** LinkedIn is not inherently
  "good for B2B"; it is good if the ICP is active there and ACV supports the CPCs.
- **Name the no's as clearly as the yes's.** Deprioritized channels must be named and explained —
  that clarity is half the value of the skill.
- **Stage gates everything.** A channel that is right at Scale is wrong at Pre-PMF. Stage fit
  is not a soft consideration — a Pre-PMF brand on brand awareness spend is a burn problem.
- **Real evidence or `[verify]`.** CAC benchmarks, competitor channel claims, and traction
  signals must come from real data or be marked `[verify]`. Never invent proof.

---

## What Not to Do

- Don't produce a ranked list of every channel with "it depends" commentary — that's not a
  strategy, it's a menu. Force the ranking.
- Don't score channels without loading brand-brain first; ICP reachability cannot be evaluated
  from a generic prompt.
- Don't reimplement ICP building, competitive research, or CAC modeling — call `icp-persona-builder`,
  `competitive-intelligence-dossier`, and `channel-roi-scorecard` when they are available.
- Don't skip the deprioritized list. The team needs explicit permission to ignore channels.
- Don't recommend more than one Primary Bet and three Supporting channels — more than four active
  channels is a resourcing failure, not a strategy.
- Don't use budget tier as the only filter. A $50k/mo budget on the wrong channel is worse than
  a $5k/mo budget on the right one.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` invoked and active brand loaded before any channel was scored?
- All four CMF dimensions scored for every candidate channel (not just the winners)?
- One Primary Bet named, not a tie; Supporting channels capped at three?
- Deprioritized channels explicitly named with one-line rationale each?
- Stage alignment verified — no Pre-PMF brand assigned a brand-awareness or PR primary bet?
- Unit economics checked against offer ACV — no channel recommended where CAC ceiling is
  structurally incompatible?
- Green flags and kill criteria stated for each shortlisted channel (observable, not vague)?
- First 30-day action is specific (platform, budget, targeting approach), not a category?
- Output saved to `./channel-strategy/[slug]-channel-strategy.md`?
