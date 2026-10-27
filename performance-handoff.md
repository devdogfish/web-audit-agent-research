# Web performance auditing handoff

Research checked 27 October 2026. Intended consumer: an agent given a few webpages, with browser tools and deterministic checks. This is a research and implementation handoff, not a completed performance audit.

## Recommended approach

Use **Lighthouse for repeatable load diagnostics, CrUX for available real-user evidence, and scripted browser journeys plus traces for runtime behavior**. Add the `web-vitals` attribution build when you control instrumentation. Use WebPageTest or Browsertime when you need more control over locations, browsers, repeat visits or journey measurement. These are architectural recommendations, not benchmark rankings.

The agent should explain bottlenecks from recorded evidence and propose experiments. It should not convert one Lighthouse score into a claim about every user, or turn a diagnostic suggestion into a guaranteed improvement.

## Authoritative resource library

| Resource | Why read it | How the agent should use it |
|---|---|---|
| [Web Vitals overview](https://web.dev/articles/vitals) | Metric definitions, current Core Web Vitals and assessment thresholds. Named author Philip Walton. | Shared vocabulary and field-assessment logic. |
| [Core Web Vitals threshold methodology](https://web.dev/articles/defining-core-web-vitals-thresholds) | Why thresholds were chosen. | Explain benchmarks without pretending they capture every aspect of experience. |
| [Optimize LCP](https://web.dev/articles/optimize-lcp) | Breaks loading delays into actionable subparts. Philip Walton and Barry Pollard. | Distinguish server delay, resource discovery, transfer and rendering. |
| [Optimize INP](https://web.dev/articles/optimize-inp) | Diagnoses input delay, processing and presentation. Jeremy Wagner, Philip Walton and Barry Pollard. | Inspect real interactions, not just startup JavaScript. |
| [Optimize CLS](https://web.dev/articles/optimize-cls) | Explains load and post-load instability and how to locate causes. Addy Osmani and Barry Pollard. | Distinguish shifted victims from the content that caused movement. |
| [PageSpeed Insights methodology](https://developers.google.com/speed/docs/insights/v5/about) | Clear lab/field distinctions, collection window and origin fallback. | Interpret supplied PSI reports correctly. |
| [CrUX API](https://developer.chrome.com/docs/crux/api/) | Programmatic page/origin data and device dimensions. | Field evidence with explicit scope and time period. |
| [Lighthouse performance scoring](https://developer.chrome.com/docs/lighthouse/performance/performance-scoring) | What the score actually measures. | Avoid substituting a composite score for bottleneck diagnosis. |
| [Lighthouse variability](https://developers.google.com/web/tools/lighthouse/variability) | Sources of measurement noise and repetition strategy. | Design comparable tests and report variability. |
| [Lighthouse user flows](https://github.com/GoogleChrome/lighthouse/blob/main/docs/user-flows.md) | Navigation, timespan and snapshot modes with examples. | Evaluate interactions and page states using the correct mode. |
| [Chrome runtime profiling tutorial](https://developer.chrome.com/docs/devtools/performance) | A concrete trace-based diagnosis workflow. | Correlate slow tasks, rendering and visual outcomes. UI examples are version-specific. |
| [W3C Resource Timing](https://www.w3.org/TR/resource-timing/) | What browser resource timings and sizes mean, including cross-origin restrictions. | Interpret measurements and missing evidence correctly. Current publication is a Candidate Recommendation Draft. |
| [W3C Server Timing](https://www.w3.org/TR/server-timing/) | How servers expose request-processing measurements. | Use backend timing when available; current publication is a Working Draft. |

These are standards and original maintainer/engineer documentation, not third-party SEO articles. Named authors and inspectable code provide provenance; they do not prove that every later edit was entirely unaided by AI.

Suggested reading order: Web Vitals → PSI methodology → variability → LCP/INP/CLS optimization → user flows → timing specifications when implementing collectors.

## Tool selection

| Tool | Best role | Constraints |
|---|---|---|
| [Lighthouse CI](https://github.com/GoogleChrome/lighthouse-ci) | Repeated Lighthouse runs, saved reports and budget/regression assertions. | Synthetic results need controlled environments. Read its [configuration](https://github.com/GoogleChrome/lighthouse-ci/blob/main/docs/configuration.md) before selecting assertions. |
| [Lighthouse user flows](https://github.com/GoogleChrome/lighthouse/blob/main/docs/user-flows.md) | Scripted navigation and timespan analysis; current documented Node examples use Puppeteer. | Timespans do not provide an overall performance score or navigation LCP. Snapshots are state audits, not load measurements. |
| [CrUX API](https://developer.chrome.com/docs/crux/api/) | Aggregated real-user evidence for eligible URLs or origins. | Requires a Google Cloud API key. URL, origin and device aggregations must not be mixed. |
| [PageSpeed Insights API](https://developers.google.com/speed/docs/insights/v5/get-started) | Convenient hosted Lighthouse diagnostics. | Google plans to discontinue CrUX data in this API and recommends direct CrUX APIs. Do not build the field collector around continued PSI field-data availability. |
| [web-vitals](https://github.com/GoogleChrome/web-vitals) | Browser metric collection and attribution for instrumentation you control. | An agent’s instrumented visit is still a lab observation. Real-user monitoring requires deployment and real traffic. Apache-2.0. |
| [WebPageTest API recipes](https://github.com/catchpoint/WebPageTest.api-recipes) | Scripted tests, result retrieval and repeat-view options. | Hosted location, account and API availability need checking at integration time. Official docs host timed out during this research; recipes and package docs supplied the verified interface evidence. |
| [Browsertime](https://www.sitespeed.io/documentation/browsertime/) and [sitespeed.io](https://www.sitespeed.io/documentation/) | Browser journeys, video/HAR/JavaScript measurements and ongoing monitoring. | More infrastructure than a small URL-based Lighthouse runner. Useful when multi-browser or detailed journey testing is required. |

Pick one primary lab runner first. The alternative tools improve observability and environment control; running several wrappers around Lighthouse does not create independent evidence.

## Measurement contract

Core Web Vitals currently comprise LCP, INP and CLS. Good thresholds are **LCP ≤ 2.5 seconds, INP ≤ 200 milliseconds, CLS ≤ 0.1**, evaluated at the 75th percentile with mobile and desktop distinguished. Do not apply these as if one synthetic sample were a population assessment. [Web Vitals](https://web.dev/articles/vitals).

PSI’s real-user view covers a trailing 28-day period and can fall back from the URL to the origin. Missing field data is not a performance failure. Preserve the returned scope and collection period. PSI documents an assessment exception when INP is unavailable: good LCP and CLS can still produce a passing assessment. Do not fabricate INP or silently describe all three metrics as measured. [PSI methodology](https://developers.google.com/speed/docs/insights/v5/about).

Recommended run manifest:

- Requested/final URL, timestamp and optional deployment identifier.
- Browser, runner and ruleset versions; operating system and hardware class.
- Viewport, device emulation, CPU/network settings and whether throttling is simulated or applied.
- Location, authentication, consent state, content variant and service-worker state.
- Cold/warm cache policy, navigation type, journey steps and stop conditions.
- Raw run IDs, failed attempts, selected summaries and retained evidence.
- Field-data URL/origin, form factor, collection period and missing metrics.

Start with three valid repetitions per scenario, expanding to five for unstable or borderline results; report median and range with individual runs retained. This is a proposed economical starting policy, not an official minimum or a statistically sufficient sample for every site. Avoid simultaneous timed runs on the same constrained machine. Lighthouse documents resource contention and page/network variability as important noise sources. [Variability guidance](https://developers.google.com/web/tools/lighthouse/variability).

Keep cold initial load, warm repeat visit and post-load task journeys separate. They answer different questions. A warm-cache speedup does not prove the first visit improved.

## Deterministic evidence versus agent judgment

Performance collection can be automated, but measured times are variable. “Deterministic check” here means reproducible procedure and explicit comparison logic, not identical timings on every run.

| Area | Evidence collector | Agent interpretation and next test |
|---|---|---|
| Loading | LCP element, timing subparts, resource initiators and trace. | Which part delays useful content? Test the smallest intervention targeting it. |
| Responsiveness | Actual action, interaction latency, main-thread and rendering trace. | Is delay input queueing, handler work or presentation? Does the user receive useful feedback? |
| Stability | Layout-shift events, screenshots and triggering actions. | What introduced movement? Is the shifted node the victim rather than the cause? |
| Network | Request waterfall, counts, transferred/decoded sizes, redirects and dependencies. | Is work avoidable, serialized unnecessarily or critical to the current task? |
| JavaScript | CPU trace, long tasks and coverage for the tested journey. | Which work can be deferred, split or removed without breaking other flows? |
| Images/fonts | Actual selected resources, display sizes, loading sequence and visual evidence. | Does an asset change improve the critical path without harming quality or stability? |
| Backend | TTFB, redirects, connection timing and available Server-Timing fields. | Which explanations are supported? Without server evidence, database latency remains a hypothesis. |
| Caching | Response directives, repeat-visit requests and service-worker behavior. | Is caching correct for the content, freshness and user scope? |
| Experience | Filmstrip, visible feedback and end-to-end task completion markers. | Can a user act promptly even when loading continues? What matters beyond the measured vitals? |

Use [LCP](https://web.dev/articles/optimize-lcp), [INP](https://web.dev/articles/optimize-inp), [CLS](https://web.dev/articles/optimize-cls) and [runtime profiling](https://developer.chrome.com/docs/devtools/performance) guidance to ground interpretations. The proposed collectors above are an implementation plan, not a claim that one package exposes all evidence automatically.

## Diagnosis procedures worth encoding

### Slow main content

Identify the actual LCP element in each scenario, then examine TTFB, resource load delay, resource load duration and element render delay. Compressing an image will not solve late discovery or a JavaScript-controlled reveal. Avoid lazy-loading a critical above-the-fold image; evaluate prioritization based on the recorded critical path. [Optimize LCP](https://web.dev/articles/optimize-lcp).

### Slow interactions

Choose meaningful actions: open navigation, type into search, apply a filter, open a dialog, or submit a safe test form. Exercise startup and settled states. Split latency into input delay, processing and presentation before suggesting fixes. A fast first response does not establish fast end-to-end asynchronous task completion; record that separately. [Optimize INP](https://web.dev/articles/optimize-inp).

### Layout movement

Observe after initial loading too: scroll, load delayed content and use navigation. Inspect reserved image/embed space and font changes. Attribute the cause from the timeline rather than blaming every shifted element. Use metric-defined exclusions rather than adding all movement yourself. [Optimize CLS](https://web.dev/articles/optimize-cls).

### Request fan-out

Record requests needed for the specific task and their dependency order. Separate authentication/setup from warm task fetches. Compare counts and end-to-end latency before and after a change; fewer requests alone do not prove speed improved. Check whether extra work repeats as visible records increase. This is a proposed application-level diagnostic extending beyond standard page-load scores.

## False conclusions to prevent

- **“Lighthouse 100 means users are fast.”** The score is computed from lab metrics; it is not the field distribution. [Scoring](https://developer.chrome.com/docs/lighthouse/performance/performance-scoring).
- **“No CrUX means poor performance.”** The URL may lack sufficient eligible data. Keep unavailable distinct from bad. [PSI](https://developers.google.com/speed/docs/insights/v5/about).
- **“Origin p75 describes this page.”** Origin data aggregates pages. Label fallback prominently. [CrUX API](https://developer.chrome.com/docs/crux/api/).
- **“A load audit proves interactions are responsive.”** Runtime work requires interaction testing; the user-flow modes serve distinct purposes. [User flows](https://github.com/GoogleChrome/lighthouse/blob/main/docs/user-flows.md).
- **“Total Blocking Time is INP.”** Treat startup blocking as diagnostic evidence, not a replacement for the interaction measurements described in [INP guidance](https://web.dev/articles/optimize-inp).
- **“Zero transfer size proves cache hit.”** Resource Timing restricts cross-origin fields unless permitted. Establish observability before interpreting zeros. [Resource Timing](https://www.w3.org/TR/resource-timing/).
- **“Unused code is safe to delete.”** Coverage is bounded by exercised states. Treat it as an investigation candidate, not a proven dead-code finding.
- **“Estimated savings can be added together.”** Recommendations may overlap on the same critical path. Validate the combined change experimentally.
- **“A faster skeleton means the task is faster.”** Track usable content and task completion alongside visible feedback; this is a product measurement recommendation.
- **“Observed TTFB identifies a slow database.”** Browser timing does not reveal internal processing without additional instrumentation. [Server Timing](https://www.w3.org/TR/server-timing/).

The code-coverage, overlapping-savings and task-completion cautions are reasoning rules for this agent; they are not universal numerical standards.

## Suggested execution pipeline

1. Inventory supplied URLs, page types, important actions and permitted interactions. Keep the scope sampled; do not claim whole-site coverage.
2. Fetch field data where available. Query distinct URL/device combinations, deduplicating origin fallbacks. Preserve missing data and errors.
3. Run isolated load scenarios with saved configuration and repeated measurements. Keep raw reports; do not retain only the best score.
4. Execute a small number of relevant journeys with explicit start/end markers. Collect traces separately if instrumentation materially changes overhead.
5. Join results by URL, state, device, environment and version. Do not average unrelated scenarios into one number.
6. Let the agent rank supported hypotheses by user impact, evidence strength and expected effort. Include the next verification experiment.
7. After changes, rerun matched scenarios. Report measured deltas and variability; use later real-user monitoring to confirm population effects.

Run ordinary metadata collection concurrently with bounds, but isolate timing-sensitive browser work. Reuse already collected reports instead of requesting repeated paid/hosted tests. Record failed tests and explicit timeouts. This pipeline is a recommended design; no application measurements were performed for this handoff.

## Finding and evaluation model

Suggested record: URL/state, measurement class (`lab`, `field`, `structural`, `hypothesis`), metric/value/unit, environment, run IDs, aggregation/window, observed evidence, suspected cause, suggested experiment, confidence and user impact. Preserve `unavailable`, `not-tested` and `blocked` separately from pass/fail. Budget thresholds are project policy unless tied to a cited standard or published metric threshold.

Evaluate the agent on controlled fixtures: late-discovered hero image, expensive event handler, delayed embed shift, unnecessary sequential fetches and a correctly cached repeat visit. Add healthy counterexamples and missing-telemetry cases. Check whether the agent identifies the right cause, cites actual evidence, abstains when needed and proposes a change that improves matched measurements. Do not reward a larger number of recommendations.

First implementation priority: reproducible Lighthouse reports plus direct CrUX lookup and a few interaction traces. Add more engines, locations or RUM only when they answer a demonstrated evidence gap.
