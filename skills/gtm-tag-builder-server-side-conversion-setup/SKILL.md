---
name: gtm-tag-builder-server-side-conversion-setup
description: >
  Takes a conversion event + destination platform and produces real, ready-to-import GTM container
  JSON with the tag, trigger, and variable spec; GA4 key-event marking steps; Consent Mode v2
  configuration (default + update commands, storage taxonomy); and a step-by-step server-side
  CAPI/Events API setup guide for the chosen destination (Meta CAPI, Google Ads Enhanced
  Conversions, or TikTok Events API). Output is concrete and operational — not advice. The tag
  JSON can be dropped straight into GTM's import flow; the CAPI guide includes the exact endpoint,
  required fields, deduplication key pattern, and a Node.js/Python fetch snippet. Calls brand-brain
  for domain and CMP context before producing anything. Use whenever the user says "set up GTM for
  [event]," "build a GTM tag," "server-side tracking," "CAPI setup," "Enhanced Conversions," "GTM
  container JSON," "Consent Mode v2," "conversion tracking," "sGTM," or "first-party data pipeline."
---

# GTM Tag Builder & Server-Side Conversion Setup

Give it a conversion event and a destination — get a working tag container JSON, the GA4 key-event
config, Consent Mode v2 wiring, and a production-ready server-side CAPI/Events API setup guide.
This skill produces implementation artifacts, not checklists. Everything is ready to import or drop
into code.

---

## Skills this calls

- **`brand-brain`** (required, Step 0) — loads the active brand's domain, CMP/consent setup, and
  any existing martech context so tag configs use the correct measurement ID, pixel ID, and domain.
  Fallback if brand-brain is absent or returns no brand: read `~/.brandbrain/brands/.active` + that
  brand's `brand.md` directly; if none exists, ask the user for their domain, GA4 Measurement ID,
  destination platform (GA4 / Meta / Google Ads / TikTok), and CMP name before proceeding.
- **`tracking-plan-taxonomy-builder-auditor`** — if the user hasn't defined the event schema yet,
  call this skill to confirm the event name, parameters, and data-layer shape before writing the
  tag. Synthesize inline when absent.
- **`data-qa-measurement-gotcha-checker`** — gate every output through a data-quality pass: fire
  conditions, deduplication logic, cross-domain tracking holes, and Consent Mode consent-state
  gotchas. Call it before finalizing; synthesize inline when absent.
- **`consent-privacy-compliance-auditor`** — on any setup that touches EU/UK/CA traffic, pass the
  Consent Mode v2 config through this skill to verify GDPR Article 6 / ePrivacy compliance, storage
  category mapping, and that no events fire on ad_storage=denied without modeling enabled. Review
  and cite; do not rebuild.
- **`utm-campaign-naming-enforcer`** — optionally call to verify the UTM structure feeding the
  `campaign_id` / `campaign_name` fields in the Events API payload is consistent and clean.

---

## How a run works

```
Step 0  Brand context    ──► call brand-brain; get domain, Measurement ID, pixel IDs, CMP
Step 1  Clarify scope    ──► event name, destination(s), sGTM needed? existing container?
Step 2  Tag + trigger    ──► produce GTM container JSON (ready to import)
Step 3  GA4 key-event    ──► steps to mark the event as a key event in GA4 UI + Ads link
Step 4  Consent Mode v2  ──► default-commands block + update-command placement + storage map
Step 5  Server-side      ──► CAPI/Events API guide with endpoint, payload, dedup, code snippet
Step 6  Data-quality gate──► call data-qa-measurement-gotcha-checker; surface any issues
Step 7  Deliver          ──► zip all artifacts under ./gtm/[brand-slug]/[event-slug]/
```

Always complete Step 0 before writing a single line of JSON.

---

## Step 0 — Load the brand (always first)

**Invoke the `brand-brain` skill** (Skill tool, `skill: brand-brain`), passing the user's request
and any named brand. Capture: domain (for `linker` cross-domain config), GA4 Measurement ID, any
Meta Pixel ID / Google Ads Conversion ID, known CMP (OneTrust, Cookiebot, Osano, custom), and
martech stack (sGTM endpoint if already configured). If brand context is missing these fields, ask
only for the ones absent before proceeding.

**Fallback if brand-brain is absent or returns no brand:** read `~/.brandbrain/brands/.active` +
that brand's `brand.md` directly; if none exists, ask the user for their domain, GA4 Measurement
ID, destination platform, and CMP name before proceeding.

---

## Step 1 — Scope clarification (minimal, only if genuinely missing)

Ask in a single batch if any of these are unresolved after brand context loads:

| Question | Why it matters |
|---|---|
| Exact conversion event name (e.g. `purchase`, `generate_lead`) | Tag fire condition + GA4 event name |
| Destination(s): GA4 / Meta CAPI / Google Ads EC / TikTok / all | Determines which JSON configs and CAPI guides to produce |
| Data-layer push or auto-event? | Variable config and trigger type differ |
| Server-side GTM already configured? (sGTM endpoint URL) | Skip sGTM infrastructure setup if yes; include it if no |
| EU/UK/CA traffic? | Triggers consent-privacy-compliance-auditor pass |

---

## Step 2 — GTM container JSON (the Simo Ahava model)

Follow the **Simo Ahava container-structure model**: every implementation uses a clean
Data Layer Variable → Trigger → Tag chain with no overlapping fire conditions.

Produce a valid GTM container import JSON with the following shape (fill with real values from
brand context + scope):

```json
{
  "exportFormatVersion": 2,
  "exportTime": "[ISO-8601 timestamp]",
  "containerVersion": {
    "container": {
      "accountId": "[GTM-ACCOUNT-ID — ask user if absent]",
      "containerId": "[GTM-CONTAINER-ID — ask user if absent]",
      "name": "[brand-slug]-[event-slug]-setup",
      "usageContext": ["WEB"]
    },
    "variable": [
      {
        "name": "DLV - [event_param]",
        "type": "v",
        "parameter": [
          {"type": "INTEGER", "key": "dataLayerVersion", "value": "2"},
          {"type": "BOOLEAN", "key": "setDefaultValue", "value": "false"},
          {"type": "TEMPLATE", "key": "name", "value": "[event_param]"}
        ]
      }
    ],
    "trigger": [
      {
        "name": "CE - [event_name]",
        "type": "CUSTOM_EVENT",
        "customEventFilter": [
          {
            "type": "EQUALS",
            "parameter": [
              {"type": "TEMPLATE", "key": "arg0", "value": "{{_event}}"},
              {"type": "TEMPLATE", "key": "arg1", "value": "[event_name]"}
            ]
          }
        ]
      }
    ],
    "tag": [
      {
        "name": "GA4 - [event_name]",
        "type": "gaawe",
        "parameter": [
          {"type": "TEMPLATE", "key": "measurementId", "value": "[GA4_MEASUREMENT_ID]"},
          {"type": "TEMPLATE", "key": "eventName", "value": "[event_name]"},
          {"type": "LIST", "key": "eventParameters", "list": [
            {"type": "MAP", "map": [
              {"type": "TEMPLATE", "key": "name", "value": "[param_key]"},
              {"type": "TEMPLATE", "key": "value", "value": "{{DLV - [param_key]}}"}
            ]}
          ]}
        ],
        "firingTriggerId": ["[trigger-id-reference]"],
        "tagFiringOption": "ONCE_PER_EVENT",
        "consentSettings": {
          "consentStatus": "NEEDED",
          "consentType": [{"type": "TEMPLATE", "value": "analytics_storage"}]
        }
      }
    ]
  }
}
```

**Naming convention (enforced):** `DLV - [param]` for Data Layer Variables; `CE - [event]` for
Custom Event triggers; `GA4 - [event]` / `META - [event]` / `GADS - [event]` for tags.
Keep every tag at exactly one firing trigger. No sequence tags unless explicitly requested.

Produce a separate tag object per destination. Output the full JSON; do not abbreviate.

---

## Step 3 — GA4 key-event marking

After tag JSON, provide exact UI steps:

1. GA4 Admin → Events → mark `[event_name]` as key event (toggle).
2. If Google Ads is linked: Google Ads → Tools → Conversions → import from GA4 → select
   `[event_name]` → set category, value, count-per-click vs. count-per-conversion.
3. Verify in GA4 DebugView that the event fires with the expected parameters before marking live.
4. Flag: key-event marking propagates within 24 hours; imported Ads conversions may take up to 3
   days to appear in campaign reporting [verify current SLA in GA4 help].

---

## Step 4 — Consent Mode v2 (always include for web)

Use the **Google Consent Mode v2 specification** (default + update pattern).

**Default command block** — fires on every page, before any tag:

```html
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  // Default: deny non-essential storage until user signals
  gtag('consent', 'default', {
    'analytics_storage':     'denied',
    'ad_storage':            'denied',
    'ad_user_data':          'denied',
    'ad_personalization':    'denied',
    'functionality_storage': 'granted',
    'security_storage':      'granted',
    'wait_for_update':       500
  });
</script>
```

Place this block **before** the GTM snippet in `<head>`. Never after.

**Update command** — fires from the CMP callback when the user grants or updates consent:

```javascript
// Example: OneTrust OptanonWrapper callback
function OptanonWrapper() {
  gtag('consent', 'update', {
    'analytics_storage':  OnetrustActiveGroups.includes('C0002') ? 'granted' : 'denied',
    'ad_storage':         OnetrustActiveGroups.includes('C0004') ? 'granted' : 'denied',
    'ad_user_data':       OnetrustActiveGroups.includes('C0004') ? 'granted' : 'denied',
    'ad_personalization': OnetrustActiveGroups.includes('C0004') ? 'granted' : 'denied'
  });
}
```

Adapt the category IDs to the brand's CMP. If CMP is unknown, note the pattern and flag it.

**Storage taxonomy map:**

| Storage type | Meaning | Typical consent category |
|---|---|---|
| `analytics_storage` | GA4 cookies / measurement | Analytics / Performance |
| `ad_storage` | Ad platform cookies | Advertising / Targeting |
| `ad_user_data` | Sending user data to ad platforms | Advertising / Targeting |
| `ad_personalization` | Personalized ad targeting | Advertising / Targeting |
| `functionality_storage` | UX-preserving cookies (language, cart) | Functional |
| `security_storage` | Fraud/security cookies | Strictly Necessary (always granted) |

**Behavioral modeling:** if the brand runs Google Ads, enable Consent Mode behavioral modeling in
Google Ads account settings to recover modeled conversions from denied-consent users [verify this
setting is available in the account tier before advising].

---

## Step 5 — Server-side CAPI / Events API setup

Produce a destination-specific guide. Default to Meta CAPI if the user hasn't specified.

### Meta Conversions API (CAPI)

**Architecture:** browser pixel → sGTM (or direct server call) → Meta CAPI endpoint.
Deduplication key: `event_id` must match between browser pixel and CAPI call exactly.

**Endpoint:** `POST https://graph.facebook.com/v19.0/[PIXEL_ID]/events?access_token=[TOKEN]`
[verify current API version at developers.facebook.com/docs/marketing-api/conversions-api]

**Minimum required payload:**

```json
{
  "data": [{
    "event_name": "[event_name]",
    "event_time": 1700000000,
    "event_id": "[uuid-matching-browser-pixel]",
    "action_source": "website",
    "event_source_url": "https://[domain]/[page]",
    "user_data": {
      "em": "[SHA-256-hashed-lowercase-email]",
      "ph": "[SHA-256-hashed-e164-phone]",
      "client_ip_address": "[ip]",
      "client_user_agent": "[ua]",
      "fbp": "[_fbp cookie value]",
      "fbc": "[_fbc cookie value]"
    },
    "custom_data": {
      "currency": "USD",
      "value": "[order_total]"
    }
  }],
  "test_event_code": "[TEST_CODE — remove before production]"
}
```

Hash PII client-side before sending; never log raw PII. `event_id` must be a UUID generated once
per event and passed to both the browser pixel (`fbq('track', ..., {eventID: uuid})`) and the CAPI
call.

**Node.js snippet (fetch):**

```javascript
const { bizSdk } = require('facebook-nodejs-business-sdk');
// Or raw fetch — use official SDK for signature handling in production
const response = await fetch(
  `https://graph.facebook.com/v19.0/${PIXEL_ID}/events?access_token=${TOKEN}`,
  {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify({ data: [payload] })
  }
);
const result = await response.json();
if (result.error) console.error('CAPI error:', result.error);
```

**Via sGTM:** configure the Meta CAPI tag in sGTM (Stape or native), map `event_id` from the
incoming GA4 client request, and enable "Mirror to browser" only if the browser pixel is already
firing (to avoid duplication without dedup).

### Google Ads Enhanced Conversions (EC)

**Approach:** send hashed first-party user data (email, phone, name, address) alongside the
standard conversion ping so Google can match to signed-in users.

**GTM tag type:** `Google Ads Conversion Tracking` with Enhanced Conversions enabled.
Set the `enhanced_conversions` parameter in the tag to map to the hashed user data object in
the data layer. Hash SHA-256 + lowercase + trim before pushing to the data layer.

**Data layer push example (at checkout confirmation):**

```javascript
dataLayer.push({
  event: 'purchase',
  transaction_id: '[order_id]',
  value: [total],
  currency: 'USD',
  enhanced_conversions: {
    email: '[sha256(lowercase(trim(email)))]',
    phone_number: '[sha256(e164_format(phone))]'
  }
});
```

### TikTok Events API

**Endpoint:** `POST https://business-api.tiktok.com/open_api/v1.3/event/track/`
Required headers: `Access-Token: [TOKEN]`, `Content-Type: application/json`.
Deduplication: set `event_id` to the same UUID used in the TikTok Pixel `ttq.track()` call.
[verify current endpoint version at ads.tiktok.com/marketing_api/docs]

---

## Step 6 — Data-quality gate

Before finalizing, call **`data-qa-measurement-gotcha-checker`** (or synthesize inline if absent)
and verify:

- [ ] Fire condition is event-specific (not all-pages); no duplicate fires possible
- [ ] `event_id` deduplication key is consistent between browser and server calls
- [ ] Cross-domain linker configured if the brand has multiple domains
- [ ] Consent Mode default-command fires before GTM snippet (not after)
- [ ] `ad_storage=denied` path: events still fire but without cookies; modeling is enabled [verify]
- [ ] sGTM container is on a first-party subdomain (e.g. `metrics.domain.com`), not the GTM CDN
- [ ] Test event code is removed before production deployment
- [ ] PII hashing is done client-side; raw PII never hits the data layer as a logged field

Flag any gaps. Do not suppress findings.

---

## Step 7 — Artifact delivery

Save all outputs under `./gtm/[brand-slug]/[event-slug]/`:

```
./gtm/[brand-slug]/[event-slug]/
  container-import.json        ← drop into GTM Import
  consent-mode-v2-snippet.html ← paste into <head> before GTM snippet
  capi-setup-guide.md          ← destination-specific guide + code snippets
  data-quality-gate.md         ← checklist output from Step 6
```

Confirm file paths in the response. Keep all inline as well so the user can copy without opening
files.

---

## Principles (Non-Negotiable)

- **Brand context before JSON.** No container JSON before `brand-brain` returns Measurement ID /
  pixel ID and domain. Wrong IDs in production are costly to unwind.
- **Real artifacts, not advice.** Every output must be importable or executable — not a description
  of what to do.
- **Simo Ahava container model.** Clean DLV → Trigger → Tag chain; one trigger per tag; no
  overlapping fire conditions.
- **Deduplication is mandatory for any server-side setup.** `event_id` must be a stable UUID
  generated once, passed to both browser and server in the same event lifecycle.
- **Consent Mode default fires first.** The default-deny block always precedes the GTM snippet.
  No exceptions.
- **Hash PII before it touches the data layer.** SHA-256 + normalize (lowercase + trim for email;
  E.164 for phone). Never log raw PII.
- **`[verify]` over invention.** API version numbers, SLA claims, and platform-specific limits are
  marked `[verify]` when not confirmed in this context.

---

## What Not to Do

- Don't produce container JSON without the brand's actual Measurement ID and pixel ID — placeholder
  IDs in production silently lose conversions.
- Don't skip the Consent Mode default-command block for any web implementation, regardless of
  geo — it is a prerequisite for behavioral modeling globally.
- Don't use GTM sequence tags to chain conversion tags; model each destination as its own tag with
  its own trigger.
- Don't set `tagFiringOption: ONCE_PER_PAGE` for purchase or lead events — use `ONCE_PER_EVENT`
  to allow multiple conversions per session (e.g., multi-item carts).
- Don't conflate client-side Pixel and server-side CAPI as alternatives — both should fire, with
  deduplication, not one replacing the other.
- Don't invent API endpoints or version numbers; mark them `[verify]` and link the canonical docs.
- Don't call this skill a "tracking audit" — it builds and ships implementation, not analysis.

---

## Quality Checklist (self-review before presenting)

- `brand-brain` called and Measurement ID / pixel ID / CMP loaded (or fallback inputs collected)?
- Container JSON is syntactically valid and uses the correct GA4 event type (`gaawe`)?
- Every tag has `consentSettings` wired to the correct storage type?
- Consent Mode default-command block is complete with all six storage types and `wait_for_update`?
- sGTM guide includes deduplication key pattern and code snippet for the chosen destination?
- `data-qa-measurement-gotcha-checker` pass completed; all findings surfaced?
- PII hashing instructions included for any destination that requires user data?
- All artifacts saved to `./gtm/[brand-slug]/[event-slug]/` with confirmed paths?
- No real API version numbers or SLA claims stated without `[verify]`?
