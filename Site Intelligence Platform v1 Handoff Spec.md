# Site Intelligence Platform: v1 Handoff Spec

Status: planning complete, ready for high-fidelity prototype. Scope: one site, public pages only.

## 1. Product in one paragraph

An enterprise platform built on a crawler. It gives marketing and operations a clear, actionable picture of site health and what changed recently, with accessibility and compliance built in. Every element on screen must drive behavior, be actionable, or be insightful. No static dashboards.

**Primary viewers:** marketing and operations. **Secondary:** accessibility and compliance specialists.

## 2. Principles

1. **Deterministic first, AI second.** All counts, trends, and priorities come from the findings table. AI never computes or re-ranks anything.
2. **Standards, not opinions.** Priority is derived from published standards, so every P-level can be explained by pointing at a source.
3. **Deep modules over many areas.** Four pages, one findings model, one detail panel, one filter system.
4. **Defer cost.** AI output is generated on demand and cached, never pre-generated for every issue.

## 3. Data available

- Raw server GET response and client-rendered HTML for every crawled page.
- Full crawl history, so any two points in time can be diffed.
- axe-core results, Lighthouse results, and manual site-health checks.

## 4. What the UI needs from the findings data

The backend findings model is already implemented, so this spec does not prescribe a schema. It lists only what the UI relies on, to be mapped onto the existing fields.

- **Instance-level findings:** one per failing element for axe, one per page-level issue for Lighthouse and manual checks, so new, existing, and fixed stay accurate across crawls.
- **Traceable:** every finding links back to its source (axe, Lighthouse, or manual check) and its page.
- **Classifiable:** enough information to derive priority (section 7), area, and page template.
- **Guidance attached:** the standard description and source link, plus hand-written guidance for manual checks.
- **Window-aware status:** new, existing, or fixed can be computed for any selected time window.
- **Countable:** deterministic roll-ups by rule, page, and template.

## 5. Global controls (available on all four pages)

There is no separate filter page. Filters live in a persistent bar on every page and are split into two kinds.

**Global filters (persist as the user moves between pages):**

- **Time window:** presets for 1 day, 7 days, 30 days, plus custom. Compared against the site state at the window start, so it works regardless of crawl cadence (daily or 3x per week).
- **Scope:** all pages, or pages new or changed within the window.
- **Status:** new, existing, fixed.
- **Priority:** P1 to P4.
- **Template.**

**Page-local filters:** rule, WCAG level, Lighthouse category, source, and similar.

Filter state lives in the URL, so every view is shareable.

**Default scope per page:** Overview, Pages, and Site Health default to changed within the window. Accessibility defaults to all pages, because legal exposure is site-wide, with a new-in-window count shown at the top.

## 6. The four pages

### 6.1 Overview (landing page, read-only rollup)

The one place where data from all areas is shown together. It owns no data; every number is a link into a filtered view on another page.

- Per-area headline counts (Accessibility, SEO/Performance, Compliance) with change over the selected window.
- New and fixed totals for the window.
- Top 5 issues by priority.
- A short AI-generated summary (see section 8).
- No blended health score.

Overview is a fixed layout. There are no user-built or pinned cards in v1.

### 6.2 Pages (crawl explorer, the cross-cutting view)

- One filterable table of all URLs, defaulting to worst page first (P1 count).
- Columns: template, status, last changed, priority counts, Lighthouse scores.
- Because it spans every area, it is the natural destination for cross-area questions (for example, "which product pages got worse this week?").
- **Page detail view** has three tabs: **Findings**, **Changes** (content, metadata, and raw-vs-rendered diff between crawls), and **Source** (raw vs. rendered HTML).

### 6.3 Accessibility

- Axe findings grouped by rule, with WCAG criterion and level.
- Each group expands to pages, then to instances.
- Default scope: all pages.

### 6.4 Site Health

- Lighthouse and manual checks, in tabs: Indexing & SEO, Performance, Security, Compliance.
- Same grouping and expansion pattern as Accessibility.

**Open for prototype:** whether 6.3 and 6.4 are literally one shared component with different presets, or two tuned views. Build the grouped-findings explorer once and decide on presentation during prototyping.

### 6.5 Shared finding detail panel

Opens from any table, at instance level. Shows the failing element, why it fails, how to fix it (standard guidance with source link), and an on-demand AI fix (section 8).

## 7. Priority model (P1 to P4)

Computed, deterministic, identical on every page, shown on every row, and the default sort everywhere.

- **Accessibility:** WCAG conformance level (A before AA before AAA) plus axe impact.
- **Lighthouse:** its own audit weights.
- **Manual checks:** one fixed level per check, documented in the app.
- **Reach** (pages and instances affected) only breaks ties inside a level.
- Any user can ask "why is this P1?" and get an answer pointing to a published standard.
- **Noise control** is priority levels, grouping by rule and template, and strong filters. There is no per-issue ignore or waiver in v1.

## 8. AI features

AI selects queries, groups, and writes words. It never computes numbers or changes priority.

| Feature | Behavior |
| --- | --- |
| **Ask bar** | Global control in the header of every page, not its own page. The user types a question; AI translates it into global filters, page-local filters, and grouping, then navigates to the best-fit page with the URL set. Single-area questions land on that area's page; cross-area questions land on Pages. The result is a normal table: shareable and exportable. |
| **Fix suggestion** | Button in the detail panel, generated only when clicked. Grounded in the standard guidance and its source link, plus the element HTML, surrounding markup, and template. Shown as a copyable snippet or diff, labeled AI-generated, never auto-applied. For a template-level group, produces one fix for the shared component. |
| **Root-cause grouping** | Groups related findings by likely shared cause (for example, "41 findings trace to the same footer component"). Does not re-rank. |
| **Overview summary** | Short summary of the window. Hard rule: every sentence links to the filtered view behind it, and a sentence without a link does not ship. |

**Cost controls:** all AI output is on demand, not precomputed. Cache results (fix suggestions by rule plus element or template signature, the summary by window plus crawl) so repeat requests are free. Do not generate for issues nobody opens.

## 9. Manual checks (v1 catalog)

- Broken links and redirect chains
- Missing or duplicate titles and meta descriptions
- Canonical and noindex issues
- Robots and sitemap problems
- Raw-vs-rendered mismatches
- Security headers
- Cookie banner and privacy link presence

Each check needs a fixed priority level and hand-written fix guidance (sections 4 and 7).

## 10. Data retention

- Finding-level history: 12 months (supports the 30-day and custom windows and trends).
- HTML snapshots: the latest crawl, plus each page's previous version whenever it changed.

## 11. Export

CSV or Excel export and copy-to-clipboard on every table. No direct Asana integration in v1 (the team uses Asana manually).

## 12. Explicitly out of scope for v1

- Multi-site and site switcher
- Authenticated pages and multi-step JS states
- Roles and permissions beyond a basic account
- Email digests and notifications
- Per-issue ignore, waivers, assignment, and ticket sync
- User-built dashboards and pinned saved views
- Blended health score
- AI that scores, ranks, or computes figures

## 13. Open items for the prototype phase

1. Exact Overview layout and which top-level numbers earn a place.
2. Whether Accessibility and Site Health share one component or two tuned views.
3. The fixed priority level for each manual check, and the mapping of Lighthouse weights to P1 to P4.
4. Template detection beyond URL patterns.
5. When root-cause grouping runs (suggested: on demand per rule or template, cached).
6. Where the ask bar sits visually, and how it communicates which filters it applied so the user can correct them.
