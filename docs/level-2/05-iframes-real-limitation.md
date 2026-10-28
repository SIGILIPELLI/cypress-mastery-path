---
description: "cy.get() genuinely could not find a real button inside an iframe at all (2000ms timeout) -- a contentDocument workaround reached it in 125ms."
---

# Iframes: a Real Cypress Limitation

## What was run

A real test against a page with a genuine nested `<iframe>`, first using a normal `cy.get()` (which this
network's own [Selenium Mastery Path](https://sigilipelli.github.io/selenium-mastery-path/level-2/06-handling-iframes/) handles natively via `switch_to.frame()`), then a real, documented
workaround — to see exactly where Cypress's own DOM querying actually stops.

```javascript
it("cannot find an iframe's real content with a normal cy.get", () => {
  cy.visit("/iframe.html");
  cy.get("#frame-btn", { timeout: 2000 });
});
```
```
AssertionError: Timed out retrying after 2000ms: Expected to find element: `#frame-btn`, but never found it.
```

`#frame-btn` genuinely exists on the real page — just inside a real nested iframe's own document, not the
top-level one. `cy.get()` only ever queries the document Cypress itself is attached to, and (unlike
Selenium, which has an explicit `switch_to.frame()` concept) has no built-in concept of "now query inside
this iframe instead."

## The real workaround — no plugin, no frame-switching API

```javascript
cy.get("#content-frame")
  .its("0.contentDocument.body")
  .should("not.be.empty")
  .then(cy.wrap)
  .find("#frame-btn")
  .click();
```
```
✓ reaches real iframe content via a direct contentDocument workaround (125ms)
```

This genuinely worked, in `125ms`, with zero additional plugins installed. `cy.get("#content-frame")`
selects the real `<iframe>` element itself (not its contents); `.its("0.contentDocument.body")` reaches
into that real DOM element's native `contentDocument` property — a standard browser API, not a Cypress
one — to get the iframe's own real document body; `cy.wrap()` then hands that real DOM node back into
Cypress's own command chain so `.find()` and `.click()` work against it normally.

## How It Actually Works

This is a direct, real consequence of the same in-browser architecture covered in
[Lesson 1](../level-1/01-what-cypress-actually-is.md): Cypress's test code runs as JavaScript inside the
top-level page, and a same-origin iframe's content is reachable from there using nothing more exotic than
the DOM's own native `contentDocument` property — the same thing any page's own JavaScript could do to
inspect its own iframe. Selenium's WebDriver protocol, by contrast, models "the current browsing context"
as explicit server-side state (`switch_to.frame()` tells the real driver to route subsequent commands to a
different frame), which is why it has a dedicated, built-in API for this and Cypress historically didn't.
The real catch with this lesson's workaround: it only works for a **same-origin** iframe — a cross-origin
iframe's `contentDocument` is blocked by the real browser's own same-origin policy, which neither this
workaround nor (without extra configuration) Cypress itself can bypass; that specific, deeper constraint is
covered directly in [Level 4, Lesson 4](../level-4/04-same-origin-constraint.md).

## Common mistakes

- Reaching for a third-party plugin (like `cypress-iframe`) before trying the real, built-in
  `contentDocument` approach — this lesson's workaround needed no dependency at all for a simple,
  same-origin case.
- Assuming the workaround generalizes to any iframe — it only works same-origin; a cross-origin iframe
  (common for embedded widgets, payment forms, or ads) is a genuinely different, harder real problem.
- Forgetting `.should("not.be.empty")` before `.then(cy.wrap)` — without it, the chain can grab the iframe's
  body before its own content has actually loaded, since `cy.get()`'s auto-waiting doesn't know to wait for
  the iframe's internal load state the way it waits for the top-level document.

## Exercise

Change `/iframe-content.html`'s real button to render after a `1000ms` `setTimeout` instead of
immediately, and re-run this lesson's workaround test unmodified. Determine, from the real result, whether
the existing `.should("not.be.empty")` check on the iframe's `body` is actually sufficient to wait out that
real delay, or whether the test needs an additional explicit wait targeting the button itself — directly
testing where this workaround's own auto-waiting coverage actually ends.
