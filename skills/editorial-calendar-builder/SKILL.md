---
name: editorial-calendar-builder
description: >
  Turns content goals, keyword clusters, and a publishing cadence into a filled-out monthly (or
  quarterly) editorial calendar that a team actually ships against — every slot a real topic with a
  format, a target keyword + intent, a funnel stage, an owner, a due-date-before-publish-date, and a
  status. Built on the CADENCE method: it reconciles ambition against real capacity, balances a
  deliberate content mix across funnel stages and formats, sequences clusters so internal links exist
  before they're needed, and pre-loads recurring and seasonal beats — so the calendar survives contact
  with a real week instead of becoming an abandoned spreadsheet. It does NOT manage brand context
  itself — it calls the `brand-brain` skill for ICP, positioning, and offer, and pulls clusters from
  the keyword/cluster skills and per-article plans from the brief skill rather than re-deriving them.
  Use whenever the user says "editorial calendar," "content calendar," "build my publishing schedule,"
  "plan next month's content," "what should we publish and when," "map my clusters to a calendar,"
  "content cadence," "fill out the calendar," or hands over content goals + a list of topics/clusters
  and a posting frequency. It schedules and assigns — it does not write the articles or the briefs.
---

# Editorial Calendar Builder

Give it goals, clusters, and a cadence; get back a calendar a team will actually execute. Not a wish-list of titles, but a real schedule: each slot has a topic, a format, a target keyword and intent, a funnel stage, a named owner, a due date that precedes the publish date, and a status — sequenced so the pieces that need internal links are published *after* the pages they link to. Every topic is filtered through the brand's real ICP, positioning, and offer, because brand context comes from the shared `brand-brain` skill, not from guessing.

This skill schedules and assigns. It does not write the articles, build the per-article briefs, or do the keyword research — it consumes those upstream and names exactly which downstream skill picks up each slot. A calendar that doesn't match the team's real capacity is just a spreadsheet that gets abandoned in week two; this one is built to be kept.

---

## Skills this calls

- **`brand-brain`** (required) — resolves and loads the active brand's ICP + awareness tendency, positioning, offer/destinations, and voice. Topic relevance, funnel-stage balance, and which clusters earn slots all key off this. Do not re-implement brand resolution/scanning here.
- *(optional, when installed / when the calendar naturally needs them)*
  - **`keyword-research-clustering-suite`** — the source of the scored, intent-tagged clusters this calendar sequences; call it (or ask for the export) if no clusters were handed over.
  - **`topic-cluster-pillar-architect`** — for the pillar→spoke dependency graph that drives publish *order* (pillars and link-target spokes ship first).
  - **`content-brief-builder`** / **`serp-research`** — the downstream owner of each slot; name it per row, don't run it. The calendar produces the schedule; the brief produces the plan.
  - **`seo-content-health-decay-audit`** / **`content-refresh-briefer`** — to fold *refresh* slots (not just net-new) into the calendar from real decay data.
  - **`cta-variant-generator`** — already downstream of the brief; referenced only so bottom-funnel slots carry an offer note.

Synthesize a column inline only when a needed source is unavailable — and say what you assumed.

---

## How a run works

```
Step 0  Load the brand     ──► call the `brand-brain` skill (it bootstraps on first use)
Step 1  Gather the inputs  ──► goals + clusters + cadence + capacity (+ recurring beats, seasonality)
Step 2  Build the calendar ──► run the CADENCE method, slot by slot
Step 3  Self-review against the checklist, then present and offer to persist
```

### Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the request and any named brand. It returns the digest — ICP + awareness tendency, positioning line, offer mechanics + destinations, voice + banned words — and the `brand.md` path. The mix and prioritization key off the ICP and the brand's funnel; topic titles obey voice + banned-words. **Do not build the calendar until brand-brain returns.**

**Fallback if `brand-brain` is not installed:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user to install `brand-brain` or answer a 3-question mini-setup (what it is · ICP + funnel shape · primary offer/destination), then proceed.

### Step 1 — Gather the inputs

Before scheduling anything, you need four things. If any are missing, ask — do not invent them:

| Input | What it sets | If missing |
|---|---|---|
| **Goals** | the mix (traffic vs. pipeline vs. retention vs. authority) | ask for the quarter's #1 content goal; default to a balanced mix |
| **Clusters / topic supply** | what fills the slots | call `keyword-research-clustering-suite` or ask for the cluster list / export |
| **Cadence** | slots per period (e.g. 8 posts/month) | ask for target frequency *and* the period (month/quarter) |
| **Real capacity** | whether the cadence is honest | ask: how many writers, what throughput, what's the realistic weekly output? |

Also capture, if available: the team's named owners, recurring beats (product changelog, newsletter, monthly roundup), seasonal/launch dates, and any existing published pages (so new pieces can link *back* to them).

---

## The CADENCE method

A calendar fails when ambition outruns capacity, the mix drifts to whatever's easy to write, or pieces get scheduled before the pages they depend on exist. CADENCE forces each of those decisions, in order — earlier letters constrain later ones.

| Letter | The decision | Output |
|---|---|---|
| **C — Capacity reconcile** | how many slots can this team *actually* fill? | a defensible slot count (≤ stated cadence) + the gap, named |
| **A — Allocate the mix** | what blend of funnel stage, format, and goal? | a target % split mapped to slot counts |
| **D — Dependency sequence** | what must publish *before* what? | a publish order where link-targets and pillars come first |
| **E — Each slot specced** | what is every row, concretely? | topic · format · keyword+intent · funnel stage · owner |
| **N — Now-dated** | when is each due and live? | due date (before publish date) + buffer + status |
| **C — Cushion & recurring** | what fixed beats and slack absorb reality? | recurring rows pre-placed + ~15–20% buffer slots |
| **E — Evaluate** | does it hold up? | the quality-check pass (below) before you present |

### C — Capacity reconcile (do this first, it's the one juniors skip)

Take the requested cadence and pressure-test it against the stated throughput. A solo writer does not ship 12 researched 2,000-word posts a month. **Schedule to real capacity, then name the gap** — "you asked for 12/month; one writer realistically ships 6 long-form or ~10 if half are short — I've built 8 (5 long + 3 short); here's what closing the gap would take." A calendar built to fantasy capacity is the one that gets abandoned. If capacity is unknown, assume conservative defaults and flag the assumption.

### A — Allocate the mix (deliberate, not whatever's easiest)

Set a target blend *before* filling slots, anchored to the brand's #1 goal and ICP funnel:

- **By funnel stage** — TOFU (awareness/traffic) · MOFU (consideration/comparison) · BOFU (decision/conversion). A traffic goal skews TOFU; a pipeline goal needs real MOFU/BOFU weight. Don't let the calendar drift to 90% TOFU because awareness topics are easier to write.
- **By format** — pillar, how-to/guide, listicle, comparison, case study, opinion/POV, news/reaction, refresh. Vary it; ten how-tos in a row bores the audience and the writer.
- **By type** — net-new vs. **refresh** (pull decay candidates via `seo-content-health-decay-audit`). A healthy calendar reserves ~15–30% for refreshing existing winners — usually higher ROI than net-new.

State the split as percentages, then convert to slot counts. Example default for a balanced 8-slot month: 50% TOFU / 30% MOFU / 20% BOFU; ~25% refresh; no format more than ~40% of slots.

### D — Dependency sequence (publish order, not just volume)

Topical authority and internal linking only work if the link *targets* exist when the linking page goes live. Sequence accordingly:

- **Pillars before spokes** where the pillar is the hub the cluster links up to.
- **Link-target spokes before the spokes that cite them** — if post B's outline links to post A, A publishes first.
- **BOFU pages live before the TOFU pieces that funnel to them** — don't drive awareness traffic to a comparison page that isn't published yet.
- Pull the dependency graph from `topic-cluster-pillar-architect` when present; otherwise infer it from the briefs' internal-link maps and note the inference.

Encode dependencies as a `Depends on` note per row, and order publish dates to honor them.

### E — Each slot specced (make it executable)

Every row is a complete hand-off, not a title:

```
| Topic (working title) | Format | Target KW · Intent | Funnel | Cluster | Owner | Next step |
```

- **Working title** in the brand's voice (banned-words obeyed) — final headline is the Headline skill's job downstream.
- **Target keyword + intent** from the cluster map (informational / commercial / transactional). One primary KW per slot; no cannibalizing two slots onto the same head term.
- **Funnel stage** so the mix stays honest as it fills.
- **Owner** — a real name or role. An unowned row doesn't ship.
- **Next step** — name the downstream skill (`serp-research` → `content-brief-builder` → drafting), so the calendar is the front of a pipeline, not a dead list.

### N — Now-dated (due before live, with a buffer)

Each slot gets a **publish date** *and* a **due date that precedes it** with realistic lead time (research → brief → draft → review → SEO → publish). Spread publish dates across the period (don't cluster four on the 30th). Stagger by owner so no one has two due the same day. Default statuses: `Idea → Briefed → Drafting → Review → Scheduled → Published`. Start every net-new slot at `Idea`.

### C — Cushion & recurring (survive a real week)

- **Pre-place recurring beats** the team already owes (newsletter, changelog post, monthly roundup) so they don't silently eat capacity.
- **Anchor seasonal/launch dates** — work backward from a launch so the supporting content lands *before* it, not after.
- **Reserve ~15–20% buffer slots** (or leave them flex) for reactive/news pieces and slippage. A 100%-packed calendar has no room for the week reality always sends.

### E — Evaluate

Run the quality checklist before presenting. If capacity, mix, or dependencies don't hold, fix them now — a beautiful calendar that can't be executed is the failure mode this skill exists to prevent.

---

## Output shape

```
# Editorial Calendar — [period]   ·   Brand: [slug, via brand-brain]
Goal: [#1 content goal]  ·  Cadence: [requested → built, with the gap named]
Mix: TOFU __% / MOFU __% / BOFU __%  ·  Refresh __%  ·  Capacity note: […]

## Calendar
| # | Publish date | Due date | Topic (working title) | Format | Target KW · Intent | Funnel | Cluster | Owner | Status | Depends on | Next step |
|---|---|---|---|---|---|---|---|---|---|---|---|
…

## Recurring & seasonal beats
[pre-placed rows: newsletter, changelog, launch-support, etc.]

## Buffer / flex slots
[the ~15–20% reserved for reactive work]

## Notes
capacity gap & what closes it · sequencing rationale · [verify] dates · what was assumed
```

Keep it scannable enough to paste into a sheet/Notion and complete enough that no slot needs a second conversation to start.

---

## Persistence

If the user wants it saved, write the calendar to `./calendars/[slug]-[period].md` (create `./calendars/` if needed) — **never** inside the skill folder, and never touch `brand.md`. Offer it; don't assume. A saved calendar is the artifact the brief and drafting skills read next. On request, also emit a CSV (or a column layout) the user can import straight into Sheets/Notion/Airtable.

---

## Principles

- **Brand-brain first.** No calendar before brand-brain returns; its ICP, positioning, and voice steer the mix and the titles.
- **Capacity is the gate.** Schedule to what the team can ship and name the gap — never to fantasy throughput.
- **The mix is deliberate.** Funnel balance and format variety are decided up front, not whatever's easiest to write.
- **Sequence by dependency.** Link targets and pillars publish before the pages that need them; BOFU exists before TOFU points at it.
- **Every slot is owned and dated.** An unowned row or a publish date with no due date doesn't ship.
- **Refresh is content too.** Reserve real slots for updating winners — often higher ROI than net-new.
- **Schedule, don't write.** This skill assigns and dates; briefing and drafting are downstream skills.
- **Truth only.** Real dates, real clusters, real owners, or `[verify]`. Never invent capacity, volume, or proof.

## What not to do

- Don't build a calendar before `brand-brain` returns the active brand.
- Don't re-implement brand resolution/scanning/interviewing, or the keyword research/clustering — call the owning skills.
- Don't schedule to the requested cadence when capacity can't sustain it; don't hide the gap.
- Don't fill 90% of slots with TOFU because it's easy; don't repeat one format down the whole month.
- Don't schedule a piece before the page it links to, or drive traffic to an unpublished destination.
- Don't leave slots unowned or undated, or stack every publish on one day.
- Don't write the articles, finalize headlines, or build the per-article briefs here — name the downstream skill instead.
- Don't invent keyword volumes, owners, or launch dates; mark unknowns `[verify]`.

## Quality checklist (self-review before presenting)

- `brand-brain` called and the active brand loaded (or bootstrapped) before anything else?
- Built to **real capacity**, with the gap vs. the requested cadence named explicitly?
- Mix stated as a target split (funnel stage + format + net-new/refresh) and the slots actually match it?
- Publish order honors dependencies (pillars/link-targets/BOFU first), with a `Depends on` note where it matters?
- Every slot complete: working title (on-voice, banned-words obeyed) · format · primary KW + intent · funnel stage · owner · next step?
- Every slot has a publish date *and* an earlier due date with realistic lead time; dates spread across the period, not clustered?
- Recurring beats pre-placed, seasonal/launch dates anchored, and ~15–20% buffer reserved?
- Refresh slots included (from real decay data when available)?
- Voice + banned-words honored; unknown dates/volumes/owners marked `[verify]`; persistence to `./calendars/` offered (never the skill folder, never `brand.md`)?
