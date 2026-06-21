---
name: utm-parameter-bulk-builder
description: >
  Takes a campaign name, source/medium/content variants, and a set of destination URLs and outputs
  a complete, consistently-named UTM-tagged link set — one row per URL × variant combination —
  ready to paste into a spreadsheet, load into a link-shortener, or hand to the paid team. The
  single UTM tracking-link generator for the skill library: enforces a shared naming taxonomy,
  catches common tagging mistakes before they pollute attribution data, and produces an audit trail
  so any analyst can decode every parameter six months later. Use when the user says "build my UTMs,"
  "generate tracking links," "UTM spreadsheet," "tag all these URLs," "bulk UTMs," "campaign URLs,"
  "I need links for my launch," or hands over a campaign brief and asks for the tracking asset.
---

# UTM Parameter Bulk Builder

Campaign name and variant list in → complete, clean, attribution-safe UTM link set out. One run produces the full grid so nothing falls through; one taxonomy so every analyst reads the data the same way six months from now.

This skill builds links and enforces naming governance. It does not configure GA4 channel groups, set up a link shortener, or write the campaign copy. If attribution rules downstream require a specific taxonomy already agreed, it enforces that; if not, it installs the Growth-Attribution Standard (below).

---

## Skills this calls

- **`brand-brain`** (required first) — loads the active brand's voice, ICP, offer destinations, and any existing UTM naming conventions stored there. UTM taxonomy often differs by brand (slugs, allowed sources, preferred shortener). Do not implement brand resolution yourself.
- **`campaign-brief-builder`** *(optional, compose)* — if the user has no campaign brief yet, call this first to produce the objective, audience, and channel mix that drives UTM source/medium choices.
- **`campaign-qa-launch-checklist-generator`** *(compose, if installed)* — the launch checklist skill calls this skill to populate its UTM validation section; when running standalone, surface checklist items as a self-review at the end.
- **`data-qa-measurement-gotcha-checker`** *(compose, if installed)* — escalate edge-case attribution anomalies (self-referral risk, iOS Safari ITP stripping, cross-domain journey breaks) to that skill rather than inventing guidance here.

---

## How a run works

```
Step 0  Load the brand       ──► call brand-brain; inherit any existing UTM taxonomy
Step 1  Collect inputs        ──► URLs × sources × mediums × campaign × content variants
Step 2  Enforce taxonomy      ──► Growth-Attribution Standard (or brand override)
Step 3  Build the link grid   ──► one row per combination; validate; flag errors
Step 4  Produce deliverables  ──► TSV/CSV block + audit-trail legend + gotcha notes
Step 5  Self-review           ──► quality checklist before presenting
```

### Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill (Skill tool, `skill: brand-brain`). It returns the active brand's digest and `brand.md`. Look for any existing UTM taxonomy block (`utm_taxonomy`, source allowlist, preferred `utm_campaign` slug format, shortener preference). If present, treat it as the override; document where the brand diverges from the Growth-Attribution Standard below.

**Fallback if `brand-brain` is absent:** read `~/.brandbrain/brands/.active` + that brand's `brand.md`; if neither exists, ask the user to install `brand-brain` or confirm the taxonomy in a 3-question mini-setup (preferred campaign-slug format · source allowlist · shortener if any), then proceed.

### Step 1 — Collect inputs

Gather everything needed before building. Prompt for anything missing:

| Input | Notes |
|---|---|
| Destination URL(s) | One or many; confirm base URL is correct before appending params |
| `utm_campaign` | The campaign name slug — see taxonomy below |
| `utm_source` list | e.g. `google`, `meta`, `newsletter`, `linkedin` |
| `utm_medium` list | e.g. `cpc`, `email`, `social`, `push` |
| `utm_content` variants | Optional; ad creative IDs, audience segment labels, placement keys |
| `utm_term` | Optional; paid search keywords |
| Link-shortener? | Yes/no; if yes, which (Bitly, Short.io, etc.) — skill generates the long form only |

When the user pastes a campaign brief, extract these fields directly. When inputs arrive mid-conversation as a table or CSV, parse and confirm before building.

---

## The Growth-Attribution Standard (naming taxonomy)

Use this when the brand has no existing taxonomy. It maps directly to GA4 default-channel-group rules, so links built here produce clean channel groupings out of the box — no custom-channel-group hacks required.

### Naming rules (non-negotiable)

1. **Lowercase everything.** `google` not `Google`; `cpc` not `CPC`. Mixed case creates duplicate channel rows in GA4.
2. **Hyphens, not spaces or underscores, in campaign names.** `q3-retention-win-back`, not `Q3 retention win-back` or `q3_retention_win-back`. Spaces URL-encode to `+` and break some platforms.
3. **No PII in any parameter.** No email addresses, user IDs, or names as content/term values.
4. **`utm_campaign` is the single source of truth for the initiative.** Format: `[quarter]-[initiative-slug]` or `[launch-name]` — short, stable, human-readable. Never the same as `utm_content`.
5. **`utm_source` must match GA4's recognized source list** (or your CRM's) so sessions classify correctly. Approved sources: `google`, `bing`, `meta`, `linkedin`, `twitter`, `instagram`, `tiktok`, `newsletter`, `pushengage`, `partner-[slug]`, `direct` *(reserved — never set manually)*.
6. **`utm_medium` must match GA4 default channel-group definitions exactly** — else traffic lands in "Unassigned."

### GA4 medium → default channel group mapping

| `utm_medium` | GA4 default channel | Use for |
|---|---|---|
| `cpc` | Paid Search / Paid Social | All paid click-based ads |
| `email` | Email | Any email campaign |
| `push` | Other | Web/app push notifications |
| `social` | Organic Social | Unpaid social posts |
| `affiliate` | Affiliates | Partner / affiliate links |
| `referral` | Referral | Manual referral links (not organic backlinks) |
| `sms` | SMS | Text campaigns |
| `display` | Display | Banner / programmatic |
| `video` | Video | YouTube, TikTok, Reels (paid) |

### `utm_content` conventions

Use `utm_content` to distinguish creative variants, audience buckets, or placements within a single source/medium pairing. Format: `[type]-[descriptor]`, e.g. `img-hero-orange`, `copy-a-painpoint`, `audience-smb-us`. Never duplicate the campaign or medium here.

---

## Building the link grid

Generate a row for every URL × source × medium × content-variant combination. Produce the output as a copy-pasteable TSV block with headers so it drops cleanly into Google Sheets:

```
destination_url  utm_source  utm_medium  utm_campaign  utm_content  utm_term  tagged_url
```

Append the `?` or `&` correctly (check whether the base URL already has query params). Encode spaces as `%20` in the raw URL; note if the user's shortener handles encoding automatically.

Flag any row where:
- `utm_medium` does not match a recognized GA4 medium (risk: "Unassigned")
- `utm_source` is not in the approved list (risk: source-level duplication in reports)
- The tagged URL exceeds 2,083 characters (IE/some proxies truncate)
- `utm_campaign` contains spaces, uppercase, or PII patterns
- Two rows are identical (duplicate tracking)

---

## Audit-trail legend

After the grid, output a compact legend block — one line per unique parameter value explaining what it represents. This is the artifact that saves an analyst's sanity in six months:

```
## UTM Legend — [campaign slug] — [brand] — [date]
utm_campaign: q3-retention-winback  →  Q3 win-back campaign for churned trial users
utm_source: meta                    →  Meta Ads (Facebook + Instagram placements)
utm_medium: cpc                     →  Paid click — maps to Paid Social in GA4
utm_content: img-a-pain             →  Creative A, pain-point angle, static image
```

Save to `./ops/utm-[campaign-slug]-[YYYY-MM-DD].md` when the user wants a persistent file.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No links built before the brand is loaded. An existing taxonomy is an override.
- **Taxonomy over creativity.** Consistent naming beats memorable naming. Enforce the standard even when the user proposes a prettier slug that would break channel grouping.
- **One medium per medium.** Never invent a new `utm_medium` value to solve an attribution problem — that just moves the mess downstream. Flag it; recommend the right fix.
- **No silent assumptions.** If a source is ambiguous (e.g. "paid influencer"), ask before assigning; wrong medium = broken channel data.
- **Audit trail is non-negotiable.** Every bulk output ships with its legend. A link grid without a legend is a future attribution bug.
- **Truth discipline.** If the user's shortener strips UTMs (a known iOS/Safari ITP behavior), say so with `[verify against your shortener's docs]` — don't pretend the link is attribution-safe.

---

## What Not to Do

- Don't implement brand-scanning or taxonomy resolution — call `brand-brain`.
- Don't invent a new `utm_medium` value to make a channel "look right" in reports.
- Don't encode spaces as `+` — use `%20`; `+` is query-string-only and breaks some parsers.
- Don't combine two sources into one row to save space — one row per source, always.
- Don't use `utm_term` for anything other than paid keywords; don't stuff creative info there.
- Don't output only the tagged URLs without the legend — that's a debt you're creating for someone else.
- Don't skip the flag pass — a silent taxonomy error is worse than a loud one.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded; existing taxonomy honored as override?
- All `utm_medium` values match GA4 default-channel-group definitions exactly?
- All `utm_source` values in the approved list or explicitly flagged?
- Every parameter value is lowercase, no spaces, no PII?
- `utm_campaign` is stable, human-readable, consistent across all rows for this initiative?
- Duplicate rows identified and removed?
- Tagged URL length checked; no row exceeds 2,083 characters?
- Audit-trail legend present and covers every unique parameter value used?
- Gotcha notes included for any rows with attribution risk (shortener stripping, cross-domain, ITP)?
- Output is a copy-pasteable TSV block, not prose?
