---
description: "cy.window() read and mutated the real page's own JS state directly, in-process -- confirming the mutation persisted, no serialization round trip."
---

# What Cypress Actually Is

## What was run

A real Cypress 16.2.1 test, installed via `npm install cypress` and run headlessly against a real local
Express app, directly accessing and mutating that page's own live `window` object — the concrete,
observable consequence of Cypress's defining architectural choice: **test code runs inside the same browser
as the application under test**, not as a separate client talking to a remote driver process.

```javascript
describe("what cypress actually is", () => {
  it("accesses the real page's own window object directly, in-process", () => {
    cy.visit("/");
    cy.window().then((win) => {
      expect(win.appInternalCounter).to.eq(0);
      const result = win.realAppFunction(5);
      expect(result).to.eq(5);
      expect(win.appInternalCounter).to.eq(5);
    });
  });
});
```

```
✓ accesses the real page's own window object directly, in-process (90ms)
1 passing
```

`win` in that callback is not a copy, a serialized snapshot, or a proxy — it's a real, direct reference to
the actual page's `window` object. Calling `win.realAppFunction(5)` genuinely executed that function inside
the real page, and the resulting mutation to `win.appInternalCounter` was immediately visible from the test
code, in the same tick.

## The real, measurable architectural contrast

```bash
ls ~/.cache/selenium/chromedriver   # from this course's sibling Selenium course
# mac-arm64

ls ~/Desktop/cypress-lab/node_modules/.bin/ | grep -i "cypress\|driver"
# cypress
```

Selenium needs a separate, real `chromedriver` binary — a standalone process the Python client talks to over
HTTP, which this network's own [Selenium Mastery Path](https://sigilipelli.github.io/selenium-mastery-path/)
verified directly at the wire-protocol level. Cypress's own installed binary list contains no such driver at
all — `npx cypress --version` reports a bundled Electron browser directly:

```
Cypress package version: 16.2.1
Cypress binary version: 16.2.1
Electron version: 41.7.0
Bundled Node version: 24.15.0
```

There is no separate driver to install, version-match, or restart — Cypress launches its own real,
bundled browser (Electron by default, or a system Chrome/Firefox it can also drive) and runs your test code
*inside* that browser's own JavaScript context, in the same event loop as the page under test.

## How It Actually Works

A Selenium script is a separate process sending real HTTP commands to a driver, which forwards them to a
browser it controls from the outside — three distinct processes, communicating over a wire protocol, for
every single command. Cypress collapses that into two processes: the real Node.js process running the test
runner, and the real browser process actually executing your test code *as JavaScript running in that
browser* — which is exactly why `cy.window()` can hand you a direct, live reference to the page's own
`window`, with no serialization step in between. This is also why Cypress is JavaScript-only (your test code
must run in the same JS context as the page) while Selenium and Playwright can offer client libraries in
many languages (their commands are just messages sent over a protocol, with no requirement that the client
language match the browser's own).

This same architectural fact is the direct reason behind two of Cypress's most distinctive real behaviors,
covered later in this course: automatic retry-ability (Level 1, Lesson 6) is possible because Cypress's
own command-queue code is running right there in the browser and can re-check a condition on every render
tick; and the same-origin constraint (Level 4, Lesson 4) exists because that in-browser test code is itself
subject to the browser's own cross-origin security boundaries — a constraint an out-of-process driver like
Selenium's doesn't share.

## Common mistakes

- Assuming Cypress is "just another Selenium-style client" with a nicer API — the in-browser execution model
  is a genuinely different architecture, not a styling difference, and it's the direct cause of several
  Cypress-specific behaviors (and limitations) covered later in this course.
- Expecting Cypress to support Python, Java, or C# test code — because test code runs *inside* the browser's
  own JS engine, Cypress is necessarily JavaScript/TypeScript-only; this isn't an arbitrary restriction.
- Assuming `cy.window()` returns some kind of snapshot or copy — it's a real, live reference; mutating it
  mutates the actual page, exactly as this lesson's real counter demonstrated.

## Exercise

Open this lesson's real app in a normal browser tab, open the DevTools console, and run
`window.realAppFunction(10)` by hand. Then write a second Cypress test that visits the same page and asserts
`win.appInternalCounter` starts at `0` again — confirming that each real `cy.visit()` gets a genuinely fresh
page load with no state carried over from your manual DevTools session, even though both are really the
same kind of direct, in-process window access.
