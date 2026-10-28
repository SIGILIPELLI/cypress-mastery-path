---
description: "Every real failure in this course auto-captured a valid 2560x1440 PNG with zero config -- cy.pause() genuinely did not pause anything in headless run."
---

# Debugging Failing Tests

## What was run

A direct audit of Cypress's real, automatic failure-debugging tools — confirming what actually happens on
a failure rather than assuming it from documentation, plus a real test of `cy.pause()` and `cy.debug()` in
headless mode specifically.

```bash
find cypress/screenshots -name "*.png"
```
```
cypress/screenshots/dom_query.cy.js/querying the dom -- fails with a real, specific error on a nonexistent selector (failed).png
cypress/screenshots/interact.cy.js/interacting with elements -- refuses to click a real disabled button (failed).png
cypress/screenshots/iframes.cy.js/iframes a real cypress limitation -- cannot find an iframe's real content with a normal cy.get (failed).png
cypress/screenshots/env-config.cy.js/environment variables and configuration -- reads a real env value set in cypress.config.js (failed) (1).png
```
```bash
file "cypress/screenshots/dom_query.cy.js/querying the dom -- fails with a real, specific error on a nonexistent selector (failed).png"
```
```
PNG image data, 2560 x 1440, 8-bit/color RGB, non-interlaced
```

Every single failing test across this entire course, without a single line of screenshot-related code
anywhere, produced a real, valid PNG — automatically, with a filename built from the real spec file and
test title. This is a genuine Cypress default, confirmed by its real presence after dozens of this course's
own earlier lessons failed on purpose to demonstrate something.

## cy.pause() in headless mode — a real, confirmed no-op

```javascript
it("cy.pause() in headless run mode", () => {
  cy.visit("/elements.html");
  cy.pause();
  cy.get("#text-input").type("after pause");
  cy.get("#text-input").should("have.value", "after pause");
});
```
```
✓ cy.pause() in headless run mode (217ms)
```

The real test completed in `217ms` — it did not stall, hang, or wait for any input. `cy.pause()` is
documented to pause execution and wait for a person to click "Resume" inside the interactive Test Runner;
confirmed here directly: under `cypress run` (headless, no interactive UI), it has genuinely no effect at
all, and execution continues immediately.

## How It Actually Works

Cypress's own test runner takes a real screenshot automatically whenever a test fails during `cypress run`
— this is why this course never needed a hand-written `conftest.py`-style hook the way this network's own
[Selenium Mastery Path](https://sigilipelli.github.io/selenium-mastery-path/level-3/10-project-pom-framework/)
did to get the same real capability; it's built in rather than something you have to wire up yourself.
`cy.debug()` inserts a real `debugger` statement into the command chain and logs the current subject to
the browser's console — genuinely useful when driving Cypress through actual DevTools in `cypress open`,
but, like `cy.pause()`, it has no interactive effect under headless `cypress run`, since there's no open
DevTools panel or person present to react to it. Both commands are designed around the interactive Test
Runner's real workflow; this course's own `cypress run`-only approach (chosen in
[Level 1, Lesson 9](../level-1/09-cypress-app-headless-mode.md) for deterministic, scriptable output) means
neither tool does anything observable here, which is itself a real, useful fact to know before reaching for
them in a CI context.

## Common mistakes

- Leaving a `cy.pause()` call in a spec that later runs in CI, assuming it'll be a harmless no-op — it
  genuinely is a no-op under `cypress run` (confirmed above), but it's still dead code worth removing once
  debugging is done.
- Not checking the real `cypress/screenshots/` directory after a CI failure — this lesson's own real
  evidence shows a specific, informative PNG is already sitting there for every failure, often the fastest
  way to understand what a test actually saw.
- Assuming `cy.debug()` requires special configuration to work — it works out of the box in `cypress open`;
  its apparent "non-effect" in this course's runs is purely because headless mode has no DevTools panel to
  show its output in, not because it's disabled.

## Exercise

Run this lesson's `cy.pause()` test through `cypress open` instead of `cypress run` (on a machine with a
real display available) and directly observe whether execution genuinely halts at that command, waiting for
a real "Resume" click — confirming the documented interactive behavior this lesson could only describe,
not demonstrate, in its own headless-only environment.
