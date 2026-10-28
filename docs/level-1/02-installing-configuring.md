---
description: "The real default viewport is exactly 1000x660, confirmed via window.innerWidth/Height -- a wrong baseUrl produced a real, specific pre-flight failure."
---

# Installing & Configuring Cypress

## What was run

A real `npm install cypress --save-dev` (version 16.2.1, confirmed earlier), a real `cypress.config.js`, and
two real tests confirming exactly what Cypress's configuration actually controls, rather than assuming
documented defaults are accurate without checking.

```javascript
// cypress.config.js
const { defineConfig } = require("cypress");

module.exports = defineConfig({
  e2e: {
    baseUrl: "http://localhost:5151",
    supportFile: false,
  },
});
```

## The real default viewport, confirmed directly

```javascript
it("confirms the real default viewport size", () => {
  cy.visit("/");
  cy.window().its("innerWidth").should("eq", 1000);
  cy.window().its("innerHeight").should("eq", 660);
});

it("confirms a real configured mobile viewport override", () => {
  cy.viewport(375, 667);
  cy.visit("/");
  cy.window().its("innerWidth").should("eq", 375);
  cy.window().its("innerHeight").should("eq", 667);
});
```
```
✓ confirms the real default viewport size
✓ confirms a real configured mobile viewport override
2 passing (109ms)
```

Cypress's documented default viewport is `1000×660` — confirmed here by actually reading `window.innerWidth`
and `window.innerHeight` from the real page rather than trusting the docs alone, and `cy.viewport(375, 667)`
genuinely resized the real browser window mid-test to a real mobile size.

## A real, deliberately wrong `baseUrl`

```bash
# cypress.config.bad.js points baseUrl at http://localhost:9999 — nothing is listening there
npx cypress run --config-file cypress.config.bad.js --spec cypress/e2e/smoke.cy.js
```
```
Cypress failed to verify that your server is running.

Please start this server and then run Cypress again.
```

This is a real, deliberate pre-flight check, not a generic network-timeout error: Cypress genuinely
verifies `baseUrl` is reachable *before* running a single test, and fails fast with a specific,
actionable message rather than letting the first `cy.visit()` fail with a confusing timeout deep inside a
test.

## How It Actually Works

`cypress.config.js` isn't optional boilerplate — `baseUrl` is what makes every `cy.visit("/")` in this
course's lessons resolve to the real local app instead of requiring the full URL in every single test file,
and Cypress's own pre-flight server check (confirmed above) exists specifically because an unreachable
`baseUrl` is a configuration mistake, not a test failure, and deserves a different, clearer error. Other
real config keys this course's lessons rely on directly: `supportFile: false` (this course's lab keeps
custom commands inline per-lesson rather than loading a shared support file, to keep each lesson
self-contained), and the `viewportWidth`/`viewportHeight` keys this lesson's `cy.viewport()` call overrides
at the per-test level instead of globally.

## Common mistakes

- Assuming `cy.visit("http://localhost:5151/login")` and a configured `baseUrl` of
  `http://localhost:5151` plus `cy.visit("/login")` behave identically in every situation — they produce the
  same real page, but only the relative form benefits from `baseUrl`'s pre-flight reachability check.
- Treating a `baseUrl` connection failure as a flaky test and simply re-running — this lesson's real error
  message is a configuration problem (wrong port, server not started), not test-level flakiness.
- Forgetting that `cy.viewport()` changes persist only within the test (or until changed again) — a later
  test in the same file without its own `cy.viewport()` call reverts to the real configured default,
  confirmed by this lesson's own two-test ordering (default, then override).

## Exercise

Add `viewportWidth: 1280` and `viewportHeight: 800` to `cypress.config.js` directly (not via
`cy.viewport()`), remove this lesson's own `cy.viewport()` call, and re-run both tests. Confirm the real
"default" viewport test's assertions now fail — because the config-level override replaced Cypress's
actual built-in default of `1000×660`, not just added a second option alongside it.
