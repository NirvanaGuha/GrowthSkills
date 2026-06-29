---
name: social-content-calendar-builder
description: >
  Turns content themes, active campaigns, and a posting frequency into a ready-to-execute
  month-long social media calendar — one row per post, with date, platform, format, topic/angle,
  copy status (stub / draft / approved), and the sibling skill that writes the post. Built on our
  working PLATFORM-THEME-CADENCE (PTC) model: it maps each platform's native format and algorithm
  reward to the brand's content pillars, syncs campaign windows and evergreen slots without
  starving either, and lands on a realistic weekly rhythm the team can actually keep. It does NOT
  manage brand context — it calls `brand-brain` for voice, ICP, offer/destinations, banned words,
  and proof; and names downstream writing skills (e.g. `content-repurposer-atomizer`,
  `headline-hook-generator`, `cta-variant-generator`) per row rather than running them, so the
  calendar is a coordination layer, not a writing task that slows everything down.
  Use whenever the user says "social calendar," "build my posting schedule," "what should I post
  this month," "content calendar for social," "map my campaigns to social," "social media plan,"
  "posting frequency," "schedule my social content," or hands over themes + platforms + a cadence.
  Produces a grid, not copy. If copy is needed, the grid tells you which skill to invoke next.
---

# Social Content Calendar Builder

Give it themes, platforms, and a frequency; get back a month of social posts that are actually scheduled. Not a mood board of ideas — a grid with a date, a platform, a format, an angle, and a copy status for every slot, synced to your campaign windows, balanced across your content pillars, and capped at a cadence your team will hold past week two.

Brand voice, ICP, offer, and banned words come from `brand-brain`. The calendar consumes those — it does not re-derive them. Post copy comes from the writing skills named in each row — the calendar does not write it.

---

## Skills this calls

- **`brand-brain`** (required) — voice adjectives, banned words, ICP + awareness tendency, offer mechanics + destination URLs, positioning, and real proof. Every slot is filtered and labeled through this context.
- *(optional, downstream per row — call them after the calendar is approved)*
  - **`content-repurposer-atomizer`** — when a long-form asset seeds multiple social slots (atomize once, distribute across the grid).
  - **`headline-hook-generator`** — for hook copy on LinkedIn, X, and video scripts; call per slot or in batch once the calendar is locked.
  - **`cta-variant-generator`** — for the action prompt in bottom-funnel or campaign slots.
  - **`proof-vault`** — to pull real stats/quotes for social proof slots; never invent.
  - **`editorial-style-guide`** — mechanical rules (hashtag policy, product-name casing, emoji policy) applied before the writing skills run.
  - **`short-form-video-script-writer`** — for Reels/TikTok/Shorts slots flagged as video.

Name the applicable skill in the "Next skill" column of each row. Do not run them now.

---

## How a run works

```
Step 0  Load the brand      ──► call the `brand-brain` skill (it bootstraps on first use)
Step 1  Gather inputs       ──► platforms, themes, campaign windows, posting frequency, team capacity
Step 2  Build the PTC grid  ──► pillar allocation → platform-format matrix → sequence + date slots
Step 3  Self-review + present, offer to persist
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`). It returns the active brand's digest: voice adjectives, banned words, ICP + awareness tendency, offer mechanics + destination URLs, real proof, positioning. Content pillar relevance, tone, and which offer angles appear in which slots all key off the ICP and positioning. **Do not build the calendar until brand-brain returns.**

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if none, ask for a 3-question mini-setup (what the brand does · ICP + awareness · primary offer + 2-3 voice adjectives), then proceed. Always prefer the call.

### Step 1 — Gather the inputs

Ask for anything missing before scheduling. Do not invent or assume.

| Input | What it sets | If missing |
|---|---|---|
| Platforms (e.g. LinkedIn, X, Instagram, Facebook, Threads) | Which rows exist; format constraints per platform | Ask — one platform is enough to start |
| Content themes / pillars (3–6 buckets) | The topic column for every slot | Ask; or infer 3 from ICP + offer + positioning and confirm |
| Active campaign windows (dates + campaign name) | Campaign-priority slots, promo copy status | Ask; if none, calendar is evergreen-only |
| Posting frequency per platform (posts/week) | Total slot count for the month | Ask; default to 3×/week if unclear, flag it |
| Team capacity (solo vs. team, any blackout dates) | Realistic total slot ceiling | Ask if frequency seems aggressive; skip if not offered |

If the user hands over a brief or a content brief already, read it for themes and campaigns before asking.

---

## The PTC model — Platform-Theme-Cadence (our working framework)

### P — Platform × Format matrix

Each platform rewards a specific native format. Assign formats before assigning topics.

| Platform | Top-performing native formats | Frequency ceiling (sustainable solo) | ICP awareness fit |
|---|---|---|---|
| LinkedIn | Long-form text post, carousel PDF, poll, short video | 1–2×/day (post, not story) | Solution-aware → Product-aware |
| X / Twitter | Single observation, thread, reply-farming anchor | 3–5×/day (but 1 anchor/day for brands) | Unaware → Problem-aware |
| Instagram | Carousel, Reel, static graphic, Stories | 1×/day feed + daily Stories | Unaware → Solution-aware |
| Facebook | Link post, native video, event, long-form text | 1×/day | Problem-aware → Product-aware |
| Threads | Short text, observation, engagement bait | 2–3×/day | Unaware → Problem-aware |
| TikTok | Short-form video (hook in 3s) | 1–2×/day | Unaware |
| YouTube Shorts | 60s vertical video | 3–5×/week | Unaware → Problem-aware |

Pick the formats that match the brand's ICP awareness stage (from brand-brain). Don't assign video if the team has no video capacity — flag the mismatch.

### T — Theme allocation (the Content Mix Rule)

Allocate slots across pillars using a deliberate mix. A default starting split (~35/25/20/15/5 — a common rule-of-thumb, not a sourced benchmark `[verify]`) — adjust by brand's funnel shape:

| Pillar type | Target share | What goes here |
|---|---|---|
| Educate / problem-aware | ~35% | How-tos, tips, explainers, data, myth-busting |
| Engage / community | ~25% | Questions, polls, opinions, reposts, conversations |
| Prove / social proof | ~20% | Case studies, testimonials, customer results, press |
| Convert / offer | ~15% | Product demos, offer announcements, comparison posts |
| Culture / brand | ~5% | Behind-the-scenes, team, values |

Skewing too hard on "convert" kills organic reach. Flag any mix where "convert" exceeds 25%. Adjust mix if the brand has a long-cycle B2B ICP (lean educate) vs. impulse B2C (lean proof + convert).

### C — Cadence: sequence and slot assignment

1. **Lock campaign slots first.** Mark campaign windows in the grid (start, peak, end). Assign "convert" and "prove" formats to peak days; "educate" before the campaign; "engage" after.
2. **Distribute themes across the week** — avoid two consecutive "convert" posts on the same platform. Alternate pillars.
3. **Apply the Single Platform Principle:** for the first version of the calendar, one post per platform per day maximum (avoids burnout and looks less desperate).
4. **Pre-assign recurring beats:** weekly tip (Monday), proof/win (Wednesday), engagement question (Friday) — adjust to brand rhythm.
5. **Flag thin slots** — if a date has no campaign angle and no live content asset to repurpose, mark it "evergreen/create" in copy status so the writing queue is visible.

---

## Calendar grid format

Output the calendar as a markdown table. One row per post.

```
| Date | Platform | Format | Pillar | Angle / Topic | Copy status | Next skill |
```

- **Date**: actual calendar date (e.g. Jun 23)
- **Platform**: slug (li, x, ig, fb, th, tt, yt)
- **Format**: text, carousel, video, poll, story, reel, thread
- **Pillar**: educate / engage / prove / convert / culture
- **Angle / Topic**: 5–12 words describing the specific post angle (not a vague theme)
- **Copy status**: stub (idea only) / draft (copy exists, needs review) / approved (ready to schedule)
- **Next skill**: the sibling skill that produces the copy (or "none" if copy is already approved)

All copy status at calendar-build time defaults to "stub" unless the user hands over existing drafts.

---

## Month-view summary (above the grid)

Before the full grid, present a summary block:

```
Brand:          [slug, via brand-brain]
Period:         [Month + year]
Platforms:      [list]
Total posts:    [N] across [M] platforms
Weekly cadence: [N]×/wk on [platform] …
Campaign slots: [campaign name · dates · peak day]
Pillar mix:     Educate NN% · Engage NN% · Prove NN% · Convert NN% · Culture NN%
Flags:          [any mismatch — over-indexed pillar, video capacity gap, thin week, etc.]
```

Flag anything before the grid. The user should see problems before they approve the schedule.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No calendar rows before `brand-brain` returns. Its ICP, voice, and banned words shape every slot.
- **Formats before topics.** Assign the native format per platform before picking the angle. Misfitting formats tank reach.
- **Mix discipline.** "Convert" slots above 25% on any platform is a red flag — say so.
- **Realistic cadence over ideal cadence.** A 3×/week schedule held beats a 2×/day schedule abandoned in week two.
- **Name the next skill, don't run it.** This skill builds the schedule. Writing happens downstream.
- **Real proof only.** If a "prove" slot has no confirmed proof, mark it `[verify]` — don't fabricate a testimonial to fill the slot.
- **Campaign sync is mandatory.** A social calendar that doesn't know your live campaigns is a distraction engine.

## What Not to Do

- Don't build the calendar before `brand-brain` returns the active brand.
- Don't write post copy — the calendar produces angles and assigns writing skills, not finished posts.
- Don't reimplement brand scanning or interview here. That lives in `brand-brain`.
- Don't generate 30 "convert" posts because the user asked for a promo calendar — flag the mix and rebalance.
- Don't invent proof, testimonials, or stats to fill "prove" slots — mark them `[verify]` and name `proof-vault`.
- Don't assign video formats without confirming the team can produce them.
- Don't produce a calendar for a brand/platform the user hasn't confirmed — ask, don't assume.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or bootstrapped) before any slot was written?
- Month-view summary present above the grid, with flags?
- Platform × format match is native to each platform?
- Pillar mix within guidelines (convert ≤ 25%)? Imbalances flagged?
- Campaign windows locked and synced (campaign slots at peak, educate slots before)?
- Each slot has a specific angle (not a vague theme), a copy status, and a named next skill?
- Cadence is realistic for the team's stated capacity?
- No invented proof in "prove" slots (real or `[verify]`)?
- Offered to save the calendar to `./social/[brand-slug]-[month]-calendar.md`?
