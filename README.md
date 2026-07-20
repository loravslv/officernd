# OfficeRnD Website Migration Plan (WordPress → Webflow)

A complete migration plan for `www.officernd.com`, covering not just a page list but risks,
dependencies, owners, verification methods, timeline, and the decisions that must be approved.

## Contents

- **[officernd-migration-plan.md](./officernd-migration-plan.md)** — the full plan (all 11 topics + timeline, RACI, dependencies, known unknowns, risks, decisions).
- **[templates/url-inventory.csv](./templates/url-inventory.csv)** — master URL disposition sheet (§1).
- **[templates/redirect-map.csv](./templates/redirect-map.csv)** — 1:1 301 redirect mapping + test status (§2).
- **[templates/script-tracking-audit.csv](./templates/script-tracking-audit.csv)** — every tag/pixel and its new home (§5).
- **[templates/forms-inventory.csv](./templates/forms-inventory.csv)** — every form and its behavior (§6).
- **[templates/access-ownership-raci.csv](./templates/access-ownership-raci.csv)** — systems, owners, access status (§7).

## Status

Draft for stakeholder review. Items marked 🔴 **VERIFY** require confirmation during Phase 1
(Audit & baseline). Phase 1's first task is a live crawl of `www.officernd.com` (Screaming Frog +
Google Search Console + WordPress export), which the preparation environment could not run because
its network egress policy blocks the host; the structure here was reconstructed from public sources.
