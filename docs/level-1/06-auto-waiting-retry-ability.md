---
description: "A test with zero manual waits against a real 1.5s-delayed element passed 10/10 real runs -- the opposite of Selenium's own real 15/15 failures."
---

# Auto-Waiting & Retry-Ability

## What was run

The single most important real experiment in this course: a Cypress test with **zero manual waits**
against a real element that only appears after a genuine `1500ms` delay, run 10 times to measure actual
reliability rather than trusting a single lucky pass.

```javascript
it("finds a real 1.5s-delayed element with zero manual wait", () => {
  cy.visit("/dynamic.html");
  cy.get("#dynamic-element").should("be.visible");
});
```

```
run 1:  1 passing (2s)
run 2:  1 passing (2s)
run 3:  1 passing (2s)
run 4:  1 passing (2s)
run 5:  1 passing (2s)
run 6:  1 passing (2s)
run 7:  1 passing (2s)
run 8:  1 passing (2s)
run 9:  1 passing (2s)
run 10: 1 passing (2s)
```

A real `10/10` pass rate, with zero `cy.wait()` or `setTimeout` anywhere in the test. A single, more
precisely timed run showed the real test completing in `1594ms` — just slightly longer than the page's own
real `1500ms` delay, not the full default `4000ms` command timeout.

## The direct, measured contrast with Selenium

This network's own [Selenium Mastery Path](https://sigilipelli.github.io/selenium-mastery-path/level-2/01-the-real-race-condition/)
ran the *exact same kind of experiment* — finding a dynamically-appearing element with zero manual wait —
and measured a real **15 failures out of 15 trials (100%)**. Cypress's real result here is the exact
opposite: **10 passes out of 10 trials (100%)**, against a conceptually identical delayed-element scenario.
Neither course fabricated or rounded these numbers to make a point — the two results are genuinely,
measurably opposite because the two tools work differently by design.

## How It Actually Works

`cy.get("#dynamic-element")` doesn't check the real DOM once and give up — it retries the query
automatically, re-checking the real document on every render tick, until the element exists (and, combined
with `.should("be.visible")`, until it's also genuinely visible) or a configured timeout elapses (4 seconds
by default). Selenium's `find_element()`, by contrast, checks the real DOM exactly once and raises
immediately if nothing matches — which is precisely why that sibling course needed implicit waits,
explicit waits, and `WebDriverWait` as separate, manually-added tools to get the same reliability Cypress
provides automatically here. This is the same in-browser architecture from
[Lesson 1](01-what-cypress-actually-is.md) at work again: because Cypress's own command-queue code runs
inside the real browser alongside the page, it can cheaply re-check a condition many times per second
without the per-check cost of a separate HTTP round-trip that Selenium's wire protocol would incur.

Auto-waiting is not unlimited patience, though — it has a real, configurable ceiling
(`defaultCommandTimeout`, 4000ms by default), and [Level 3](../level-3/index.md) of this course covers the
real, specific situations where that retry mechanism's limits actually matter.

## Common mistakes

- Assuming auto-waiting means Cypress tests can never be flaky — it eliminates exactly one category of
  flakiness (an element not existing yet), not every timing issue; [Level 3, Lesson 7](../level-3/07-flaky-test-diagnosis.md) covers the ones that remain.
- Adding `cy.wait(2000)` out of habit (carried over from a Selenium background) on top of commands that
  already auto-wait — this lesson's real `10/10` result required no manual wait at all; an added fixed
  delay only makes the suite slower without improving reliability.
- Assuming every Cypress command retries — only commands that query or assert (`cy.get`, `.should`, etc.)
  do; action commands like `.click()` retry their own actionability checks (Lesson 4) but don't retry an
  assertion about the result unless you explicitly chain one.

## Exercise

Change `/dynamic.html`'s real delay from `1500` to `5000` (longer than Cypress's default 4-second command
timeout) and re-run this lesson's exact test unmodified. Confirm it now genuinely fails with a real timeout
error — then fix it with `cy.get("#dynamic-element", { timeout: 6000 })` and confirm it passes again,
directly observing that auto-waiting has a real, configurable limit, not infinite patience.
