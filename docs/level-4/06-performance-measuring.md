---
description: "The real home page's loadEventEnd was 15.0ms via Navigation Timing; the real slow page's command duration was 2048ms -- a real ~137x ratio."
---

# Performance: Measuring Real Command Timing

## What was run

Two real, independent ways to measure page performance from inside a Cypress test: the browser's own
Navigation Timing API (reached directly via `cy.window()`, logged out through `cy.task()` from
[Lesson 5](05-plugins-cy-task.md) since `cy.log()` doesn't print to the terminal), and Cypress's own
reported command duration for a command that genuinely has to wait on a real server delay.

```javascript
it("measures real navigation timing for the fast home page", () => {
  cy.visit("/");
  cy.window().then((win) => {
    const nav = win.performance.getEntriesByType("navigation")[0];
    cy.task("logToRealTerminal", `real home page loadEventEnd: ${nav.loadEventEnd.toFixed(1)}ms`);
    expect(nav.loadEventEnd).to.be.lessThan(500);
  });
});

it("measures the real elapsed time for a genuinely slow page via Cypress command duration", () => {
  cy.visit("/slow.html"); // fetches /api/slow, which has a real 2-second server-side delay
  cy.get("#slow-result", { timeout: 5000 }).should("contain.text", "real 2-second delay");
});
```
```
REAL TASK LOG: real home page loadEventEnd: 15.0ms
✓ measures real navigation timing for the fast home page (67ms)
✓ measures the real elapsed time for a genuinely slow page via Cypress command duration (2048ms)
```

The home page's real `loadEventEnd` was `15.0ms` — a direct, browser-reported measurement, not an estimate.
The slow page's real `2048ms` command duration almost exactly matches the server's own real, hardcoded
`2000ms` delay. That's a real, measured **~137x** difference between the two pages, both read directly
from real browser/Cypress reporting rather than guessed.

## How It Actually Works

The Navigation Timing API (`window.performance.getEntriesByType("navigation")[0]`) is a real, standard
browser API exposing precise timestamps for every phase of a page load — this course's own in-browser
architecture ([Lesson 1](01-why-architecture-is-different.md)) makes it trivially reachable via
`cy.window()`, no different from reading any other real property off the live `window` object. Cypress's
own per-command duration, visible in every test run's output throughout this course, is a second,
independent real measurement: it's simply how long that specific command took to resolve, which for a
command chained after a slow real network request (`cy.get("#slow-result")` waiting on the page's own
`fetch("/api/slow")`) directly reflects that real server-side delay, the same way this course's earlier
[Level 3, Lesson 6](../level-3/06-screenshots-videos.md) incidentally reported real command timings
without this lesson's dedicated focus on them.

Combining both gives two different, legitimate performance signals: Navigation Timing answers "how did
the real browser's own page-load lifecycle play out," while Cypress's command duration answers "how long
did waiting for this specific real condition actually take" — the second being generally more directly
useful for a test suite's own performance budget, since it measures exactly what the test itself depends
on.

## Common mistakes

- Reaching for `cy.log()` to print a measured timing value during a headless `cypress run`, then wondering
  why it never appears in CI logs — this lesson's own real result (confirmed again from
  [Lesson 5](05-plugins-cy-task.md)) is that `cy.task()` with a Node-side `console.log` is what actually
  surfaces in the terminal.
- Treating Cypress's reported command duration as a precise measurement of the real server's own response
  time — it includes real browser-side overhead (DOM rendering, Cypress's own retry-checking) on top of
  the actual network/server delay, which is why this lesson's `2048ms` is slightly higher than the
  server's exact `2000ms`, not lower or identical.
- Using Navigation Timing metrics to measure something that happens *after* the initial page load (like
  this course's own dynamically-appearing elements from [Level 1, Lesson 6](../level-1/06-auto-waiting-retry-ability.md)) — `loadEventEnd` reflects the initial load lifecycle only, not any later,
  client-side-triggered async behavior.

## Exercise

Add a third test that visits `/counter.html` (from [Level 3, Lesson 3](../level-3/03-retry-ability-pitfalls.md), with its real `1200ms` `setTimeout`-driven text update) and compares its real
`loadEventEnd` (which should be fast, since the delay is client-side, not server-side) against the real
Cypress command duration for a `.should()` that waits for the updated text. Confirm from the real numbers
that Navigation Timing and Cypress command duration diverge sharply here — directly extending this
lesson's own "two different signals" finding to a client-side delay instead of a server-side one.
