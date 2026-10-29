# Axe-core Guide: What It Covers and How to Run It (WCAG 2.1 AA)

Oct 27, 2026 · @Luigi Official

Axe checks only narrow, machine-testable parts of WCAG 2.1 AA. Run it so no result is lost, then hand its uncertain results, plus everything it cannot check, to the manual assessor (see the agent handoff document).

## What each axe result means

| Result | Meaning | Not the same as |
| --- | --- | --- |
| violations | A rule ran and found a definite failure. | The only problems in that area. |
| passes | One specific check passed on specific elements. | The whole WCAG criterion is met. |
| incomplete | Axe could not decide (“needs review”). | Fine. It always goes to manual review. |
| inapplicable | The rule found nothing to test. | A pass. Verify the feature is truly absent. |
| (in no bucket) | The rule was off, filtered out, or the page or state was never scanned. | Safe. This is the silent gap. |

Four things leave gaps even on a clean report: a rule covers only part of a criterion; no rule exists (28 of the 50 criteria, per the matrix below); a rule is off by default (experimental, deprecated, WCAG 2.2); and “needs review only” rules, which can never produce a clean pass. Axe also sees only the pages and states you scan.

## Run axe so nothing is skipped

1. Pin the axe-core version and record it with every scan.
2. Run the WCAG 2.1 AA tags, and switch on the two experimental rules that map to 2.1 AA criteria.
3. Capture all four result buckets and store the raw JSON.
4. Check that every expected rule appears in a bucket. If it does not, it did not run.
5. Scan each state (modal open, error shown, menu expanded), each viewport, and each step of a workflow, not just page load.
6. Run best-practice rules separately and label them non-WCAG.

Axe is a JavaScript library, so from a Python backend you drive a real browser with Playwright and inject a vendored, pinned copy of axe-core (copy `axe.min.js` from the npm package into your repo). Use the async API inside FastAPI, run scans as background jobs rather than in the request handler, and inject with `add_init_script` before navigating so axe is also present inside iframes.

```python
from playwright.async_api import async_playwright

AXE_JS = open('vendor/axe.min.js', encoding='utf-8').read()  # pinned axe-core version
SCOPE_TAGS = ['wcag2a', 'wcag2aa', 'wcag21a', 'wcag21aa']
EXTRA = ['css-orientation-lock', 'label-content-name-mismatch']  # off by default
BUCKETS = ['violations', 'incomplete', 'passes', 'inapplicable']

RUN_AXE = '''
async ([tags, extra]) => {
  const expected = axe.getRules()
    .filter((r) => r.tags.some((t) => tags.includes(t)))
    .filter((r) => !r.tags.includes('deprecated'))
    .filter((r) => !r.tags.includes('experimental') || extra.includes(r.ruleId))
    .map((r) => r.ruleId);
  const results = await axe.run(document, {
    runOnly: { type: 'tag', values: tags },
    rules: Object.fromEntries(extra.map((id) => [id, { enabled: true }])),
    resultTypes: ['violations', 'incomplete', 'passes', 'inapplicable']
  });
  return { version: axe.version, expected, results };
}
'''

async def scan(page):
    # Call after navigating and after putting the page into the state to test
    return await page.evaluate(RUN_AXE, [SCOPE_TAGS, EXTRA])

async def main(url):
    async with async_playwright() as p:
        browser = await p.chromium.launch()
        page = await browser.new_page(viewport={'width': 1280, 'height': 800})
        await page.add_init_script(AXE_JS)  # runs in every frame, including iframes
        await page.goto(url)
        raw = await scan(page)  # then open a modal, trigger errors, etc., and scan again
        await browser.close()
    return raw
```

The silent-gap check (`notRun`) happens in the handoff builder further down.

## Coverage matrix: WCAG 2.1 A and AA

Rules come from the axe-core 4.13 documentation; “(off)” means disabled by default. Treat this as a starting estimate and regenerate it from your pinned version with `axe.getRules()` (the `wcagNNN` tags map to criteria: `wcag143` is 1.4.3). Even “Substantial” never means complete.

| SC | Name | Axe rules | Coverage |
| --- | --- | --- | --- |
| 1.1.1 | Non-text Content | image-alt, input-image-alt, area-alt, object-alt, role-img-alt, svg-img-alt, aria-meter-name, aria-progressbar-name | Partial |
| 1.2.1 | Audio-only and Video-only (Prerecorded) | None (audio-caption is deprecated, off) | None |
| 1.2.2 | Captions (Prerecorded) | video-caption (needs review only) | Minimal |
| 1.2.3 | Audio Description or Media Alternative | None | None |
| 1.2.4 | Captions (Live) | None | None |
| 1.2.5 | Audio Description (Prerecorded) | None | None |
| 1.3.1 | Info and Relationships | list, listitem, definition-list, dlitem, th-has-data-cells, td-headers-attr, aria-required-children, aria-required-parent, aria-hidden-body; (off) p-as-heading, table-fake-caption, td-has-header | Partial |
| 1.3.2 | Meaningful Sequence | None | None |
| 1.3.3 | Sensory Characteristics | None | None |
| 1.3.4 | Orientation | (off) css-orientation-lock | Partial if enabled |
| 1.3.5 | Identify Input Purpose | autocomplete-valid | Partial |
| 1.4.1 | Use of Color | link-in-text-block | Minimal |
| 1.4.2 | Audio Control | no-autoplay-audio (needs review only) | Minimal |
| 1.4.3 | Contrast (Minimum) | color-contrast | Substantial |
| 1.4.4 | Resize Text | meta-viewport | Minimal |
| 1.4.5 | Images of Text | None | None |
| 1.4.10 | Reflow | None | None |
| 1.4.11 | Non-text Contrast | None | None |
| 1.4.12 | Text Spacing | avoid-inline-spacing | Minimal |
| 1.4.13 | Content on Hover or Focus | None | None |
| 2.1.1 | Keyboard | frame-focusable-content, scrollable-region-focusable | Minimal |
| 2.1.2 | No Keyboard Trap | None | None |
| 2.1.4 | Character Key Shortcuts | None | None |
| 2.2.1 | Timing Adjustable | meta-refresh | Minimal |
| 2.2.2 | Pause, Stop, Hide | blink, marquee | Minimal |
| 2.3.1 | Three Flashes or Below Threshold | None | None |
| 2.4.1 | Bypass Blocks | bypass (needs review only) | Minimal |
| 2.4.2 | Page Titled | document-title | Partial |
| 2.4.3 | Focus Order | None (tabindex is best-practice only) | None |
| 2.4.4 | Link Purpose (In Context) | link-name | Partial |
| 2.4.5 | Multiple Ways | None | None |
| 2.4.6 | Headings and Labels | None (empty-heading and heading-order are best-practice only) | None |
| 2.4.7 | Focus Visible | None | None |
| 2.5.1 | Pointer Gestures | None | None |
| 2.5.2 | Pointer Cancellation | None | None |
| 2.5.3 | Label in Name | (off) label-content-name-mismatch | Partial if enabled |
| 2.5.4 | Motion Actuation | None | None |
| 3.1.1 | Language of Page | html-has-lang, html-lang-valid, html-xml-lang-mismatch | Partial |
| 3.1.2 | Language of Parts | valid-lang | Minimal |
| 3.2.1 | On Focus | None | None |
| 3.2.2 | On Input | None | None |
| 3.2.3 | Consistent Navigation | None | None |
| 3.2.4 | Consistent Identification | None | None |
| 3.3.1 | Error Identification | None | None |
| 3.3.2 | Labels or Instructions | form-field-multiple-labels (needs review only) | Minimal |
| 3.3.3 | Error Suggestion | None | None |
| 3.3.4 | Error Prevention (Legal, Financial, Data) | None | None |
| 4.1.1 | Parsing | (off, deprecated) duplicate-id, duplicate-id-active | Minimal |
| 4.1.2 | Name, Role, Value | aria-allowed-attr, aria-required-attr, aria-roles, aria-valid-attr, aria-valid-attr-value, aria-prohibited-attr, aria-hidden-focus, button-name, link-name, label, select-name, frame-title, nested-interactive, and others | Substantial |
| 4.1.3 | Status Messages | None | None |

The `target-size` rule maps to WCAG 2.2, not 2.1, and is off by default. Regenerate the matrix if the enterprise moves to 2.2.

## What to hand to the agent from axe

For every scan, export this package and give it to the agent together with the target register and the handoff document. Violations go to developers; everything else here needs a human-style decision.

```python
from datetime import datetime, timezone

def pick(rule):
    items = []
    for node in rule['nodes']:
        checks = node.get('any') or node.get('all') or node.get('none') or [{}]
        items.append({
            'rule': rule['id'],
            'help': rule['helpUrl'],
            'selector': node['target'],
            'html': node['html'],
            'message': checks[0].get('message'),
        })
    return items

def build_handoff(raw, url, state, viewport):
    res = raw['results']
    seen = {r['id'] for b in BUCKETS for r in res[b]}
    return {
        'meta': {
            'axe': raw['version'],
            'url': url,
            'state': state,  # e.g. 'checkout step 2, error shown'
            'viewport': viewport,
            'date': datetime.now(timezone.utc).isoformat(),
        },
        'incomplete': [i for r in res['incomplete'] for i in pick(r)],  # agent reviews each
        'notRun': [i for i in raw['expected'] if i not in seen],  # silently did not run
        'inapplicable': [r['id'] for r in res['inapplicable']],  # agent verifies the feature is absent
        'violations': [i for r in res['violations'] for i in pick(r)],  # for developers
    }
```

Everything the matrix marks None, Minimal or Partial is covered by the manual checks in the agent handoff document, so the agent does not need this document.

## Sources

- axe-core rule descriptions: https://github.com/dequelabs/axe-core/blob/develop/doc/rule-descriptions.md
- axe-core API (getRules, run options): https://github.com/dequelabs/axe-core/blob/develop/doc/API.md
- WCAG 2.1: https://www.w3.org/TR/WCAG21/
