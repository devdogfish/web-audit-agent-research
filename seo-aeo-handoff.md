# SEO and AEO audit agent handoff

Research checked 27 October 2026. Intended use: an agent receives several webpages, runs reproducible technical checks, then assesses content and search intent with evidence.

**Recommendation:** build on raw HTTP inspection, an HTML parser and Playwright; use Google Search Central as the technical reference, platform-specific crawler documentation for AI search, and owner-authorized search analytics for observed visibility. Do not manufacture an “AEO compliance” score from speculative rules. This architecture is a recommendation, not a benchmark result.

## What the agent can establish

Keep four questions separate: can the crawler access the page; is its content eligible for indexing; does a search engine actually index/serve it; does an answer system select and cite it? Passing technical requirements does not guarantee crawling, indexing or serving. [Google Search Essentials](https://developers.google.com/search/docs/essentials).

For Google AI Overviews and AI Mode, eligibility requires indexing and eligibility for a search snippet; no additional technical requirements apply. Robots, CDN access, internal discovery and visible textual content matter. [AI features and your website](https://developers.google.com/search/docs/appearance/ai-features).

A public-page audit can assess observed technical conditions and content. It cannot establish site-wide duplicate coverage, actual Google-selected canonicals, traffic losses, business outcomes or universal AI visibility without additional data. State which evidence is missing rather than reporting a pass.

## Authoritative reading library

| Resource | Extract for the agent | Boundary |
|---|---|---|
| [Google Search Essentials](https://developers.google.com/search/docs/essentials) and [technical requirements](https://developers.google.com/search/docs/essentials/technical) | Baseline eligibility and policy categories. | Requirements are not ranking guarantees. |
| [JavaScript SEO](https://developers.google.com/search/docs/crawling-indexing/javascript/javascript-seo-basics) | Raw versus rendered content, canonical consistency, rendering failures. | Local Chromium is useful evidence, not Google's production renderer. |
| [Robots introduction](https://developers.google.com/search/docs/crawling-indexing/robots/intro) and [robots metadata](https://developers.google.com/search/docs/crawling-indexing/robots-meta-tag) | Crawl controls versus indexing/snippet controls; HTML and HTTP directives. | A crawler must access a page to read its page-level directives. |
| [Canonicalization](https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls) | Redirect, canonical-link and sitemap signals. | Declared preference differs from actual search-engine selection. |
| [Crawlable links](https://developers.google.com/search/docs/crawling-indexing/links-crawlable) | Anchor markup and descriptive link text. | A working click handler alone does not establish crawlable navigation. |
| [Titles](https://developers.google.com/search/docs/appearance/title-link) and [snippets](https://developers.google.com/search/docs/appearance/snippet) | Descriptive metadata, duplication and page alignment. | Search appearance can differ from supplied metadata. |
| [Structured-data policies](https://developers.google.com/search/docs/appearance/structured-data/sd-policies) and [supported feature gallery](https://developers.google.com/search/docs/appearance/structured-data/search-gallery) | Applicable feature types, requirements and content consistency. | Syntax validity, feature eligibility and actual display are different. |
| [People-first content](https://developers.google.com/search/docs/fundamentals/creating-helpful-content) | Originality, evidence, usefulness and purpose questions. | A qualitative rubric, not a computable ranking formula. |
| [Google's generative AI optimization guide](https://developers.google.com/search/docs/fundamentals/ai-optimization-guide) | Current first-party AEO/GEO guidance and myth corrections. | Google-specific claims must not become universal claims about every AI system. |
| [OpenAI crawlers](https://developers.openai.com/api/docs/bots) | Search, training and user-request agent distinctions. | Permission to crawl is not a promise of citation. |
| [Microsoft AI content guidance](https://www.about.ads.microsoft.com/en/blog/post/october-2025/optimizing-your-content-for-inclusion-in-ai-search-answers) | Clear headings, useful answers, evidence and semantic consistency. | Named first-party guidance; recommendations, not a deterministic ranking specification. |
| [RFC 9309](https://www.rfc-editor.org/rfc/rfc9309) | Robots Exclusion Protocol semantics. | Combine protocol tests with documented crawler-specific behavior. |

For conditional modules, use [sitemap guidance](https://developers.google.com/search/docs/crawling-indexing/sitemaps/overview) and [localized-page guidance](https://developers.google.com/search/docs/specialty/international/localized-versions). A small, well-linked site may not need a sitemap; multilingual checks only apply when localized versions exist.

These are primary publications and inspectable technical references. They establish provenance and accountability, not a guarantee that no author ever used AI assistance. Avoid unattributed “ranking factor” lists and vendor claims unsupported by method or data.

## Libraries and tools

| Tool | Useful role | Important limit |
|---|---|---|
| [Cheerio](https://cheerio.js.org/docs/intro/) | Parse fetched HTML/XML; extract titles, directives, links and structured data cheaply. | Does not render CSS, load page resources or execute JavaScript. |
| [Playwright](https://playwright.dev/docs/intro) | Capture rendered states, content, network failures and interaction-dependent behavior. | Browser success does not prove crawler access or index inclusion. |
| [Lighthouse](https://github.com/GoogleChrome/lighthouse) | Supplement with its automated SEO audit category and machine-readable output. | Treat as selected checks, not a complete SEO/AEO verdict. |
| [Schema.org Validator](https://schema.org/docs/validator.html) | Examine extracted JSON-LD, RDFa and Microdata and syntax problems. | General vocabulary validation differs from Google's feature-specific eligibility. |
| [Google Rich Results Test](https://search.google.com/test/rich-results) | Manual/reference verification for supported Google rich-result features. | Do not assume an undocumented public automation API. Passing does not guarantee display. |
| [Search Console URL Inspection API](https://developers.google.com/webmaster-tools/v1/urlInspection.index/inspect) | Owner-authorized indexed-version status. | Does not perform a live URL indexability test. |
| [Screaming Frog SEO Spider](https://www.screamingfrog.co.uk/seo-spider/user-guide/general/) | Optional established crawler for wider inventories; list mode, headless CLI, saved crawls and exports. | Commercial features and deployment requirements; unnecessary overhead for a few URLs if browser infrastructure already exists. |

Use a standards-tested robots implementation rather than a quick substring check. Fixture-test wildcard matching, user-agent selection, path specificity, encoding, redirects and fetch failures against [RFC 9309](https://www.rfc-editor.org/rfc/rfc9309) and [Google's interpretation](https://developers.google.com/crawling/docs/robots-txt/robots-txt-spec). Pin versions and record configuration. No runtime or comparative engine benchmark was performed for this handoff.

## Deterministic evidence versus judgment

This matrix is a proposed implementation plan, grounded in the references above.

| Area | Reproducible checks | Agent assessment or additional evidence |
|---|---|---|
| Retrieval | Status, redirect chain, final URL, content type, TLS/navigation failures. | Whether an intentional redirect/error fits the URL's purpose. |
| Crawl access | Fetch and evaluate robots per target crawler; capture WAF/challenge responses. | Real crawler access needs verified logs; spoofing a user-agent does not prove it. |
| Index controls | Parse robots meta and X-Robots-Tag; preserve source and effective directives. | Whether noindex is intended for this page. Actual indexing needs platform evidence. |
| Rendering | Compare initial HTML and rendered main content, links and canonical declarations. | Whether missing information is essential; whether bot limitations affect discovery. |
| Canonicalization | Conflicting declarations, malformed targets, redirect/error/noindex targets. | Correct canonical intent and duplicate equivalence require related pages. |
| Metadata | Missing/empty titles, duplicate sampled titles/descriptions. | Whether title and summary describe page purpose and satisfy intended queries. |
| Navigation | Extract actual hrefs, test bounded targets and fragments, record redirect/error outcomes. | Logical information architecture; orphan status requires a larger inventory. |
| Structured data | Parse syntax, types, feature-required fields; compare explicitly available values. | Truthfulness, main-entity relevance and agreement with visible content. |
| Localization | Codes, alternate targets and reciprocal references where fetched. | Whether pages are equivalent localized versions and suit the audience. |
| Answers/content | Identify sections, claims, sources, dates, authors and entity names. | Completeness, ambiguity, original value, trustworthy evidence and answer usefulness. |
| AI visibility | Record platform-specific permissions and measured citations/impressions if available. | No local HTML test proves future citation, ranking or factual endorsement. |

Prioritize reproducible blockers affecting intended public content. Classify absent metadata or proposed content improvements separately from confirmed failures. Example: “canonical points to a 404 observed at time X” is stronger than “this page will not rank.”

## AEO details worth encoding correctly

**Google does not require llms.txt, special schema or rigid content chunking.** Its current guide explicitly says llms.txt does not affect Google Search visibility. Keep machine-readable documentation conveniences separate from search-ranking requirements. [Google AI guide](https://developers.google.com/search/docs/fundamentals/ai-optimization-guide).

**llms.txt remains useful in a different context.** Jeremy Howard's [proposal](https://llmstxt.org/) describes guided access to agent-friendly website information. It is not a universal search-engine contract. An audit can report its presence as an optional agent-navigation capability, never fail SEO for its absence.

**Search and training permissions differ.** OpenAI documents OAI-SearchBot for search, GPTBot for potential training data and ChatGPT-User for certain user-triggered visits. Blocking GPTBot does not imply blocking OAI-SearchBot. Inspect those policies independently and preserve the owner's intent. [OpenAI crawler documentation](https://developers.openai.com/api/docs/bots).

**Prefer useful answers over formulaic rewrites.** Microsoft recommends clear headings, evidence and suitable answer/list/table formats. Adapt those to actual reader tasks; do not impose an FAQ on every page or claim a guaranteed citation boost. [Microsoft guidance](https://www.about.ads.microsoft.com/en/blog/post/october-2025/optimizing-your-content-for-inclusion-in-ai-search-answers).

**Current feature support matters.** Google deprecated FAQ rich results starting May 7, 2026 and later removed the feature documentation. Advice saying ordinary FAQPage markup unlocks those rich results is stale. FAQ content and Schema.org vocabulary can still have other uses. [Google documentation updates](https://developers.google.com/search/updates).

## False positives to prevent

1. **Robots disallow means deindexed.** It controls crawling; it is not reliable removal from Google's index. Page-level noindex also cannot be read while the page is blocked. [Robots introduction](https://developers.google.com/search/docs/crawling-indexing/robots/intro).
2. **A 200 response proves a useful indexable page.** Inspect rendered content, directives and error-like states rather than judging status alone. [Google technical requirements](https://developers.google.com/search/docs/essentials/technical).
3. **Missing canonical is always an error.** Google can select a canonical without an explicit declaration. Flag conflicting or inappropriate signals, not mere absence as a universal blocker. [Canonicalization guidance](https://developers.google.com/search/docs/crawling-indexing/consolidate-duplicate-urls).
4. **Title or description length is a hard validity threshold.** Google truncates titles as needed and can generate snippets from page content. Length thresholds may be review heuristics, not search-engine requirements. [Titles](https://developers.google.com/search/docs/appearance/title-link), [snippets](https://developers.google.com/search/docs/appearance/snippet).
5. **Keyword density or minimum word count proves quality.** Google explicitly denies a preferred word count. Evaluate whether readers receive a complete, supported answer. [People-first guidance](https://developers.google.com/search/docs/fundamentals/creating-helpful-content).
6. **Valid JSON-LD guarantees a rich result.** Feature policies, supported types and content consistency still apply; appearance is never guaranteed. [Structured-data policies](https://developers.google.com/search/docs/appearance/structured-data/sd-policies).
7. **Client-rendered content is automatically invisible.** Google renders JavaScript; other bots differ. Report observed dependence or failure, not blanket impossibility. [JavaScript SEO](https://developers.google.com/search/docs/crawling-indexing/javascript/javascript-seo-basics).
8. **A sitemap or notification guarantees indexing.** A sitemap helps discovery. [IndexNow](https://www.indexnow.org/documentation) is a change-notification mechanism; submission success is not evidence of ranking or AI citation.

## Observing actual outcomes

Use owner-authorized reports instead of guessing from page markup:

- [Google Generative AI performance report](https://support.google.com/webmasters/answer/16984139) reports impressions for AI Overviews and AI Mode. The current help page states worldwide rollout completed August 31, 2026; insufficient impressions can prevent a report appearing. Older documentation that mentions only combined Web reporting is incomplete for this date.
- [Bing AI Performance](https://www.bing.com/webmasters/help/ai-performance-9f8e7d6c) reports aggregated, sampled citation activity across supported Microsoft/partner surfaces and grounding-query context. Citations are not clicks, rankings or proof of causality. Sparse activity may not surface; the report is not an exhaustive answer log.
- [URL Inspection API](https://developers.google.com/webmaster-tools/v1/urlInspection.index/inspect) concerns the indexed version. Label its observation time separately from the current page fetch, and do not present it as a live retest.

For bespoke prompt tests, predefine questions, platforms, language, region, account/session state, dates and repeated trials. Save complete responses and citations. Measure citation frequency, correct attribution, referral traffic and conversions separately. Those are proposed experimental practices, not standardized AEO metrics. A before/after increase alone does not establish that a page edit caused it.

## Original research worth reading

[GEO: Generative Engine Optimization](https://arxiv.org/abs/2311.09735), Aggarwal and colleagues, accepted to KDD 2024, introduces a benchmark and visibility metrics. The [full paper](https://arxiv.org/html/2311.09735v3) describes experiments with content modifications and differing domain effects. Read its methods and limitations before borrowing tactics: its maximum reported improvement is not a forecast for a current production platform or your site. Useful for experiment design, not deterministic checks or guaranteed uplift.

## Proposed implementation and report contract

1. Establish intended audience, page purpose and indexability intent. Default unknown intent to review, especially login, account, staging and transaction pages.
2. Fetch page-level HTTP evidence and per-origin policy files with bounded concurrency. Reuse evidence within a run; record freshness. Fetch only related canonical/alternate/link targets required by chosen checks.
3. Render pages and capture main content plus metadata after explicit readiness conditions. Compare raw and rendered states. Log timeouts and challenges as blocked evidence, not SEO defects.
4. Run versioned technical rules; attach source documentation and exact failing evidence. Then retrieve criterion-relevant guidance for the agent's qualitative review.
5. Keep outcomes separate: confirmed technical issue, suspected content issue, opportunity, intentional exclusion, pass-for-tested-check, not-applicable, not-tested and blocked.
6. Report affected URL/state, timestamp, browser/user-agent, source location, expected/actual observation, confidence, user/search impact, remediation and citation. Preserve raw evidence artifacts for replay.

For evaluation, build fixtures with intentional noindex, valid cross-domain canonicalization, JavaScript-generated content, multilingual variants, valid/invalid structured data, WAF failures and complete but short content. Have specialists review qualitative findings. Measure false positives, missed known defects, evidence completeness, repeatability and per-page cost. Never substitute an invented aggregate score for demonstrated search outcomes.
