---
name: pop-up-sticky-bar-widget-builder
description: >
  Produces self-contained, paste-ready HTML/JS/CSS widgets for the two highest-ROI on-page
  conversion surfaces: modal pop-ups and sticky bars. Inputs are an offer (what you're promoting),
  trigger conditions (exit-intent, scroll %, time-on-page, URL match, session count), and targeting
  rules (device, cookie state, segment). Output is a single embeddable snippet — real working code,
  not a wireframe — with the trigger logic wired in, brand styles applied, and a matching
  data-layer push for GA4 event tracking. Handles the full combinatorial matrix: newsletter capture,
  lead-gen, cart-abandonment, discount reveal, content upgrade, announcement bar, cookie consent
  teaser, and mobile-safe sticky CTA. Does NOT require a third-party pop-up SaaS to run; the widget
  drops straight into any site that can accept a `<script>` tag or GTM Custom HTML tag. Use when the
  user says "build me a pop-up," "exit-intent widget," "sticky bar," "discount pop-up," "email
  capture overlay," "scroll-triggered modal," "announcement banner," "embed a lead form," or hands
  over an offer and asks for a conversion widget.
---

# Pop-up & Sticky-Bar Widget Builder

Give it an offer and trigger rules, get a working widget. Every output is a self-contained
HTML/JS/CSS snippet — no SaaS dependency, no third-party cookie, no page-speed penalty from a
bloated platform SDK. Drop it into a GTM Custom HTML tag, a WordPress theme footer, or a Webflow
embed block and it runs.

This skill builds the widget. It does not set strategy on which offer to run, write the email
sequence the widget feeds, or manage the post-capture follow-up. Those jobs belong to
`cta-variant-generator`, `lead-nurture-drip-builder`, and `notification-opt-in-prompt-optimizer`
respectively — call them when you need the surrounding program.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — resolves voice, colors, fonts, banned words, and offer
  mechanics. The widget's copy, palette, and CTA are derived from the returned digest.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
  brand's `brand.md` directly; if none exists, ask the user for brand hex colors, button label, and
  banned words before proceeding.
- **`cta-variant-generator`** — when the user hasn't supplied the overlay headline or button label,
  call this to generate on-brand, awareness-appropriate CTA copy rather than writing freehand.
- **`form-friction-auditor`** — when the widget includes a form with more than one visible field,
  call this to pressure-test field count and order before finalising the markup.
- **`email-compliance-auditor-gdpr-can-spam`** — when the widget collects email addresses,
  invoke to confirm opt-in language, unsubscribe hook, and consent mechanics are compliant for
  the user's sending region.
- **`consent-privacy-compliance-auditor`** *(optional)* — invoke when the widget uses cookies or
  localStorage for suppression/frequency capping and the brand operates under GDPR/CCPA.
- **`notification-opt-in-prompt-optimizer`** *(optional)* — when the widget's goal is a push-
  notification opt-in rather than email capture, delegate copy + timing to this skill.
- **`tracking-plan-taxonomy-builder-auditor`** *(optional)* — when the user has an existing GA4
  event taxonomy, pass it here to verify the `dataLayer.push` events emitted by the widget
  conform to their naming conventions before shipping.

---

## How a run works

```
Step 0  Load brand        ──► call brand-brain; extract palette, voice, banned words, offer
Step 1  Scope the widget  ──► surface type × trigger × goal (quick questions if ambiguous)
Step 2  Copy gate         ──► call cta-variant-generator if headline/button not supplied
Step 3  Compliance gate   ──► call email-compliance or consent auditor if email/cookie is in scope
Step 4  Build             ──► produce the widget (HTML/CSS/JS block; see framework below)
Step 5  Wire tracking     ──► append dataLayer.push events for GA4
Step 6  Delivery          ──► inline snippet + GTM paste-in guide + suppression knobs to adjust
```

---

## Funnel-moment targeting (our working model — the opinionated layer)

This is a house checklist, not an established framework. It borrows three stages from Dave
McClure's Pirate Metrics / AARRR funnel (which has five: Acquisition, Activation, Retention,
Referral, Revenue) and applies only the three that on-page widgets actually serve. Every widget
decision maps to one of those three moments — that determines which surface type, which trigger,
and which offer wins:

| Moment | Surface | Primary trigger | Offer that converts |
|---|---|---|---|
| **Acquisition** (Awareness → first action) | Modal | Exit-intent on landing / blog | Lead magnet, discount, free trial |
| **Activation** (first value moment) | Modal or inline bar | Scroll 60 %+ on product / pricing page | Demo CTA, onboarding nudge, risk-reducer |
| **Retention** (return visit) | Sticky bar | Time ≥ 30 s or ≥ 2 page views this session | Loyalty offer, upsell, new feature alert |

If the user has not specified a moment, ask one question — "What should the visitor do right after
closing or submitting this widget?" — and derive the moment from the answer.

---

## Widget surfaces

### Modal pop-up

The default output is a centered lightbox overlay with a backdrop. Structure:

```html
<!-- [brand-slug]-popup — generated by pop-up-sticky-bar-widget-builder -->
<div id="pe-popup-overlay" style="display:none; ...backdrop styles...">
  <div id="pe-popup-box" style="...box styles from brand palette...">
    <button id="pe-popup-close">✕</button>
    <!-- headline, body, form or CTA -->
  </div>
</div>
<script>
(function () {
  // Suppression check (localStorage key pe-popup-[slug]-seen)
  // Trigger logic (exit-intent | scroll | time)
  // Show / hide helpers
  // dataLayer.push events (popup_shown, popup_submitted, popup_dismissed)
})();
</script>
```

Styles are inlined on the elements (not a separate `<style>` block) so the snippet is safe inside
GTM Custom HTML without a `<head>` context. All positioning is `position:fixed; z-index:99999` to
survive stacking-context conflicts.

### Sticky bar

A full-width bar pinned to top or bottom of viewport. Uses the same suppression and event
scaffolding. When the page scrolls past 80 %, the bar fades in (if set to scroll-triggered);
otherwise it persists on load.

```html
<!-- [brand-slug]-stickybar — generated by pop-up-sticky-bar-widget-builder -->
<div id="pe-sticky-bar" style="position:fixed; bottom:0; left:0; width:100%; ...">
  <!-- short copy + CTA button -->
  <button id="pe-sticky-close">✕</button>
</div>
```

---

## Trigger logic reference (real JS patterns shipped in output)

| Trigger | Implementation shipped |
|---|---|
| **Exit-intent (desktop)** | `document.addEventListener('mouseleave', handler)` — fires once when cursor leaves viewport top edge |
| **Exit-intent (mobile)** | `window.addEventListener('pagehide', handler)` — mobile has no mouseleave; pagehide is the nearest proxy |
| **Scroll %** | `window.addEventListener('scroll', () => { if (scrolled > threshold && !shown) show() })` |
| **Time on page** | `setTimeout(show, ms)` wrapped in a `DOMContentLoaded` listener |
| **Session count** | `sessionStorage.getItem('pe-visit-count')` incremented on load; trigger fires at configured count |
| **URL match** | `if (window.location.pathname.match(pattern)) { ... }` guard around the entire init block |

All triggers are OR-composable: pass multiple to fire on whichever comes first.

---

## Suppression knobs (always included in output)

The widget ships with a localStorage suppression key so it never fires again after submission or
dismissal (configurable TTL in days). The code block at the top of the `<script>` is clearly
labelled with the config object the user adjusts:

```js
var CONFIG = {
  slug:        'brand-slug',         // namespace for localStorage
  suppressDays: 30,                  // 0 = suppress for session only
  showAfterMs:  3000,                // time trigger in ms
  scrollPct:    60,                  // scroll % trigger (0–100)
  exitIntent:   true,                // desktop exit-intent on/off
  mobileExit:   true                 // mobile pagehide on/off
};
```

---

## GA4 dataLayer events (always wired in)

Every widget emits three events without additional configuration:

| Event name | Fired when | Key parameters |
|---|---|---|
| `popup_shown` or `sticky_shown` | Widget becomes visible | `widget_slug`, `trigger_type`, `offer_type` |
| `popup_submitted` or `sticky_submitted` | Form submitted / CTA clicked | `widget_slug`, `email_captured` (bool) |
| `popup_dismissed` or `sticky_dismissed` | Close button or backdrop click | `widget_slug`, `seconds_visible` |

If the brand has an existing GA4 taxonomy (surfaced by `tracking-plan-taxonomy-builder-auditor`),
conform event names to their schema instead of the defaults above.

---

## Output format

The delivered artifact is always three sections:

1. **The widget snippet** — one pasteable block, self-contained, clearly commented. Every
   configurable value lives in the `CONFIG` object at the top, not scattered through the code.
2. **GTM paste-in instructions** — five bullets: tag type → Custom HTML, trigger → All Pages
   (or URL-matched Page View), paste location, firing rules note, and one-line test instruction.
3. **Suppression + A/B note** — how to reset suppression during testing, and how to split-test
   two variants by serving each to a 50 % random cookie split (pattern included inline).

Save to `./widgets/[brand-slug]-[surface]-[offer-slug].html` when the user says to persist.

---

## Principles (Non-Negotiable)

- **Brand-brain first.** No markup before `brand-brain` returns. Its palette and banned words
  override everything here.
- **One trigger wins.** If multiple triggers are configured, the first one to fire takes it; the
  widget never shows twice in a session.
- **Suppression is not optional.** Every widget ships with suppression logic. A widget that re-fires
  on every page load is a dark pattern, not a growth tool.
- **Mobile-safe by default.** Modals get `max-width:90vw; max-height:85vh; overflow-y:auto`. Sticky
  bars get `padding-bottom: env(safe-area-inset-bottom)` for iOS notch safety.
- **No external dependencies in the snippet.** No CDN fetch, no third-party pixel, no jQuery
  requirement. Pure vanilla JS. The widget must work in a GTM Custom HTML tag with script tag
  support disabled for external domains.
- **Real code, not pseudocode.** The output runs on paste. If a value is unknown (e.g., the form
  action endpoint), mark it `/* TODO: replace */` — but the surrounding logic is complete.

---

## What not to do

- Don't design a widget without calling `brand-brain` first — off-brand hex colors and banned-word
  violations are the most common reason widgets get rejected by design review.
- Don't ship a modal that lacks a visible close button — WCAG 2.1 SC 2.1.2 (keyboard trap) and
  basic usability both require an escape.
- Don't skip the suppression key — a pop-up that re-fires on every visit trains users to ignore it
  and tanks the brand's sender reputation downstream when the captured emails are low-quality.
- Don't put form fields inside a sticky bar — sticky bars are for single-click actions only; use a
  modal for anything requiring input beyond one field.
- Don't generate the widget before calling `email-compliance-auditor-gdpr-can-spam` when the form
  collects email in a GDPR or CCPA jurisdiction — an unconsented email capture is a liability, not
  a lead.
- Don't use `!important` on every rule — scope specificity with the widget's ID wrapper instead.

---

## Quality checklist (self-review before presenting)

- `brand-brain` called and palette / voice / banned words applied to all copy and color values?
- `cta-variant-generator` called (or user-supplied headline + button label used verbatim)?
- Trigger logic is real JS (not pseudocode), fires correctly, and is OR-composable?
- Suppression key namespaced to brand slug, TTL configurable in the `CONFIG` object?
- Mobile-safe sizing and iOS safe-area padding present?
- All three `dataLayer.push` events wired (shown, submitted, dismissed) with correct parameter names?
- GTM paste-in instructions included (5 bullets)?
- Email collection? → `email-compliance-auditor-gdpr-can-spam` called or flagged as required?
- Cookie/localStorage suppression in a GDPR region? → `consent-privacy-compliance-auditor` flagged?
- Output saved to `./widgets/[brand-slug]-[surface]-[offer-slug].html` if persistence requested?
- Widget runs on paste with no external dependencies?
