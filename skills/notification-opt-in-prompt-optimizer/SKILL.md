---
name: notification-opt-in-prompt-optimizer
description: >
  Web or app opt-in surface → optimized permission-prompt timing, two-step opt-in copy,
  and trigger placement recommendations. Applies PRIME-ASK-REDUCE, our working model: a native
  browser/OS permission dialog is a one-shot, unrepeatable ask — this skill engineers the
  pre-prompt (the custom "primer" screen), the placement trigger (page, event, scroll depth,
  session depth), and the friction-reducer microcopy so the user arrives at the native dialog
  already primed to accept. Covers web push (Chrome/Firefox/Safari), iOS app push (UNUserNotification),
  Android 13+ POST_NOTIFICATIONS, and in-app notification opt-ins. Produces: a placement trigger
  spec, primer copy variants (headline + body + CTA pair), native-dialog simulation copy (where
  configurable), and a post-denial recovery path. Composes brand voice from `brand-brain`, push
  copy from `push-notification-copy-generator`, CTA sharpness from `cta-variant-generator`, and
  routes the finished copy to `lifecycle-email-push-copy-reviewer` for a final audit pass.
  Use when the user says "improve push opt-in rate," "write permission prompt," "opt-in copy,"
  "notification permission ask," "primer screen," "permission prompt timing," "two-step opt-in,"
  "push subscription rate low," or "how do I ask for notification permission."
---

# Notification Opt-In Prompt Optimizer

The browser's native permission dialog is a one-shot event. Once a user clicks "Block," you cannot re-ask without an explicit browser settings change — the door closes. This skill exists to make sure you earn that click before the dialog appears, not during it.

The operating model is **PRIME-ASK-REDUCE**: prime intent with a custom UI before the browser/OS fires; ask at the right moment (page, event, scroll depth, session); reduce friction with copy that names the exact value, not a generic "stay updated." Every artifact is on-brand because brand context comes from `brand-brain`, not from guessing here.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads voice, banned words, ICP, and proof. Opt-in copy does not implement brand resolution; that lives in `brand-brain`, once.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for (1) product name and what notifications they send, (2) ICP + what value the notifications deliver, (3) 3 voice adjectives + banned words, before proceeding.
- **`push-notification-copy-generator`** — drafts the example notification copy shown in the primer screen ("Here's what you'll get"); call it with the brand + notification type to produce a concrete preview.
- **`cta-variant-generator`** — sharpens the primer's action button label (the "Yes, allow" equivalent) across value and commitment angles; call for 3–5 label variants then recommend one.
- **`lifecycle-email-push-copy-reviewer`** — final audit pass on the primer copy and recovery-path message for voice, CTA strength, and character-limit compliance; call before presenting output.
- **`in-app-microcopy-writer-auditor`** *(optional)* — if the opt-in surface is an in-app modal, call for accessibility and microcopy compliance pass.

---

## How a run works

```
Step 0  Load brand         ──► brand-brain (always first)
Step 1  Triage surface     ──► web push | iOS push | Android push | in-app opt-in
Step 2  Audit current ask  ──► placement, primer presence, copy, denial handling
Step 3  Set trigger spec   ──► PRIME trigger rules (page + event + depth)
Step 4  Write primer copy  ──► headline / body / CTA (+ push-notification-copy-generator preview)
Step 5  CTA sharpening     ──► cta-variant-generator → recommend one label
Step 6  Recovery path      ──► post-denial re-engagement copy + settings deep-link text
Step 7  Reviewer pass      ──► lifecycle-email-push-copy-reviewer
Step 8  Deliver artifact   ──► inline or saved to ./opt-in/[brand-slug]-opt-in-spec.md
```

---

## Step 0 — Load the brand (always first)

Invoke the `brand-brain` skill before writing a single word of copy. It returns the active brand's digest: voice adjectives, banned words, offer, ICP + awareness tendency, real proof, and destination URLs.

Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that brand's `brand.md` directly; if none exists, ask the user for (1) product name and what notifications they send, (2) ICP + what value the notifications deliver, (3) 3 voice adjectives + banned words, before proceeding.

---

## PRIME-ASK-REDUCE (our working model)

### PRIME — build intent before the native dialog fires

The primer is a fully custom UI element (modal, banner, slide-in, full-screen sheet) that appears **before** the browser or OS fires its native dialog. It does three things:

1. **Names the exact value** — "Get back-in-stock alerts before they sell out" beats "Allow notifications."
2. **Shows a concrete example** — embed one real notification preview (call `push-notification-copy-generator` for this).
3. **Sets a low-commitment micro-CTA** — "Yes, show me" not "Subscribe now." The primer CTA fires the native dialog; it is not the ask itself.

A primer that describes generic "updates" is indistinguishable from spam consent. A primer that names a specific, high-value signal converts.

### ASK — trigger placement rules

The native dialog fires immediately after the user taps the primer's affirmative CTA. The trigger *for the primer* is the load-bearing decision. Use the **Permission Trigger Matrix**:

| Context | Earliest sane trigger | Recommended trigger |
|---|---|---|
| eCommerce / product page | After 30 s dwell OR scroll 60 % | Product page, post add-to-cart intent signal |
| SaaS / app | After first successful action (activated moment) | Post-aha-moment; never on load |
| Media / blog | Second pageview in session OR 3 min read time | Article finish (scroll 85 %) |
| eCommerce checkout flow | Post-order-confirm (thank-you page) | Thank-you page, order confirmed |
| App (iOS/Android) | After feature use OR onboarding complete | Contextual: trigger on the feature that benefits from push |

Never fire the primer on first page load, in the first 10 seconds, or inside an existing modal stack (stacking modals guarantees dismissal). Never fire on a page the user arrived at via an external ad (they haven't chosen you yet).

**Session depth floor:** require at least 1 prior session or 2 pageviews before triggering, except on high-intent pages (checkout confirm, post-sign-up, post-purchase).

### REDUCE — friction-reduction copy principles

Three copy levers that reduce opt-in friction:

1. **Specificity over abstraction.** "Get notified when [Product X] is back in stock" outperforms "Stay updated." Name the trigger.
2. **Micro-commitment softening.** If the ICP is privacy-sensitive or awareness is low, add a one-line frequency anchor: "We send 2–3 per week, you can turn off anytime."
3. **Risk-reversal closer.** The last line of the primer body or the microcopy below the CTA should be a zero-risk closer drawn from the brand's real proof: free to turn off, no personal data sold, etc. Mark any unconfirmed claim `[verify]`.

---

## Primer copy structure

```
PRIMER SCREEN
─────────────────────────────────────────
[Brand icon or notification bell icon]

Headline   (≤ 60 chars)
Body copy  (≤ 120 chars; name the value + example content)
[Example notification preview — from push-notification-copy-generator]
Microcopy  (≤ 60 chars; frequency + zero-risk)

[ Primary CTA ]   [ Not now / No thanks ]
─────────────────────────────────────────
↓ Primary CTA fires native dialog
```

**Headline patterns that work:**
- Benefit-led: "Never miss a [specific event]" / "Be first when [trigger]"
- Specificity: "Get [Trigger Name] alerts — before everyone else"
- Curiosity: "We'll only ping you when it actually matters"

**Avoid:** "Allow notifications" as the headline (that's the browser's job), "Stay connected," "Subscribe for updates," "We'd like to send you notifications" (passive, brand-centric, low-conversion).

---

## Post-denial recovery path

After a user clicks "Block" on the native dialog (or "No thanks" on the primer), the standard rule is: **do not re-ask.** Instead:

1. **In-session soft recovery:** on the next high-intent action (e.g., second add-to-cart, checkout), surface a non-modal, single-line prompt with a settings deep-link. Copy: "Missed notification alerts? [Enable in browser settings →]"
2. **Email fallback:** if you have their email, the first post-session message can mention push as an alternative channel. Copy pattern: "Prefer email? You're already set. To also get [benefit] via browser alerts: [Enable here →]"
3. **Settings deep-link text (web push):** "To turn on alerts: open Chrome → Settings → Privacy → Notifications → [site] → Allow." Keep this scannable; a wall of instructions is ignored.

Do not show the primer again in the same session after a "Block." Waiting 30+ days and triggering on a new high-intent surface is acceptable; daily retry is dark-pattern territory.

---

## Platform-specific constraints

| Platform | Native dialog | Configurable copy | Retry rules |
|---|---|---|---|
| Chrome / Edge (web push) | "Allow" / "Block" only; no custom body | Primer only | One auto-prompt per origin; re-ask requires user gesture |
| Firefox (web push) | Same as Chrome | Primer only | Blocked = permanent until user resets |
| Safari (web push, macOS 13+) | "Allow" / "Deny" | No primer support pre-dialog | Permission state persists in Keychain |
| iOS app (UNUserNotification) | System alert with app name only | Primer screen before `requestAuthorization()` | One system prompt per app install; subsequent requests need `UNUserNotificationCenter` settings link |
| Android 13+ (POST_NOTIFICATIONS) | System permission dialog | Primer + `shouldShowRequestPermissionRationale()` rationale UI | Up to 2 system prompts per install (first decline is soft; second decline is permanent) |

Flag any mis-match between the user's stated platform and these rules.

---

## Output format

```markdown
## Notification Opt-In Spec — [Brand slug]
Surface: [web push | iOS push | Android push | in-app]
Brand: [slug, via brand-brain]
ICP awareness stage: [from brand-brain]

### Trigger placement recommendation
[page / event / depth / session-floor + rationale]

### Primer screen variants
Variant A — [angle: value-led / specificity / curiosity]
  Headline: ...
  Body: ...
  Notification preview: [from push-notification-copy-generator]
  Microcopy: ...
  CTA (primary): ... | CTA (dismiss): ...

Variant B — [different angle]
  ...

Recommended: Variant [X] because [one line].

### CTA label options
[from cta-variant-generator — 3–5 options, one recommended]

### Post-denial recovery
In-session: ...
Email fallback: ...
Settings deep-link text: ...

### Platform constraint notes
[any flags]

### Reviewer pass
[lifecycle-email-push-copy-reviewer verdict — pass / flags]
```

Save to `./opt-in/[brand-slug]-opt-in-spec.md` when the user asks; inline otherwise.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No copy before `brand-brain` returns. Voice + banned words override everything here.
- **One-shot awareness.** Treat the native dialog as irreversible; every decision in this skill is made to protect that shot.
- **Name the value, not the channel.** "Get back-in-stock alerts" not "Enable notifications."
- **Specific beats generic, always.** A concrete notification preview in the primer doubles as social proof of what the channel delivers.
- **No dark patterns.** No fake urgency, no pre-checked consent, no buried dismiss option, no re-prompting after a hard Block.
- **Honest proof only.** Frequency claims and zero-risk closers must be verifiable; unconfirmed: `[verify]`.
- **Platform rules are hard limits.** If the user's ask conflicts with browser/OS constraints, flag it before writing copy.

## What Not to Do

- Don't fire the primer on first page load or in the first 10 seconds.
- Don't re-ask in the same session after a user clicks Block or dismisses.
- Don't use "Stay connected," "Stay updated," or "We'd like to notify you" as the headline.
- Don't implement brand scanning/interviewing/storage — call `brand-brain`.
- Don't write push notification body copy from scratch — call `push-notification-copy-generator` for the preview.
- Don't invent proof or frequency claims; mark unconfirmed data `[verify]`.
- Don't stack the primer on top of another modal.

## Quality Checklist (self-review before presenting)

- `brand-brain` called and active brand loaded (or bootstrapped) before any copy written?
- Voice + banned words honored; only real proof used (rest `[verify]`)?
- Trigger placement rule follows Permission Trigger Matrix and session-depth floor?
- Primer has: specific value headline (≤60 chars), body (≤120 chars), concrete notification preview (from `push-notification-copy-generator`), microcopy risk-reducer, CTA pair?
- At least 2 primer variants on different angles; one recommended with rationale?
- CTA labels sharpened via `cta-variant-generator`?
- Post-denial recovery path covers in-session, email fallback, and settings deep-link?
- Platform constraints checked; conflicts flagged?
- `lifecycle-email-push-copy-reviewer` pass completed before presenting?
