# Web audit agent research

Primary-source research for building an agent that audits a small set of webpages using deterministic checks, browser exploration and evidence-based judgment. Researched 27 October 2026.

| Handoff | Covers |
|---|---|
| [Accessibility](accessibility-handoff.md) | WCAG/ACT resources, axe and alternatives, interaction testing, contextual judgment and false positives. |
| [SEO and AEO](seo-aeo-handoff.md) | Crawling, indexing, structured data, AI search visibility, content assessment and unsupported optimization claims. |
| [Performance](performance-handoff.md) | Lighthouse, CrUX, runtime traces, repeatable measurements and bottleneck diagnosis. |

Each handoff includes recommended tools, authoritative reading, check design, evidence requirements and implementation priorities. SEO and AEO share a handoff because their discoverability foundations overlap; accessibility and performance need distinct testing methods.

## Suggested starting stack

- Browser automation for page state, interactions and evidence collection.
- axe-core for automated accessibility rules.
- HTTP/HTML inspection plus engine-specific documentation for SEO/AEO.
- Lighthouse for lab diagnostics; CrUX for available field data; traces for interactions.
- Agent review for context, prioritization and hypotheses, with source-linked findings.

Share URL/state inventories and evidence across domains. Run performance measurements separately from concurrent scans so instrumentation and CPU contention do not distort results.

## Reporting contract

Separate confirmed observations, suspected issues, recommendations, passed checks, not-applicable checks, missing evidence and blocked tests. Record URL, state, tool/version, environment, reproduction steps and source. A few pages establish sampled coverage—not site-wide accessibility conformance, guaranteed search visibility or population-wide performance.

These documents are AI-assisted research syntheses grounded in linked standards, official documentation, inspectable source code and original specialist work. They are not the primary sources themselves. Recheck changing crawler policies, API behavior and tool versions before implementation. No runnable auditor is included.
