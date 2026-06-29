---
name: bulk-scheduling-csv-builder
description: >
  Takes a content batch — approved social posts, push notifications, or a mixed set — and produces
  a platform-correct bulk-upload CSV ready to import directly into Buffer, Hootsuite, or Later.
  Handles per-network character limits, image-URL columns, link-in-bio logic (Later), first-comment
  scheduling (Instagram, LinkedIn), hashtag placement rules, timezone normalization, and required
  vs. optional column schemas for each tool. Works from a paste, a content calendar doc, or the
  output of sibling skills (social-content-calendar-builder, push-notification-copy-generator,
  instagram-caption-reel-script-writer, tiktok-script-hook-generator, x-thread-writer,
  linkedin-post-writer). Validates every row against the tool's import spec before delivering —
  no "failed row" surprises on upload. Use when the user says "schedule these posts," "bulk upload,"
  "make the CSV," "export to Buffer/Hootsuite/Later," "schedule the content calendar," or hands
  over a batch and names a scheduling tool.
---

# Bulk Scheduling CSV Builder

Turn an approved content batch into an importable CSV in one pass — field names, column order, character limits, first-comment logic, timezone offsets, and image column formatting matched exactly to Buffer, Hootsuite, or Later's own import spec. No manual reformatting, no failed rows on upload.

This skill formats and validates. It does not write new copy, redesign posts, or re-sequence a content strategy. If the copy is weak or off-brand, say so and call the right sibling — but don't rewrite inline.

---

## Skills this calls

- **`brand-brain`** (required) — active brand's timezone, default hashtag policy, banned words, and channel-level voice rules. Every row inherits these before the user sees the CSV. **Fallback if `brand-brain` is absent or returns no brand:** read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for (1) primary timezone, (2) hashtag placement policy, (3) banned words before continuing — never build the CSV on an undefined timezone.
- *(compose, when available)* **`social-content-calendar-builder`** — if the user wants the calendar built first, call this skill before building the CSV. If output is already provided, skip.
- *(compose, when available)* **`push-notification-copy-generator`**, **`instagram-caption-reel-script-writer`**, **`tiktok-script-hook-generator`**, **`x-thread-writer`**, **`linkedin-post-writer`** — call the relevant one if copy for a network is missing and the user wants it drafted.
- *(compose, when available)* **`utm-parameter-bulk-builder`** — if tracked URLs are needed in the link columns, call this to generate UTM-tagged URLs before building the CSV.
- *(review, when available)* **`post-quality-reviewer-voice-auditor`** — optional review pass before export; flag, don't block.

---

## How a run works

```
Step 0  Load brand context      ──► call brand-brain; extract timezone, hashtag policy, voice rules
Step 1  Identify tool + networks ──► Buffer | Hootsuite | Later; which networks are in the batch
Step 2  Ingest the content batch ──► paste / calendar doc / sibling-skill output
Step 3  Map to column schema     ──► apply our Column Authority table for each tool (see below)
Step 4  Validate every row       ──► char limits, required fields, image-URL format, date/time
Step 5  Emit the CSV + a summary ──► save to ./social/[brand-slug]-bulk-[tool]-[YYYY-MM-DD].csv
Step 6  Report anomalies         ──► flag rows that need user action; never silently truncate copy
```

---

## Column Authority — our working name for the per-tool import spec

### Buffer

Required columns: `Date`, `Time`, `Message`, `Link` (optional), `Photo` (optional — direct image URL).
- **Date:** `MM/DD/YYYY`. **Time:** 12-hour `HH:MM AM/PM` in the brand's primary timezone (note the timezone label in the summary row).
- `Message` is the full post body. Buffer splits network at import via its connected profiles — one row per post, not one row per network. If the user has separate Buffer profiles per network, create separate CSVs or tabs and flag it.
- Instagram first comment: Buffer supports a `First Comment` column in Pro/Agency plans — include it when hashtags are pushed to the first comment per brand policy.
- Character ceiling enforcement: X/Twitter 280 (URLs count as 23 [verify]); LinkedIn 3,000; Facebook 63,206; Instagram 2,200. Truncate at the word boundary, append "…", mark the row `[TRUNCATED — review]` in the summary.

### Hootsuite

Required columns: `Date`, `Time`, `Message`, `Network` (exact strings: `Twitter`, `Facebook`, `Instagram`, `LinkedIn`, `Pinterest`, `TikTok`), `Profile`.
- **Date:** `YYYY-MM-DD`. **Time:** 24-hour `HH:MM` in UTC (Hootsuite normalizes on import — convert from brand timezone; show the offset in the summary).
- `Profile` = the Hootsuite social profile name (ask user if unknown; mark `[NEEDS PROFILE NAME]`).
- `Attachments` column: comma-separated direct image/video URLs. Max 4 images or 1 video per row for most networks.
- No native first-comment column — if brand policy pushes hashtags to first comment on Instagram, note this as a manual post-upload step in the summary.
- One row = one network post. If the same content goes to three networks, output three rows with identical timestamps (or staggered by 2 min to avoid platform overlap — ask the user).

### Later

Required columns: `Caption`, `Scheduled Time`, `Media URL`.
- **Scheduled Time:** ISO 8601 `YYYY-MM-DDTHH:MM:SS` in the brand's timezone; Later accepts account timezone on import.
- `Media URL` is mandatory — Later is media-first. If no image URL is provided, mark the row `[MISSING MEDIA — cannot import]` and surface it in the summary. Do not silently drop the row.
- `First Comment` column supported for Instagram — use it for hashtag push per brand policy.
- Link-in-bio logic: Later's link-in-bio maps each post to a landing URL. Populate the `Caption Link` column (Later-specific) with the destination URL when provided; otherwise leave blank and note in the summary that the link-in-bio slot will be unset.
- TikTok via Later: `Caption` max 2,200 chars; `Hashtags` in a separate column (Later splits them). If hashtags are embedded in the caption, extract and move them.

---

## Timezone Normalization Protocol

1. Read the brand's primary timezone from `brand-brain` (e.g., `America/New_York`).
2. If the batch has explicit send times, convert to the tool's required format (12h, 24h, or ISO) and append the IANA timezone label in the summary.
3. If the batch has only relative times ("9 AM Monday"), resolve against today's date (use the session date) and note the resolved dates in the summary.
4. If timezone is unknown, ask before building the CSV — a silent UTC assumption corrupts every send time.

---

## Hashtag & First-Comment Logic

| Network | Placement rule | First-comment support |
|---|---|---|
| Instagram | Brand policy from `brand-brain`; if policy = "first comment," move all `#tags` out of Caption into `First Comment` column | Buffer (Pro+), Later — yes; Hootsuite — manual |
| LinkedIn | In-copy hashtags only (≤3–5 [verify]) — no first-comment scheduling in any major tool | N/A |
| X / Twitter | In-copy, count toward char limit | N/A |
| Facebook | In-copy or omit — algorithm does not reward hashtag stuffing | N/A |
| TikTok | Later splits caption from hashtags — keep separate from body copy | N/A |

---

## Validation Checklist (per row, before emit)

- Required columns for the chosen tool are populated (no blank required cell).
- Character count within network ceiling; truncated rows flagged in summary, not silently cut.
- Date/time format matches the tool's exact spec; timezone resolved.
- Image URL is a direct link (ends in `.jpg`, `.png`, `.mp4`, etc.) not a redirect or a CDN preview page.
- For Later: every row has a `Media URL`; rows missing media are surfaced, not dropped.
- For Hootsuite: `Network` values match the exact accepted strings.
- Banned words from `brand-brain` are not present; if found, flag the row and suggest a fix.
- UTM parameters present on all links if the user requested tracking.

---

## Output format

**Primary artifact:** `./social/[brand-slug]-bulk-[tool]-[YYYY-MM-DD].csv`

**Summary block (inline, after the CSV):**

```
## Bulk CSV Summary — [Tool] — [Date]
Brand: [slug] | Timezone: [IANA zone] | Networks: [list]
Rows: [n total] | [n clean] clean | [n flagged] flagged

Flags:
- Row 4 [TRUNCATED] Instagram caption at 2,200 chars — review "…" ending
- Row 9 [MISSING MEDIA] No image URL for Later import — add before uploading
- Row 12 [NEEDS PROFILE NAME] Hootsuite profile column blank
- Row 15 [BANNED WORD] "revolutionary" found — brand policy disallows; suggested: "proven"

Manual steps required post-upload:
- Hootsuite Instagram rows: push hashtags to first comment manually after scheduling
- Later: confirm link-in-bio slots for rows 2, 6, 11 (Caption Link column populated)
```

---

## Principles

- **Column fidelity above all.** The CSV is worthless if the tool rejects it. Match the import spec exactly — no invented columns, no re-ordered required fields.
- **Never silently truncate or drop.** Every alteration is flagged. The user decides whether to shorten copy or delay the post.
- **Brand-brain first.** Timezone, hashtag policy, and banned words come from the brand, not from defaults.
- **Compose, don't rewrite.** If copy is missing, call the right sibling skill. If copy is present but weak, flag it — don't rewrite inline.
- **One tool per CSV.** Buffer, Hootsuite, and Later have different schemas. Never mix them in one file.
- **Timezone is not optional.** Always confirm timezone before resolving relative times.

## What Not to Do

- Don't guess at profile names for Hootsuite — mark the cell `[NEEDS PROFILE NAME]` and surface it.
- Don't embed image files — only direct URLs in image columns.
- Don't push hashtags into the LinkedIn caption beyond the brand's stated limit.
- Don't silently adapt Later rows that are missing media — flag them; Later will reject media-free rows.
- Don't reimplement brand resolution — call `brand-brain`.
- Don't produce a CSV before `brand-brain` has confirmed the active brand's timezone.

## Quality Checklist

- `brand-brain` called; timezone, hashtag policy, and banned words loaded?
- Tool identified (Buffer / Hootsuite / Later); correct column schema applied?
- Every required column populated; no blank required cells?
- Character limits enforced per network; truncations flagged in summary, not silent?
- Image URLs are direct links; Later rows missing media flagged (not dropped)?
- Hootsuite `Network` values match exact accepted strings?
- Summary block lists total rows, clean count, all flags, and any required manual post-upload steps?
- CSV saved to `./social/[brand-slug]-bulk-[tool]-[YYYY-MM-DD].csv`?
