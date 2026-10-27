# Web accessibility resources for an auditing agent

Researched 27 October 2026. Scope: an agent given a small set of webpages, combining deterministic checks with exploratory assessment.

Recommended starting point: **Playwright plus axe-core for browser execution and automated rules; W3C WCAG Understanding, ACT rules, and ARIA APG for the knowledge base; Accessibility Insights and TetraLogical for test procedures.** Add an independent engine only after measuring its additional useful findings. This recommendation is a synthesis, not a comparative performance benchmark.

The sources below are standards-body publications, maintainers’ documentation/source, named accessibility specialists, and original research. This provides accountable provenance; it cannot establish that no AI ever assisted any sentence. No SEO comparison articles are used as evidence.

## The most useful reading library

| Resource | What to extract for your agent | Authority and limitation |
|---|---|---|
| [WCAG 2.2](https://www.w3.org/TR/WCAG22/) | Success-criterion IDs, levels, requirements, definitions and exceptions. Start with A and AA. | Normative standard. A partial rule check is not proof that its whole criterion passes. |
| [Understanding WCAG 2.2](https://www.w3.org/WAI/WCAG22/Understanding/) | Intent, affected users, examples, exceptions, interpretation. Best central retrieval corpus. | W3C explanatory guidance, not normative requirements. |
| [WCAG Techniques and Failures](https://www.w3.org/WAI/WCAG22/Techniques/) | Concrete failures, implementation examples, remediation evidence. | Techniques are examples, not mandatory implementation choices. Other conforming approaches exist. |
| [Accessibility Conformance Testing rules](https://www.w3.org/WAI/standards-guidelines/act/rules/) | Applicability, assumptions, expectations, test cases, mappings. Best foundation for specifying custom checks. | Informative. Preserve approved/proposed/deprecated status; rules are not all fully automated. |
| [ARIA Authoring Practices Guide](https://www.w3.org/WAI/ARIA/apg/) | Interaction plans for dialogs, tabs, menus, comboboxes and other widgets; expected roles, states and keyboard operations. | Component guidance. A departure from a pattern is not automatically a WCAG violation. |
| [WAI Tutorials](https://www.w3.org/WAI/tutorials/) | Practical image, form, table, navigation and structure examples. | Strong remediation and contextual reasoning material. |
| [WCAG Evaluation Methodology](https://www.w3.org/TR/WCAG-EM/) | Scope, representative samples, complete processes, reporting. | Current publication is WCAG-EM 2.0, July 2026 Group Note. Use it to define what the audit actually covers. |
| [Accessibility Insights Assessment](https://accessibilityinsights.io/docs/web/getstarted/assessment/) | Step-by-step automated, assisted and manual testing procedures. | Current assessment page covers WCAG 2.2 AA; some overview material still mentions 2.1. Prefer the specific procedure. |
| [WebAIM WCAG checklist](https://webaim.org/standards/wcag/checklist) | Compact human-readable index for triage and navigation. | Deliberately simplified. Resolve ambiguous cases against W3C material. |
| [TetraLogical Accessible Components Library](https://tetralogical.com/resources/accessible-components-library/) | Requirements, Gherkin scenarios, working tests, compatibility notes and explicit manual checks. | Specialist-authored, inspectable code. Examples to adapt, not a generic arbitrary-page scanner or production-ready design system. |

Suggested reading order: Playwright’s integration guide → ACT rule examples → Accessibility Insights procedures → TetraLogical scenarios → WCAG Understanding and APG for each observed component. Read WCAG-EM before designing the report.

## Executable libraries and tools

| Tool | Fit | Decision |
|---|---|---|
| [axe-core](https://github.com/dequelabs/axe-core) | Browser-based rules engine; WCAG-tagged and best-practice rules; MPL-2.0. | Recommended baseline. Use existing rule implementations before inventing DOM checks. |
| [Playwright with axe](https://playwright.dev/docs/accessibility-testing) | Browser navigation and state setup, followed by `@axe-core/playwright` analysis. | Recommended orchestrator when your agent must interact with pages. |
| [Pa11y](https://github.com/pa11y/pa11y) | CLI/JavaScript automation; HTML_CodeSniffer default runner, optional axe runner, scripted actions. | Useful for URL batches. Pa11y using axe is not a second independent checking engine. LGPL-3.0 repository license. |
| [IBM Equal Access](https://github.com/IBMa/equal-access) | Independent engine and Node accessibility-checker package, including Playwright integration. | Good candidate for a second-engine experiment. Apache-2.0 repository license. |
| [Siteimprove Alfa](https://github.com/Siteimprove/alfa) | Modular ACT-based evaluation, structured outcomes, custom rules and question-based unresolved results. | Particularly interesting for agent-assisted evaluation. More integration work; maintainers flag most integrations as experimental. MIT license. |
| [QualWeb](https://github.com/qualweb/qualweb) | Alternative evaluation project; former standalone repositories point to this monorepo. | Research candidate. Verify current package APIs before adopting; the monorepo’s top-level README is sparse. |
| [WAVE API](https://wave.webaim.org/api/details) | Hosted evaluation with structured results and documentation API; separate stand-alone options. | Useful external alternative. Hosted service requests at most two concurrent calls; arbitrary authenticated workflows favor local browser control. |
| [Lighthouse](https://developer.chrome.com/docs/lighthouse/accessibility/scoring) | Weighted accessibility audit score. | Useful summary, poor primary evidence model. Manual audits do not affect the score. A perfect score cannot establish accessibility. |

Do not count wrappers as independent coverage. Compare engines on the same rendered states and evaluate confirmed incremental findings, duplicates, false positives, runtime and integration cost. Jonathan Pool’s [2023 nine-tool comparison](https://arxiv.org/abs/2304.07591) found complementary findings across 121 pages. That supports experimenting with multiple engines, not a claim about today’s best engine or a mandate to run all nine.

### Important axe details

Keep all four outcome categories: `violations`, `passes`, `incomplete`, `inapplicable`. An incomplete result needs investigation; an inapplicable result has no matching content. Preserve rule IDs, engine version, selectors/frame paths, HTML snippets, tags and help links. Separate best-practice results from WCAG failures. These fields and distinctions are documented in the [axe API](https://github.com/dequelabs/axe-core/blob/develop/doc/API.md).

Use a real browser. The [axe README](https://github.com/dequelabs/axe-core) explicitly notes limited JSDOM support and that color-contrast does not work there. Its quoted average automated detection percentage is not a coverage guarantee for your pages.

Pin dependencies and record browser and engine versions per run. Test configuration changes against fixtures before upgrading. This is an engineering recommendation for reproducibility, not a promise that a pinned engine stays complete.

## Divide checks by evidence needed

This is a proposed division of work informed by the sources above. A single WCAG criterion often contains both mechanically testable and contextual questions.

| Area | Deterministic evidence to collect | Agent judgment or specialist verification |
|---|---|---|
| Images | Name/alternative presence; exposed or decorative status. | Whether alternative conveys the image’s purpose; whether a chart has an adequate equivalent. |
| Controls | Computed name, role, states and label associations. | Whether the name communicates the action and matches the visible affordance. |
| Keyboard | Actual key presses, focused elements, activation results and escape paths. | Logical sequence, reachable task functionality, appropriate focus placement. |
| Contrast | Ratios where foreground/background and text properties are resolvable. | Complex imagery, gradients, applicability and state-dependent appearance. |
| Layout | Narrow-viewport, text-enlargement and spacing-override measurements; clipping/overlap candidates. | Whether information or functionality is lost; whether exceptions apply. |
| Target size | Hit-region geometry and neighboring target spacing. | Inline, equivalent-control, native-control and essential-presentation exceptions. |
| Forms | Error associations, invalid states, focus transitions and changed DOM. | Understandable instructions/errors and usable recovery. |
| Dynamic updates | Live-region and role/state changes. | Whether actual assistive technologies announce the right information at the right time. |
| Content | Heading, title, landmark and link-name inventory. | Descriptiveness, meaningful order and contextual link purpose. |

An agent can collect evidence and flag suspected barriers; that does not make its semantic judgment deterministic. Preserve that distinction in output.

## High-value mistakes to prevent

1. **Requiring nonempty alt on every image.** Decorative or redundant images can appropriately use empty alt; functional images need their purpose conveyed. Use the [WAI image decision tree](https://www.w3.org/WAI/tutorials/images/decision-tree/).
2. **Treating missing `aria-label` as missing name.** Accessible names can come from several mechanisms. Consult [Accessible Name Computation](https://www.w3.org/TR/accname/) and inspect computed semantics. The [1.2 document](https://www.w3.org/TR/accname-1.2/) is still a Working Draft as checked; distinguish draft guidance from the Recommendation.
3. **Treating spacing thresholds as required default styles.** The test is whether content survives user overrides: line height 1.5×, paragraph spacing 2×, letter spacing .12×, word spacing .16×, with applicable script/language considerations. [Understanding Text Spacing](https://www.w3.org/WAI/WCAG22/Understanding/text-spacing.html).
4. **Flagging every target below 44 pixels as an AA failure.** WCAG 2.2 AA target-size minimum is 24×24 CSS pixels with exceptions, including spacing and inline targets. [Understanding Target Size Minimum](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html).
5. **Equating any horizontal overflow with failed reflow.** Test loss and scrolling requirements at the relevant dimensions; genuinely two-dimensional content has exceptions. A narrow viewport also does not replace text-resize testing. [Understanding Reflow](https://www.w3.org/WAI/WCAG22/Understanding/reflow.html).
6. **Calling modal focus containment a keyboard trap.** A modal intentionally contains focus; evaluate its keyboard dismissal and sensible return path. Initial focus need not always be the first interactive element. [APG modal dialog pattern](https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/).
7. **Inferring screen-reader usability from DOM alone.** Browser semantics are useful evidence, not a record of actual speech or interaction. [GOV.UK assistive technology testing](https://www.gov.uk/service-manual/technology/testing-with-assistive-technologies) provides task-oriented procedures; its old version lists should not determine current test environments.

## Proposed audit architecture

1. **Define scope.** Save supplied URLs, allowed workflows, authentication conditions, target WCAG version/level, browsers, viewports and exclusions. Report a sampled audit when only a few pages are provided. [WCAG-EM](https://www.w3.org/TR/WCAG-EM/).
2. **Inventory each rendered page.** Collect headings, landmarks, forms, widgets, links, media and frame boundaries. Retain screenshots and relevant DOM/semantic evidence. [Playwright ARIA snapshots](https://playwright.dev/docs/aria-snapshots) can supply a structured representation; they are not a screen-reader session.
3. **Run deterministic rules on each important state.** Initial load, menu/dialog open, active tab, validation errors and task outcomes. Playwright documents setting up states before scanning. Use explicit readiness conditions and log states that could not be reached. [Playwright accessibility testing](https://playwright.dev/docs/accessibility-testing).
4. **Explore with a bounded plan.** Give the agent component-specific questions from APG, Insights and WCAG Understanding. Require real keyboard actions for keyboard claims. Treat inaccessible frames, timeouts and unsupported interactions as coverage gaps.
5. **Verify findings.** Replay actions, apply exceptions, collect expected versus actual behavior, then classify automated failure, suspected barrier, best-practice recommendation or unresolved test. Do not convert uncertainty to a pass.
6. **Report coverage alongside findings.** List tested states and omitted workflows. Group repeated component defects while keeping affected instances and reproduction evidence. Separate engine severity, practical user impact and confidence.

For a few supplied pages, prioritize meaningful state coverage over broad crawling. Reuse collected page evidence, retrieve only relevant criteria, and bound concurrent browser work. Measure cold launch separately from navigation, state setup, engine execution and model review. Those are design recommendations; no runtime benchmark was performed here.

Suggested finding record: URL, state identifier, steps, viewport/browser, element/frame locator, criterion and level, engine rule/version if applicable, expected/observed behavior, evidence attachments, outcome, confidence, user impact and source link. Suggested outcomes: fail, pass-for-tested-check, needs-review, not-applicable, not-tested, blocked. These are proposed product semantics, not a new conformance standard.

## How to build a trustworthy reference corpus

Use separate records for normative criteria, interpretations, ACT rules, component patterns and implementation examples. Keep source URL, publication/retrieval date, version/status and relevant exceptions with each record. Retrieve by component and criterion; preserve provenance when generating a finding. Avoid presenting a technique or APG recommendation as a mandatory WCAG implementation.

The [TetraLogical introduction](https://tetralogical.com/blog/2026/10/14/accessible-components-library/) explains its Playwright/Cucumber approach. It is unusually useful for turning prose into executable scenarios, but its automation claims concern its own known components. Do not extrapolate them to arbitrary webpages.

For human context, include [TetraLogical’s firsthand image-accessibility research](https://tetralogical.com/blog/2022/07/21/research-insight-accessibility-of-images/), [its screen-reader HTML support investigation](https://tetralogical.com/blog/2025/07/10/html-support/), and [WAI guidance on involving users](https://www.w3.org/WAI/test-evaluate/involving-users/). These help explain barriers that a structural checklist can miss. A small qualitative study illustrates experiences rather than establishing universal prevalence.

## Research directly relevant to your agent

[Agentic Web Accessibility Auditing](https://arxiv.org/abs/2609.09379), Mishra and colleagues, September 2026, is an unusually close match. Its criterion-specific agents combine browser actions with evidence collection. The [full paper](https://arxiv.org/html/2609.09379v2) describes 250 page–criterion records from 24 pages across 11 scholarly platforms: reported recall was 86%, with 56% precision against its reference labels.

Treat this as an early preprint, not validation of general autonomous auditing. Negatives were inferred from absence in audit reports; saved-page states and development exposure constrain interpretation. Positive cases covered only 15 criteria despite broader implementation. Its inspectable prompts, tools and supporting records are useful design material. The practical lesson is to evaluate both missed defects and false alarms, retain evidence and allow abstention.

For your own evaluation, assemble expert-reviewed pages and component states including both violations and valid exceptions. Measure per-check precision/recall, reproducibility, untested/blocked rate, state coverage, time and cost. Compare axe alone, agent alone, and combined output on the same inputs. Judge performance against expert-reviewed evidence, not issue counts or a Lighthouse score.

## Recommended starting scope

First version: axe plus browser interaction; keyboard and modal checks; forms and error recovery; names/images/headings; reflow and spacing review. Use source-linked findings and explicit uncertainty. Then benchmark IBM Equal Access or Alfa for incremental value. Keep actual assistive-technology and disabled-user testing in the verification process, following [WAI’s evaluation guidance](https://www.w3.org/WAI/test-evaluate/).
