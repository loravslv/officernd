# OfficeRnD Website Migration Plan — WordPress → Webflow

**Property in scope:** `www.officernd.com` (the marketing website)
**Owner of this document:** Marketing / Web lead (to be confirmed — see §7)
**Version:** 1.0 (draft for stakeholder review)
**Last updated:** 2026-07-20

---

## 0. Резюме на български (Executive summary)

Този документ е пълен план за миграция на маркетинг сайта `www.officernd.com` от **WordPress** към **Webflow**. Той не е само списък със страници — обхваща рискове, зависимости, отговорници (RACI), начини за проверка, времеви график, известни неизвестни и решенията, които трябва да бъдат одобрени, преди да започне работа.

Планът покрива 11 задължителни теми: (1) пълен sitemap и URL инвентар; (2) план за 301 пренасочвания; (3) запазване на SEO; (4) съдържание и CMS мапинг; (5) одит на скриптове и проследяване; (6) форми и интеграции; (7) достъп и собственост; (8) поверителност и сигурност; (9) технически и дизайн изисквания; (10) миграция и QA; (11) пускане и възстановяване.

**Важно ограничение при изготвянето:** средата, в която беше изготвен този план, няма мрежов достъп до `officernd.com` (egress policy блокира хоста), затова структурата по-долу е реконструирана от публично достъпна информация и трябва да бъде потвърдена с реален краул (Screaming Frog + Google Search Console + WordPress export) като **първа задача** от изпълнението. Всяко място, което изисква потвърждение, е маркирано с 🔴 **VERIFY**.

Придружаващи работни таблици (в папка `/templates`): URL инвентар, redirect карта, одит на скриптове, инвентар на форми, RACI за достъп.

> Документът по-долу е на английски, защото се изпълнява съвместно с агенция/инструменти, чийто работен език е английски. При желание мога да преведа целия документ на български.

---

## Table of contents

1. [Complete sitemap & URL inventory](#1-complete-sitemap--url-inventory)
2. [Redirect plan](#2-redirect-plan)
3. [SEO preservation & baseline](#3-seo-preservation--baseline)
4. [Content & CMS mapping (WordPress → Webflow)](#4-content--cms-mapping-wordpress--webflow)
5. [Scripts & tracking audit](#5-scripts--tracking-audit)
6. [Forms & integrations](#6-forms--integrations)
7. [Access & ownership](#7-access--ownership)
8. [Privacy & security](#8-privacy--security)
9. [Technical & design requirements](#9-technical--design-requirements)
10. [Migration & QA process](#10-migration--qa-process)
11. [Launch & recovery plan](#11-launch--recovery-plan)
12. [Timeline](#12-timeline)
13. [RACI — responsibilities](#13-raci--responsibilities)
14. [Dependencies](#14-dependencies)
15. [Known unknowns](#15-known-unknowns)
16. [Top risks & mitigations](#16-top-risks--mitigations)
17. [Decisions that must be approved before build](#17-decisions-that-must-be-approved-before-build)

---

## What we know about the current site (verified from public sources)

These facts anchor the plan and were confirmed via public search (not a live crawl):

| Fact | Evidence | Implication |
|---|---|---|
| Site runs on **WordPress** | Gated PDFs served from `/wp-content/uploads/YYYY/MM/…` | WP export + custom-field audit needed (§4) |
| Primary marketing domain `www.officernd.com` | Homepage + all marketing URLs | In scope for migration |
| **Two product lines**: Flex (coworking) & Workplace (hybrid) | `/coworking-software/`, `/pricing/flex/`, hybrid pages | Two parallel IA trees to preserve |
| Solution/pillar pages with children | `/coworking-software/coworking-crm/`, `/coworking-operations/`, `/member-mobile-apps/`, `/hybrid-work/` | Silo/hub structure — internal-link equity to protect |
| Large **blog** with taxonomies | `/blog/…` webinar recaps, guides, product posts | Highest-volume content type; CMS mapping critical |
| **Resources hub** + gated assets | `/resources/`, ebook PDFs in `/wp-content/uploads/` | Gated-asset + form flow to rebuild (§6) |
| A **stray test page is indexed**: `/resources-test/` | Appeared in `site:` results | 🔴 Should be `noindex` + removed, not migrated (finding) |
| Standard corporate pages | `/about/`, `/contact-us/`, `/why-officernd/`, `/integrations/`, `/pricing/…`, `/security/` | Migrate/consolidate |
| `/security/` is an **external UpGuard Trust Center** | Trust center branding | Keep as external embed/redirect; do NOT rebuild in Webflow |
| **Subdomains NOT in scope** (separate systems) | See below | Must be documented, DNS-preserved, and cross-linked |

**Subdomains — document and preserve, do NOT migrate:**

| Subdomain | System | Action |
|---|---|---|
| `app.officernd.com` | The SaaS product (login) | Out of scope. Preserve DNS. Never touch. |
| `help.officernd.com` | Help Center (Intercom-style) | Out of scope. Preserve. Keep header/footer links. |
| `developer.officernd.com` | API docs (ReadMe) | Out of scope. Preserve. |
| `coworkingplus.officernd.com` | Community/partner property | 🔴 VERIFY owner & status. Out of scope. |

> 🔴 **VERIFY:** the definitive page list comes from three sources reconciled together — (a) Screaming Frog crawl of `www.officernd.com`, (b) WordPress admin export / `wp_posts` table, (c) Google Search Console "Pages" (indexed + not-indexed) and the last 16 months of GSC performance data. Anything present in GSC/analytics but missing from the crawl (orphans) must be caught here.

---

## 1. Complete sitemap & URL inventory

**Goal:** every current URL classified with a migration decision, so nothing is silently dropped.

### 1.1 How the inventory is built (execution steps)
1. Crawl `www.officernd.com` with **Screaming Frog** (render mode = JS; include PDFs; follow `/sitemap.xml` and `robots.txt`). Export **all** URLs.
2. Export WordPress content: Tools → Export (All content) + a DB dump of `wp_posts`, `wp_postmeta`, `wp_terms`. This catches drafts, unpublished, and orphan pages the crawler misses.
3. Pull **GSC → Indexing → Pages** (indexed + excluded) and **GSC → Search results** (last 16 months) to get every URL that has ever received impressions/clicks.
4. Pull **GA4** landing-page report (last 12 months) for traffic volume per URL.
5. Pull **Ahrefs/Semrush** top pages by organic traffic and by referring domains (backlink targets).
6. Reconcile all sources into one master list → `/templates/url-inventory.csv`.

### 1.2 Columns captured per URL (see `url-inventory.csv`)
`URL` · `Page type` · `Template` · `HTTP status` · `Indexable (y/n)` · `Organic clicks (12mo)` · `Sessions (12mo)` · `# Referring domains` · `Top keywords` · `In XML sitemap?` · `Depth` · `Word count` · `Has form?` · `Gated asset?` · `Language` · `Decision (migrate/consolidate/redirect/archive/delete)` · `Destination URL` · `Owner` · `Notes`

### 1.3 Page types to inventory (checklist — none may be skipped)
- [ ] Homepage
- [ ] Product/solution pillar pages (Flex, Workplace) and their child pages
- [ ] Feature pages (CRM, operations, mobile apps, hybrid work, billing, analytics, visitor hub, growth hub…)
- [ ] Pricing pages (`/pricing/flex/`, `/pricing/workplace/` 🔴 VERIFY, add-ons)
- [ ] `/integrations/` index **and every individual integration detail page** (50+ integrations → potentially 50+ URLs)
- [ ] Comparison / "vs competitor" / alternatives pages (if any)
- [ ] Industry / use-case / persona pages
- [ ] `/resources/` hub + filtered views
- [ ] Blog index + **every blog post**
- [ ] Blog **author archives**, **category archives**, **tag archives**, **date archives** (WP generates these automatically — decide index vs noindex vs redirect)
- [ ] Blog pagination (`/blog/page/2/` …)
- [ ] Webinar pages (upcoming + on-demand recordings)
- [ ] Case studies / customer stories
- [ ] Gated landing pages (ebook/report/guide download pages)
- [ ] **Thank-you / confirmation pages** (post-form) — often orphaned but used as conversion goals
- [ ] Campaign / paid-landing pages (may be unlinked; find via GA4 + ad platforms)
- [ ] Event / community pages
- [ ] About, Careers, Team, Contact
- [ ] Legal pages: Privacy Policy, Cookie Policy, Terms, DPA, GDPR, Sub-processors, Acceptable Use
- [ ] `/security/` (external UpGuard — special handling)
- [ ] **PDFs and other binary assets** (`/wp-content/uploads/**/*.pdf`, images used as downloads)
- [ ] Search results page, 404 page, sitemap, RSS feed (`/feed/`)
- [ ] Utility: `/resources-test/` and any other test/staging pages → **delete + noindex** (finding)
- [ ] Localized/translated URLs if any (🔴 VERIFY — see hreflang in §3)

### 1.4 Decision rules
- **Migrate** — page has traffic, backlinks, or strategic value and has a 1:1 equivalent in the new IA.
- **Consolidate** — thin/overlapping pages merged into a stronger page; all old URLs 301 to the survivor.
- **Redirect** — no equivalent, but URL has equity/backlinks → 301 to nearest relevant page.
- **Archive** — keep content but `noindex` (e.g. old webinar recaps still linked internally).
- **Delete** — zero value, no backlinks, no traffic (e.g. `/resources-test/`, expired campaigns) → 301 to parent or 410 if truly dead and spammy.

**Risk:** orphan/campaign/thank-you pages missed → broken conversion tracking & lost PPC landing pages.
**Dependency:** GA4 + ad-account + GSC access (§7) before inventory can be completed.
**Owner:** SEO lead (build) + Marketing ops (campaign/TY pages).
**Verification:** row count in `url-inventory.csv` ≥ (GSC indexed pages + PDFs); every row has a Decision and (if not delete) a Destination URL. Zero blanks.

---

## 2. Redirect plan

**Goal:** preserve every inbound link and ranking with a clean, chain-free 301 map.

### 2.1 Mapping (`/templates/redirect-map.csv`)
Columns: `Old path` · `New path` · `Type (301)` · `Reason` · `Has backlinks?` · `Tested (pre-launch staging)` · `Tested (post-launch prod)` · `Final status code` · `Notes`.
Every non-"delete" URL from §1 must have exactly **one** row. Every "consolidate" old URL points to the survivor.

### 2.2 Rules
- **All redirects are 301 (permanent)**, single hop. **No chains** (A→B→C must be flattened to A→C **and** B→C) and **no loops**.
- **Trailing slash:** WordPress typically serves with a trailing slash. Pick **one canonical form** (recommend: keep trailing slash to match current URLs) and enforce it site-wide; redirect the non-canonical form. Do not flip the convention — flipping doubles the redirect table and risks chains.
- **Capitalization/case:** force lowercase; 301 any mixed-case variant to lowercase.
- **Protocol/host:** force `https://www.` — redirect `http://`, non-`www`, and any legacy hosts to the canonical `https://www.officernd.com`.
- **Query strings:** preserve tracking params on redirect where the destination needs them; strip only known-noise params (define allowlist).
- **Old campaign URLs:** pull from ad platforms + email tool + `utm`-heavy landing pages in GA4; redirect each to its live equivalent or the relevant pillar. These are the most commonly forgotten.
- **PDFs:** if a gated PDF URL has backlinks, keep the URL live (host the file in Webflow assets or a stable location) or 301 to the new download; never 404 a linked asset.
- **Webflow constraint:** Webflow's native 301 redirect table is capable but has a practical size limit and no regex on lower plans. If the map is large or needs pattern rules (e.g. all `/tag/*`), 🔴 **DECISION**: implement redirects at **Webflow** vs at the **CDN/reverse-proxy layer (Cloudflare)**. Cloudflare Bulk Redirects / Rules is recommended for large sets, wildcards, and case/slash normalization; keep only a small hand-picked set in Webflow. (See Decision D4.)

### 2.3 Testing after launch
- Pre-launch: run the full `Old path` list through the **staging** redirect config with Screaming Frog (List mode) → assert every source returns **single-hop 301 → correct target → 200**.
- Launch day: re-run the same list against **production**. Fail the launch gate if any source returns a chain, a loop, a 404, or a wrong target.
- Ongoing (weeks 1–4): re-crawl weekly; watch GSC "Page with redirect" and "Not found (404)" reports.

**Risk:** redirect chains/loops dilute PageRank and slow crawl; missed campaign URLs kill paid traffic.
**Dependency:** finalized URL inventory (§1) and finalized new IA/URL structure.
**Owner:** SEO lead (map) + Dev/DevOps (implementation at CDN) .
**Verification:** Screaming Frog List-mode report shows 100% single-hop 301→200; GSC 404 report flat after launch.

---

## 3. SEO preservation & baseline

**Goal:** capture a frozen pre-launch baseline and carry over every on-page SEO signal so rankings survive.

### 3.1 Pre-launch baseline (capture BEFORE anything changes — this is a gate)
- Full Screaming Frog crawl export (all on-page elements) → store as `baseline-crawl-YYYYMMDD.xlsx`.
- GSC performance export: queries, pages, countries, devices — last 16 months.
- GA4: sessions, conversions, top landing pages — last 12 months.
- Rank tracker: snapshot of top ~200 tracked keywords + positions (Ahrefs/Semrush/Rank tracker).
- Backlink export: all referring domains + their target URLs (Ahrefs). Flag high-authority targets — these URLs **must not** 404.
- Save the current `robots.txt` and all XML sitemaps.
- Core Web Vitals baseline (CrUX / PageSpeed) for key templates.

### 3.2 Per-page elements to carry over (verify on staging)
- `<title>` and meta description (1:1 unless deliberately improved).
- Heading structure (single `<h1>`, logical `h2/h3`).
- **Canonical tags** — self-referencing on the new URLs; watch Webflow's default canonical behavior.
- **Index/noindex rules** — replicate current noindex on thin/utility pages; ensure new staging is fully `noindex` and gets un-noindexed only at launch (critical Webflow gotcha).
- **Structured data / schema** — inventory current JSON-LD (Organization, Article, BreadcrumbList, FAQ, Product, Breadcrumbs, VideoObject for webinars) and rebuild in Webflow (custom code / CMS-bound). Validate with Rich Results Test.
- **hreflang** — 🔴 VERIFY if the site is multilingual/multiregional. If yes, reproduce hreflang cluster exactly; if no, no action.
- **Internal links** — preserve pillar↔cluster linking; run a broken-internal-link check post-migration.
- **Image alt text** — migrate alt attributes with images (see §4 media).
- **XML sitemaps** — generate fresh Webflow sitemap; submit in GSC at launch; keep only clean 200-canonical URLs in it.
- **robots.txt** — port intentional rules; **remove any staging `Disallow: /`** at launch; ensure sitemap reference is correct.
- **Pagination** — decide handling for blog pagination (`/blog/page/N/`); ensure crawlable or replaced with a crawlable "load more".

### 3.3 Rankings/backlinks to protect
- The pillar pages (`/coworking-software/…`), pricing, integrations index, and top blog posts carry the most equity — treat their URLs as **frozen** where possible (keep same path).
- Any URL with a referring domain in the backlink export is redirect-mandatory (never delete-to-404).

**Risk:** noindex leaking from staging to production; lost schema → lost rich results; canonical misconfig → duplicate content.
**Dependency:** baseline captured before content freeze; staging fully noindexed.
**Owner:** SEO lead.
**Verification:** post-launch crawl diff vs baseline (titles/meta/canonicals/schema present); GSC "Indexed" count recovers to baseline within 2–4 weeks; rank tracker stable ±2 positions on tracked terms.

---

## 4. Content & CMS mapping (WordPress → Webflow)

**Goal:** map every WP content type, taxonomy, and field to a Webflow CMS Collection — and flag anything Webflow can't do natively.

### 4.1 Source audit (WordPress)
Inventory from WP admin + DB:
- **Post types:** Posts (blog), Pages, and any custom post types (e.g. Case Studies, Webinars, Integrations, Resources, Glossary, Team). 🔴 VERIFY the full CPT list — likely several given the site size.
- **Taxonomies:** categories, tags, plus custom taxonomies (topic, product, industry, content-type).
- **Custom fields / ACF:** every ACF field group and field (e.g. gated-asset file, CTA text, author bio, read-time, hero image, related-content refs). Export field definitions.
- **Reusable components / blocks:** Gutenberg blocks, patterns, shortcodes, global blocks (CTA banners, testimonials, comparison tables) → map to Webflow **Components / Symbols**.
- **Authors:** WP users mapped to a Webflow "Authors" CMS collection (name, photo, bio, social).
- **Publication/modified dates:** preserve original publish dates (SEO + display).
- **Relationships:** post→author, post→category, post→related posts, resource→gated file, integration→category. Model these as **Reference / Multi-reference** fields in Webflow.
- **Filters & search:** current resource/blog filtering behavior and on-site search.

### 4.2 Target mapping (Webflow CMS)
| WordPress | Webflow |
|---|---|
| Blog Posts | `Blog Posts` collection |
| Categories / Tags | `Categories`, `Tags` collections (reference/multi-ref) |
| Authors (WP users) | `Authors` collection |
| Case Studies (CPT) | `Case Studies` collection |
| Webinars (CPT) | `Webinars` collection (fields: date, recording URL, gated file) |
| Integrations (CPT) | `Integrations` collection (logo, category, description, link) |
| Resources/Ebooks (CPT) | `Resources` collection (asset file, gated y/n, form ref) |
| ACF fields | CMS fields (text/rich-text/image/file/reference/option/switch) |
| Gutenberg reusable blocks | Webflow Components |
| Static pages | Static Webflow pages |

### 4.3 Webflow CMS limits to flag now (🔴 potential custom dev)
- **Collections:** max ~40 CMS Collections per site (10k items each; 2M/site on higher plans). Consolidate CPTs carefully.
- **Reference fields:** an item can hold **one** single-reference and limited multi-reference (max 5 multi-reference fields, up to a capped number of referenced items). Complex WP relationship graphs may not map 1:1 → may need denormalization or custom logic.
- **Nested collection lists** are limited in the Designer (can't deeply nest collection lists natively) → filtering/related content may need **Finsweet Attributes** or custom JS.
- **On-site search / filtering:** Webflow has no native full-text search or faceted filter → requires **Finsweet CMS Filter/Search**, **Jetboost**, or an external search (Algolia). Plan and budget this.
- **Pagination** is basic; large blog archives may need Finsweet load-more or static pagination.
- **Item limits per plan** and **CMS API rate limits** affect bulk import.
- **Redirects, forms submissions storage, and password protection** differ from WP — see §2, §6.
- **Migration tooling:** use **Webflow's WordPress importer** / **Udesly** / a scripted CSV import via the Webflow CMS API. Validate field mapping on a sample of 20 posts before bulk import.

**Risk:** relationship/filter features that WP does natively but Webflow can't → scope creep / custom dev.
**Dependency:** finalized IA + CMS schema sign-off before build; content freeze before final import.
**Owner:** Webflow developer (schema) + Content lead (QA of imported content).
**Verification:** item counts match WP export per collection; sample audit of 25 posts (fields, images, authors, dates, relationships) 100% correct; all filters/search functional on staging.

---

## 5. Scripts & tracking audit

**Goal:** know every script/pixel, its owner, purpose, load location, and correct home (GTM vs Webflow custom code vs native integration) — with a clean measurement plan and consent gating.

### 5.1 Inventory (`/templates/script-tracking-audit.csv`)
For each tag: `Name` · `Vendor` · `Purpose` · `Currently loads via (theme/plugin/GTM)` · `Load location (head/body/footer)` · `Still needed?` · `New home (GTM/Webflow code/integration)` · `Consent category` · `Owner` · `Notes`.

Scripts to look for (audit both the page source **and** the current GTM container):
- **GA4** (measurement ID) — confirm single source of truth; avoid double-tagging (hardcoded + GTM).
- **Google Tag Manager** container(s).
- **Google Ads** conversion + remarketing tags.
- **LinkedIn Insight Tag**, **Meta/Facebook Pixel**, **Twitter/X**, **Reddit**, **Bing UET** (typical B2B SaaS stack — 🔴 VERIFY actual set).
- **HubSpot** tracking code (very likely — see §6) and/or **Marketo/Pardot**.
- **Consent Management Platform** (Cookiebot / OneTrust / Osano — 🔴 VERIFY) — must load first and gate the rest.
- **Chat widget** (Intercom/Drift/HubSpot chat).
- **Heatmaps/session replay** (Hotjar/Clarity/FullStory).
- **A/B testing** (VWO/Optimizely/Google Optimize-successor).
- **Schema/JSON-LD** injectors, font loaders, video embeds (Wistia/Vimeo/YouTube), review widgets (G2/Capterra badges).

### 5.2 Where each belongs
- **GTM:** all marketing/analytics tags (GA4, Ads, LinkedIn, Meta, Bing, Hotjar) → managed in GTM, **triggered by consent state**, not hardcoded.
- **Webflow custom code:** only what must be in `<head>` early (CMP loader, GTM snippet, critical schema, font preloads). Use site-wide vs page-level code slots deliberately.
- **Native integration:** HubSpot forms/chat via HubSpot's Webflow integration or embed; not duplicated in GTM.

### 5.3 Measurement plan (events & conversions)
- Document current GA4 events & key conversions **before** migration (Admin → Events / Key events) so nothing is lost.
- Define the event map: `page_view`, `generate_lead` (demo request), `sign_up`, `file_download` (gated assets), `form_submit` (per form), `book_demo`, `webinar_register`, `chat_open`, `outbound_click` (to `app.` / `help.`).
- **Duplicate-event prevention:** single GA4 config tag; use GTM trigger exceptions; deduplicate Ads/Meta conversions with `transaction_id`/event_id; avoid GA4 both hardcoded and via GTM.
- **Cross-domain tracking:** configure GA4 cross-domain across `www.` ↔ `app.officernd.com` ↔ `help.` so demo→signup journeys stitch. Preserve `_ga` linker; add all domains to GA4 "configure your domains".
- **Consent Mode v2:** implement Google Consent Mode v2 (ad_storage, analytics_storage, ad_user_data, ad_personalization) wired to the CMP.

**Risk:** tags double-fire or go missing → corrupted analytics & ad optimization; consent not honored → GDPR exposure.
**Dependency:** CMP decision (§8), GTM access (§7), GA4 access.
**Owner:** Marketing ops / analytics lead + external analytics consultant if needed.
**Verification:** GTM Preview + GA4 DebugView on staging: every defined event fires once, with correct params, only after consent; Tag Assistant clean; ad-platform test conversions register; 48-hr GA4 real-time parity check post-launch vs pre-launch baseline volumes.

---

## 6. Forms & integrations

**Goal:** every form and third-party integration reproduced with identical behavior and CRM attribution.

### 6.1 Forms inventory (`/templates/forms-inventory.csv`)
Per form: `Form name` · `Page(s)` · `Fields` · `Required/validation` · `Hidden fields (utm, page, GCLID)` · `Routing/notification` · `CRM object created` · `Consent checkbox` · `Spam protection` · `Success behavior (msg/redirect)` · `Error handling` · `Owner`.

Forms almost certainly present:
- **Book a demo / Request a demo** (primary conversion) — routes to HubSpot/CRM, notifies sales, sets lifecycle stage.
- **Contact us** (`/contact-us/`).
- **Gated asset downloads** (ebooks/reports/guides) — form → deliver PDF + email + CRM tag.
- **Webinar registration** — likely via webinar tool (Zoom/GoToWebinar/Livestorm) or HubSpot.
- **Newsletter / blog subscribe.**
- **Pricing / sales contact.**

### 6.2 Behavior to preserve for each
- Client + server validation, inline error messages.
- **Hidden fields:** UTM parameters, `gclid`/`msclkid`, referring page, form-source, IP/country → required for CRM attribution. These break easily in migration — test explicitly.
- **Consent:** GDPR marketing-consent checkbox (unticked by default) + link to privacy policy; store consent with the submission.
- **Spam protection:** reCAPTCHA v3 / hCaptcha / honeypot (Webflow native has basic reCAPTCHA + honeypot; HubSpot forms have their own).
- **Confirmation:** thank-you page (redirect, so it's trackable) **and/or** inline message; deliver gated file.
- **Error handling:** graceful failure + retry; ensure submissions aren't silently lost.

### 6.3 Integration decision — how forms are built (🔴 DECISION D5)
Two viable patterns:
- **A) Native HubSpot forms embedded in Webflow** — best CRM attribution, consent, and workflow reuse; recommended if HubSpot is the CRM. Tracking/dedupe handled by HubSpot.
- **B) Webflow native forms → HubSpot via integration/webhook (Zapier/Make/native)** — more design control, but you must rebuild attribution, consent storage, and notifications, and Webflow stores submissions with limits.
Recommend **A** for lead-gen forms, **B** only for trivial forms (newsletter) if desired.

### 6.4 Integrations to map & test
| Integration | Purpose | Migration action |
|---|---|---|
| **HubSpot / CRM** | Lead capture, attribution, workflows | Re-embed forms, verify tracking code, re-test workflows |
| **Webinar tool** | Registrations | Re-embed/re-link; test end-to-end |
| **Chat** (Intercom/Drift/HubSpot) | Sales chat | Reinstall snippet via GTM/HubSpot; consent-gate |
| **Reviews** (G2/Capterra badges) | Social proof | Re-embed widgets |
| **Scheduling** (Chili Piper/Calendly) | Demo booking after form | Re-integrate; test round-robin routing |
| **APIs/webhooks** | Any custom data flows | Inventory + re-point endpoints |

**Risk:** lost UTM/attribution → sales can't source leads; broken demo→scheduling handoff → lost pipeline; silent form failures.
**Dependency:** CRM (HubSpot) access (§7); decision D5; consent/CMP (§8).
**Owner:** Marketing ops (CRM) + Webflow dev (build) + Sales ops (routing verification).
**Verification:** submit a real test lead through **every** form on staging and prod; confirm CRM record with correct attribution fields, notification received, correct lifecycle stage, gated file delivered, thank-you page fires conversion. Sign-off from Sales ops.

---

## 7. Access & ownership

**Goal:** confirmed access to every system **before** work begins. Missing access is the #1 cause of migration delay.

Track in `/templates/access-ownership-raci.csv`. Confirm each account's owner, admin, and that the team has working access:

| System | What for | Owner (🔴 confirm) | Access needed by |
|---|---|---|---|
| **WordPress admin + hosting** | Export, current config, staging | IT/Web | Dev, SEO |
| **Webflow** (workspace + site) | Build target | Marketing/Web | Dev, Designer |
| **Domain registrar** | DNS records for cutover | IT | DevOps (launch) |
| **DNS provider / CDN (Cloudflare?)** | DNS, SSL, redirects, WAF | IT/DevOps | DevOps |
| **Google Tag Manager** | Tag migration | Marketing ops | Analytics |
| **GA4** | Baseline + events | Marketing ops | Analytics |
| **Google Search Console** | Baseline, sitemaps, launch monitoring | SEO | SEO |
| **Bing Webmaster Tools** | Bing baseline/sitemap | SEO | SEO |
| **CRM (HubSpot)** | Forms, attribution, workflows | Marketing ops | Dev, Marketing ops |
| **Cookie-consent platform** | Consent config | Legal/Marketing | Analytics/Dev |
| **Form/webinar/chat/scheduling tools** | Re-integration | Marketing ops | Dev |
| **Ad accounts** (Google/LinkedIn/Meta/Bing) | Pixels, campaign URLs, conversions | Growth/PPC | Analytics |
| **Ahrefs/Semrush + rank tracker** | Baseline, backlinks | SEO | SEO |
| **Email/marketing automation** | Campaign URLs, deliverability | Marketing ops | Marketing ops |
| **UpGuard Trust Center** (`/security/`) | Keep external | Security | Security |
| **API keys / third-party services** | Any embeds | Various | Dev |

**Risk:** access discovered missing mid-build → blocked tasks, slipped launch; orphaned admin credentials post-launch.
**Dependency:** none — this is the **first gate**. No build task starts until its systems' access is confirmed.
**Owner:** PM (chases access) with each system owner.
**Verification:** every row in the access CSV marked "confirmed working" with a named owner + backup before Phase 2 starts. Kickoff gate.

---

## 8. Privacy & security

**Goal:** GDPR-compliant consent and data handling preserved; credentials rotated after cutover. (OfficeRnD is ISO 27001 / GDPR / SOC 2 — the bar is high.)

- **Cookie categories & consent:** replicate categories (necessary, preferences, statistics, marketing). Non-necessary scripts **blocked until consent** (Consent Mode v2 + CMP auto-blocking). Re-scan cookies after build and update the CMP declaration.
- **Privacy/Cookie/Legal pages:** migrate Privacy Policy, Cookie Policy, Terms, DPA, Sub-processors, GDPR — verify latest legal-approved versions with Legal; keep URLs stable (backlinks + legal references).
- **Scripts gated until consent:** verify in DebugView that no analytics/ad tag fires pre-consent (esp. EU traffic).
- **Data retention:** confirm form-submission retention in Webflow/HubSpot aligns with policy; set GA4 retention window; document data flows for the RoPA.
- **Form-data handling:** submissions travel over HTTPS to CRM; PII minimized; consent stored with record.
- **Security headers:** set at CDN (Cloudflare) — HSTS, CSP (allowlist Webflow + tag vendors), X-Content-Type-Options, Referrer-Policy, Permissions-Policy. Test CSP doesn't block legitimate tags.
- **User permissions:** least-privilege in Webflow/GTM/HubSpot; remove ex-vendors.
- **Credential hygiene after launch:** rotate/remove old WP admin accounts, API keys, and any shared logins; revoke agency access no longer needed; decommission WP hosting only **after** rollback window closes (see §11).

**Risk:** tags firing pre-consent (GDPR fine exposure); CSP breaking tags; stale credentials.
**Dependency:** CMP decision; Legal sign-off on policy versions; CDN config.
**Owner:** Legal/DPO (policy & consent) + Security/DevOps (headers, credentials).
**Verification:** pre-consent cookie scan shows only necessary cookies; CMP declaration matches actual cookies; security-header scan (securityheaders.com) grade A; credential-rotation checklist signed post-launch.

---

## 9. Technical & design requirements

**Goal:** the new site meets performance, accessibility, responsive, and design-system standards.

- **Browser/device support:** latest 2 versions of Chrome, Safari, Firefox, Edge; iOS/Android; define minimum breakpoints.
- **Responsive:** all templates verified at mobile/tablet/desktop/large; no horizontal scroll; tap targets ≥44px.
- **Accessibility:** target **WCAG 2.2 AA** — color contrast, focus states, keyboard nav, alt text, ARIA on nav/menus/forms, skip links, semantic headings. Audit with axe + manual keyboard test.
- **Performance / Core Web Vitals:** LCP < 2.5s, INP < 200ms, CLS < 0.1 on key templates. Webflow is generally fast; risks are heavy hero media, too many embeds, and unoptimized images.
- **Image/video optimization:** serve WebP/AVIF, responsive `srcset`, lazy-load below fold; host webinar/demo video via Wistia/Vimeo/YouTube (not self-hosted). Migrate alt text (§3).
- **Fonts:** self-host or use Webflow-hosted fonts with `font-display: swap`; preload primary font; match brand.
- **Animations:** reproduce key interactions (Webflow Interactions/GSAP); keep them lightweight; respect `prefers-reduced-motion`.
- **Navigation:** rebuild header/footer as Webflow Components (Symbols); include cross-links to `app.`/`help.`/`developer.` subdomains; mega-menu parity.
- **Site search:** Webflow has no native search → Finsweet/Jetboost/Algolia (decided in §4).
- **404 page:** custom branded 404 with search + top links; return proper 404 status.
- **Design system:** build reusable components, style guide (typography, color variables, spacing, buttons, cards, CTA blocks) as Webflow Variables + Components before page build. Consider the **Lumos/Client-First** class system for maintainability.

**Risk:** CWV regression from heavy media/embeds; accessibility gaps; design drift without a component system.
**Dependency:** finalized design/brand; CMS schema (§4).
**Owner:** Designer + Webflow dev.
**Verification:** Lighthouse ≥90 perf/SEO/best-practices/accessibility on key templates; axe scan 0 critical issues; manual keyboard + screen-reader pass; cross-browser QA sheet complete.

---

## 10. Migration & QA process

**Goal:** a controlled build → freeze → migrate → QA → sign-off pipeline.

1. **Staging:** build in a Webflow staging (`*.webflow.io`) that is **fully `noindex` + password-protected**. Never let staging get indexed.
2. **Content freeze:** announce a freeze on WP edits from the final-import date; log any emergency WP changes to re-apply.
3. **Migration method:** schema build → sample import (20 items) → validate → bulk import via Webflow WP-importer/CSV/API → media migration (with alt text) → components → page build → redirects config.
4. **Automated QA:**
   - Screaming Frog crawl of staging: broken links, missing titles/meta, missing alt, noindex audit, orphan check.
   - Redirect List-mode test (§2.3).
   - HTML/CSS + accessibility (axe) automated pass.
   - Lighthouse/CWV batch.
5. **Manual QA:**
   - Page-by-page visual QA vs WP original (content parity, images, links).
   - Every form submitted end-to-end (§6).
   - Every tag/event verified in GTM Preview + GA4 DebugView (§5).
   - Cross-browser/device matrix.
6. **SEO crawl comparison:** diff baseline crawl (§3.1) vs staging crawl — titles, meta, canonicals, schema, headings, alt, internal links — investigate every delta.
7. **Analytics verification:** all events fire once, consent honored, cross-domain works.
8. **Sign-off criteria (all must be ✅ before launch):**
   - [ ] URL inventory 100% dispositioned, redirect map tested green on staging
   - [ ] Content parity QA complete, 0 P1 defects
   - [ ] All forms create correct CRM records with attribution
   - [ ] All tags/events verified, consent-gated
   - [ ] SEO crawl diff reviewed, no unintended noindex/canonical/schema loss
   - [ ] Accessibility & CWV thresholds met
   - [ ] Legal sign-off on policy pages + consent
   - [ ] Rollback plan tested (§11)
   - [ ] Stakeholder sign-offs: Marketing lead, SEO, Dev, Legal/DPO, Sales ops

**Risk:** launching with unresolved P1s; staging indexed; content drift during freeze.
**Dependency:** all prior sections complete; access (§7).
**Owner:** PM (process) + each functional lead (their sign-off).
**Verification:** signed QA checklist; go/no-go meeting recorded.

---

## 11. Launch & recovery plan

**Goal:** a safe cutover with a fast, tested rollback and intensive monitoring.

### 11.1 Pre-launch (T-24h)
- [ ] Full **backup of the WordPress site** (files + DB) + export; store off-host. This is the rollback source of truth.
- [ ] Lower DNS **TTL** to 300s at least 48h before cutover.
- [ ] Confirm **SSL** provisioned for the production domain on Webflow/CDN (test the cert on staging domain first).
- [ ] Publish Webflow site to a pre-prod check; final redirect map loaded (Webflow and/or Cloudflare).
- [ ] Freeze confirmed; final content re-sync of any freeze-window WP edits.

### 11.2 Launch (DNS/domain steps)
1. Add production domain in Webflow; get target A/CNAME records.
2. Update DNS at registrar/Cloudflare to point `www` (and apex, with redirect to `www`) to Webflow.
3. Verify SSL active (no mixed content, valid cert, HTTPS forced).
4. **Do NOT touch subdomain records** (`app.`, `help.`, `developer.`, `coworkingplus.`) — leave untouched.
5. Remove staging `noindex`/password on production; publish clean `robots.txt` + XML sitemap.
6. Submit sitemap in **GSC** and **Bing Webmaster Tools**; request indexing of top pages.
7. Run production redirect List-mode test → assert 100% single-hop 301→200.

### 11.3 Rollback procedure (tested beforehand)
- Trigger condition: catastrophic failure (site down, mass 404s, forms/CRM broken, tracking dead) not fixable within the agreed window (e.g. 2h).
- Action: repoint DNS back to WordPress origin (records saved). Because TTL is low, propagation is fast. WP host stays live and untouched until the rollback window (recommend **2 weeks**) closes.
- Communicate; capture root cause; re-attempt after fix.

### 11.4 Monitoring — first 24 hours
- Uptime/synthetic monitor (Pingdom/UptimeRobot) on homepage + key templates.
- GA4 real-time: traffic present and roughly matching baseline volume/pattern.
- Real-time form-submission + CRM-lead watch (submit live test lead post-cutover).
- Error monitoring: crawl for 404s/500s; watch server/CDN logs.
- Ad platforms: confirm conversions still firing; check landing-page URLs of live campaigns didn't break.

### 11.5 Monitoring — weeks 1–4
- Daily then weekly Screaming Frog crawls; fix new 404s/redirect issues.
- **GSC:** watch Coverage/Indexing (indexed count recovering), "Page with redirect", 404s, Core Web Vitals, manual actions; compare Performance vs baseline (expect a small temporary dip, recovering in 2–4 weeks).
- Rank tracker weekly vs baseline.
- Bing Webmaster equivalent checks.
- Weekly issue-triage with named owner per issue.
- **Decommission WordPress** only after the rollback window closes and metrics are stable; then rotate/remove old credentials (§8).

**Risk:** ranking dip beyond expected; forms/tracking silently broken; premature WP decommission losing rollback.
**Dependency:** low TTL set in advance; tested rollback; monitoring access.
**Owner:** DevOps (DNS/SSL/rollback) + SEO (GSC) + Marketing ops (forms/tracking).
**Verification:** post-launch crawl clean; GSC indexing recovering; 0 open P1 issues at end of week 1; sign-off to decommission WP.

---

## 12. Timeline

Indicative; assumes a mid-size site (hundreds of URLs, one primary language). Adjust after the real inventory (§1).

| Phase | Duration | Key outputs | Gate |
|---|---|---|---|
| **0. Access & discovery** | Week 1 | Access confirmed (§7); tooling ready | Access gate |
| **1. Audit & baseline** | Weeks 1–2 | URL inventory, SEO baseline, script/form/CMS audits (§1,3,4,5,6) | Baseline captured |
| **2. Strategy & sign-off** | Week 3 | Final IA/URL map, redirect map, CMS schema, decisions D1–D6 approved | **Decisions approved** |
| **3. Design system & templates** | Weeks 3–5 | Components, style guide, key templates (§9) | Design sign-off |
| **4. Build & CMS migration** | Weeks 5–8 | Collections, imported content, pages, forms, tags (§4,5,6) | Feature complete |
| **5. QA** | Weeks 8–9 | Automated + manual QA, SEO crawl diff, redirect tests (§10) | QA sign-off |
| **6. Launch** | Week 10 | DNS cutover, redirects live, sitemaps submitted (§11) | Go/No-go |
| **7. Hypercare** | Weeks 10–14 | 24h + weekly monitoring, fixes, WP decommission | Stable → close |

**Total: ~10–14 weeks.** Blog-heavy sites or multilingual sites add time in Phases 1 & 4.

---

## 13. RACI — responsibilities

| Workstream | Responsible | Accountable | Consulted | Informed |
|---|---|---|---|---|
| Access & PM | PM | Web/Marketing lead | System owners | All |
| URL inventory & redirects | SEO lead | Web lead | Dev/DevOps | Marketing |
| SEO baseline & preservation | SEO lead | Web lead | Content | Marketing |
| CMS mapping & build | Webflow dev | Web lead | Content, SEO | — |
| Content migration & QA | Content lead | Web lead | SEO | — |
| Design system & templates | Designer | Web lead | Brand | — |
| Scripts & tracking | Analytics/Marketing ops | Marketing lead | Dev, Legal | Growth |
| Forms & integrations | Marketing ops | Marketing lead | Sales ops, Dev | Sales |
| Privacy & security | Legal/DPO + DevOps | Legal/DPO | Marketing ops | All |
| DNS/SSL/launch/rollback | DevOps | IT lead | Dev, SEO | All |
| QA & sign-off | PM | Web lead | All leads | Execs |

🔴 **VERIFY:** replace generic roles with named people before kickoff.

---

## 14. Dependencies

- **Access before build** (§7) — hard gate for everything.
- **Baseline before content freeze** (§3) — can't re-capture the "before" once WP changes.
- **Final IA/URL map before redirect map before build** — order matters; changing URLs late invalidates redirects.
- **CMS schema sign-off before bulk import** — schema changes after import are costly.
- **CMP decision before tag/consent build** (§5, §8).
- **CRM/forms decision (D5) before form build** (§6).
- **Low DNS TTL set ≥48h before launch** (§11).
- **Legal sign-off on policy pages + consent before launch** (§8).
- **CDN/redirect-layer decision (D4) before redirect implementation** (§2).
- WordPress host must stay live through the rollback window (§11) before decommission (§8).

---

## 15. Known unknowns (must be resolved during Phase 1)

- 🔴 Exact **URL count** and full list (needs live crawl — blocked in this environment; first execution task).
- 🔴 Full list of **WordPress custom post types, taxonomies, and ACF fields**.
- 🔴 Whether the site is **multilingual / has hreflang**.
- 🔴 The **actual CMP** in use (Cookiebot/OneTrust/Osano/other).
- 🔴 The **CRM** (assumed HubSpot) and marketing-automation tool.
- 🔴 The exact **tag/pixel set** and whether tags are hardcoded vs in GTM.
- 🔴 **Webinar/chat/scheduling** vendors.
- 🔴 Whether `/pricing/workplace/` and other pricing variants exist.
- 🔴 Ownership of `coworkingplus.officernd.com` and any other subdomains.
- 🔴 Volume of **backlinks per URL** (affects redirect priority).
- 🔴 Contractual constraints on **WP hosting** (notice period for decommission).
- 🔴 Any **campaign/landing pages** built outside WP (Unbounce/Instapage/HubSpot LPs).

---

## 16. Top risks & mitigations

| # | Risk | Likelihood | Impact | Mitigation | Owner |
|---|---|---|---|---|---|
| R1 | **Ranking/traffic loss** from broken redirects, lost equity, or noindex leak | Med | High | Complete 1:1 redirect map + tested green; baseline diff; staging noindexed; monitor GSC | SEO |
| R2 | **Staging gets indexed** (duplicate content) | Med | High | Password + noindex on staging; remove only at launch; verify robots | Dev/SEO |
| R3 | **Forms/CRM attribution break** → lost pipeline | Med | High | End-to-end test every form on staging + prod; verify hidden fields/UTM; Sales ops sign-off | Marketing ops |
| R4 | **Tracking double-counts or disappears** | Med | High | Single GA4 source; GTM Preview/DebugView; dedupe; 48h parity check | Analytics |
| R5 | **Consent/GDPR non-compliance** (tags pre-consent) | Med | High | Consent Mode v2 + CMP auto-block; pre-consent cookie scan | Legal/DPO |
| R6 | **Webflow CMS limits** block a needed feature (search/filter/relationships) | Med | Med | Early schema spike; Finsweet/Jetboost/Algolia budgeted; flag custom dev | Webflow dev |
| R7 | **Missed campaign/orphan/thank-you pages** | Med | Med | Reconcile GA4 + ad accounts + GSC, not just crawl | Marketing ops |
| R8 | **Access discovered missing** mid-project | Med | Med | Access gate in Week 1; named owners + backups | PM |
| R9 | **CWV/accessibility regression** | Low | Med | Component system; image optim; Lighthouse/axe gates | Designer/Dev |
| R10 | **Premature WP decommission** kills rollback | Low | High | Keep WP live 2 weeks post-launch; decommission checklist | DevOps |
| R11 | **Redirect volume exceeds Webflow limits / needs wildcards** | Med | Med | Implement bulk/wildcard redirects at Cloudflare (D4) | DevOps |
| R12 | **Legal pages migrated with stale versions** | Low | High | Legal confirms latest versions pre-launch | Legal |

---

## 17. Decisions that must be approved before build

| ID | Decision | Options | Recommendation | Approver |
|---|---|---|---|---|
| **D1** | Keep vs redesign IA/URLs | Keep URLs (max SEO safety) / Redesign | **Keep URLs** where equity exists; redesign only navigation/UX | Marketing + SEO |
| **D2** | Trailing-slash & case convention | Trailing slash / no slash | **Keep trailing slash** (matches current) + force lowercase | SEO |
| **D3** | Consolidate vs migrate thin content | 1:1 all / consolidate | Consolidate thin/duplicate; 301 to survivors | SEO + Content |
| **D4** | Redirect implementation layer | Webflow / Cloudflare / both | **Cloudflare** for bulk+wildcards, Webflow for a few | DevOps + SEO |
| **D5** | Form/CRM pattern | Native HubSpot embed / Webflow→HubSpot | **Native HubSpot** for lead-gen forms | Marketing ops |
| **D6** | On-site search & filtering solution | Finsweet / Jetboost / Algolia / none | Pick per budget; **Finsweet** for CMS filter, Algolia if full search needed | Web lead |
| **D7** | Rollback window length | 1 / 2 / 4 weeks | **2 weeks** before WP decommission | DevOps + SEO |
| **D8** | Handling of `/security/` (UpGuard) & subdomains | Keep external / rebuild | **Keep external**, cross-link only | Security |
| **D9** | Localization/hreflang scope | Single language / multilingual | Confirm in Phase 1; scope accordingly | Marketing |

---

## Appendix — companion templates (in `/templates`)
- `url-inventory.csv` — the master URL disposition sheet (§1)
- `redirect-map.csv` — 1:1 301 mapping + test status (§2)
- `script-tracking-audit.csv` — every tag/pixel + its new home (§5)
- `forms-inventory.csv` — every form + behavior (§6)
- `access-ownership-raci.csv` — systems, owners, access status (§7)

*Prepared as a draft for stakeholder review. All items marked 🔴 VERIFY require confirmation during Phase 1 (Audit & baseline), whose first task is a live crawl of `www.officernd.com` that this preparation environment could not perform due to an egress-policy block on the host.*
