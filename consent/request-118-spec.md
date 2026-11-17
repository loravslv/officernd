# Request #118: cookie consent rebuilt the Design System 3.0 way

Lora 17 Nov 11:56: "make the cookie consent use the same approach as in design system 3".

## What DS3 does now (read from designsystem-stanvision.webflow.io, 17 Nov)
- Consent Mode v2 with **categories**: necessary (always on), preferences, analytics, marketing. Each category maps to its Consent Mode signals.
- Banner `#cookie-banner` has two views in one card: **summary** (message + Customize / Reject all / Accept all) and **settings** (one row per category with a switch + Save choices / Reject all / Accept all).
- Footer "Cookie settings" (`#cookie-settings`) opens the settings view. Turning a category off deletes its first-party cookies and reloads the page so already-running tags stop.
- Stored choice v2 `{ preferences, analytics, marketing, version: 2, t }`; old v1 granted/denied is migrated, nobody is asked twice.
- `consent_update` dataLayer event with per-category values. HubSpot gets `setHubSpotConsent` + `revokeCookieConsent`.
- Layout and switch looks are Webflow classes (`cookie__*`), only position/transition stay in the embed.

## What OfficeRnD has (older DS3 copy from 22 Oct)
All-or-nothing Accept / Reject, a close (x) "acknowledged" state outside the EU, no categories, no settings view. Footer link and Cookie Policy link already correct. GTM_ID still empty (pending client), no other consent-gated scripts on the site (checked /, /glossary, /contact-us, /coworking-software, /blog, /pricing/flex, /hybrid-work-software).

## Changes (owner applies)

### 1. Site settings > Custom code > Head
Replace only the first block (the `<!-- Consent ... -->` comment + its `<script>`) with `head-consent-v2.html` in this folder. Theme pre-paint and Motion switch scripts stay untouched.
Kept from OfficeRnD's version (not in DS3): staging test switches `?consent_region=eu|row|auto` and `?gtm=1` kept for the tab. Added vs DS3: an EU-timezone visitor whose IP is outside the region list also gets the denied update.

### 2. Global CSS & Elements (86b51892), embed js__consent (a73fc444)
Replace the whole content with `js__consent-v2.html`. Same as DS3 plus: switches may be Webflow Divs (script adds role="switch", tabindex 0, Space/Enter toggle).

### 3. Global CSS & Elements: #cookie-banner structure (keep the existing ids where noted)
```
cookie__message #cookie-banner                       (existing)
  cookie__info-wrapper                               (existing)
    Div cookie__view  [data-consent-view=summary]    NEW
      p-small  "We use cookies for analytics and marketing, only with your consent. Read our [Cookie Policy]."  (existing element 1a58...891a, moved in; Cookie Policy page link kept)
      cookie__btns-wrapper                           (existing 354b..., moved in)
        Button Ghost      "Customize"   link #consent-customize   NEW
        Button Secondary  "Reject all"  link #consent-reject      (existing 2c116f5c, text + link changed)
        Button Primary    "Accept all"  link #consent-accept      (existing 7239a125, text + link changed)
    Div cookie__view is--hidden  [data-consent-view=settings]   NEW
      p-small  "Choose which cookies we can use. You can change this any time under Cookie settings in the footer."
      Div cookie__categories
        Div cookie__category [data-consent-category=necessary]
          Div cookie__category-text
            p-small text-weight-medium  "Necessary"
            p-small text-secondary      "Keep the site secure and working, and remember your cookie choice. Always on."
          Div cookie__switch is--locked [data-consent-switch=necessary] [aria-checked=true] [aria-disabled=true]
            Div cookie__switch-knob
        Div cookie__category [data-consent-category=analytics]
          ... "Analytics" / "Show us how visitors use the site, so we can see what works and fix what doesn't."
          Div cookie__switch [data-consent-switch=analytics] [aria-checked=false] > Div cookie__switch-knob
        Div cookie__category [data-consent-category=marketing]
          ... "Marketing" / "Measure our ads and show you relevant ones on other sites."
          Div cookie__switch [data-consent-switch=marketing] [aria-checked=false] > Div cookie__switch-knob
      cookie__btns-wrapper
        Button Ghost      "Save choices" link #consent-save
        Button Secondary  "Reject all"   link #consent-reject
        Button Primary    "Accept all"   link #consent-accept
```
Button size small on all six (as now). Button looks: OfficeRnD's current Secondary for Reject and Primary for Accept are kept (DS3 uses Primary for both); Customize / Save = Ghost as in DS3.

### 4. New classes (values from DS3, OfficeRnD variables)
- `cookie__view`: flex column, row-gap spacing-block/spacing-24. Combo `is--hidden`: display none.
- `cookie__categories`: flex column, border-top 1px theme gray-mode/200.
- `cookie__category`: flex, justify space-between, align flex-start, column-gap spacing-16, padding-top/bottom spacing-12, border-bottom 1px gray-mode/200.
- `cookie__category-text`: flex column, row-gap spacing-4, flex 1 1 0%.
- `cookie__switch`: flex, justify flex-start, align center, width 2.5rem, height 1.5rem, padding 0.1875rem, margin-top 0.125rem, radius br-pill, bg gray-mode/400, cursor pointer, flex 0 0 auto, transition background-color 0.2s; focus-visible outline 2px solid text-primary, offset 2px. Combo `is--on`: bg button-primary/background, justify flex-end. Combo `is--locked`: bg button-primary/background, opacity 0.5, cursor not-allowed, justify flex-end.
- `cookie__switch-knob`: 1.125rem x 1.125rem, radius br-pill, bg theme background, flex 0 0 auto.

### Tested by injection on staging /glossary (17 Nov, Chrome)
EU mode: summary opens after 600 ms; Customize shows the 3 rows; Space on Analytics turns it on (role switch, label "Analytics"); Save choices stores {analytics: true, others false}, sends consent update analytics granted / ads denied and `consent_update` "custom"; footer Cookie settings reopens the settings view with Analytics on and focus on it; Reject all stores all false, deletes a test `_ga` cookie and reloads. 390 px: card 8 px from the edges, 3 buttons on one row, settings view 463 px tall, no horizontal scroll.
