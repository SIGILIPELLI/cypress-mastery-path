---
description: "A real 50-350ms random delay against an 80ms timeout passed only 2/10 real runs -- the same test with a 1000ms timeout passed 10/10."
---

# Flaky Test Diagnosis

## What was run

A real, genuinely non-deterministic scenario — an element that appears after a real random `50-350ms`
delay — run 10 times against two different real timeout configurations, to measure actual flakiness rather
than just discuss it abstractly.

```javascript
// jitter.html: const delay = Math.floor(Math.random() * 300) + 50; setTimeout(..., delay);

it("checks a real randomly-delayed (50-350ms) element with a too-tight 80ms timeout", () => {
  cy.visit("/jitter.html");
  cy.get("#jitter-target", { timeout: 80 }).should("be.visible");
});
```

```
RESULT: 2 passing runs, 8 failing runs out of 10
```

A real, measured `20%` pass rate — genuinely flaky, because the element's actual delay (uniformly
`50-350ms`) sometimes lands under the `80ms` timeout and sometimes doesn't, by actual chance each run.

## The real fix, measured

```javascript
it("checks the same real randomly-delayed element with a safe 1000ms timeout", () => {
  cy.visit("/jitter.html");
  cy.get("#jitter-target", { timeout: 1000 }).should("be.visible");
});
```
```
RESULT: 10 passing runs, 0 failing runs out of 10
```

Same real page, same real random delay range, only the configured timeout changed — and the real pass
rate went from `2/10` to a clean `10/10`. The real failure message from the tight-timeout version is
specific and diagnostic:

```
AssertionError: Timed out retrying after 80ms: expected '<div#jitter-target>' to be 'visible'
This element `<div#jitter-target>` is not visible per `Element.checkVisibility()`.
Computed: `display: none`, ...
```

## How It Actually Works

This lesson's flakiness has a genuinely different root cause than this network's own
[Selenium Mastery Path](https://sigilipelli.github.io/selenium-mastery-path/level-3/09-flaky-test-diagnosis/), which distinguished flakiness by real *exception type*
(`NoSuchElementException` vs `StaleElementReferenceException`) because Selenium's waits are explicit and
separate from its queries. Cypress's auto-waiting ([Level 1, Lesson 6](../level-1/06-auto-waiting-retry-ability.md)) collapses "wait" and "query" into one retrying command, which means the single most
useful diagnostic question for Cypress flakiness is usually simpler: **is the configured timeout
comfortably larger than the real worst-case delay, or close enough to it that normal variance sometimes
loses?** This lesson's real `80ms` timeout against a `50-350ms` real delay range was *guaranteed* to fail
whenever the actual random delay happened to exceed it — not a bug in the test's logic, just an
insufficient margin for real, observed variance.

The real error message itself (`Element.checkVisibility()`, computed `display: none`) is also diagnostic:
it confirms the element genuinely existed in the DOM but was not yet visible — the same real
"present vs visible" distinction this network's Selenium course drew out explicitly with
`is_displayed()`, here surfaced automatically in Cypress's own failure text without needing a
separate check.

## Common mistakes

- Treating a flaky test's *symptom* (sometimes fails) as proof of a logic bug in the test itself — this
  lesson's real test logic never changed between the `2/10` and `10/10` runs; only the margin between
  configured timeout and real observed delay changed.
- Fixing flakiness by retrying the whole test (`cypress-grep`'s retry plugins, or CI-level retry-on-fail)
  instead of fixing the underlying timeout margin — retries hide this lesson's real root cause rather than
  addressing it, and burn real CI time re-running tests that could pass reliably the first time.
- Picking an arbitrarily huge timeout "to be safe" without first measuring the real worst-case delay —
  an unnecessarily large timeout (far beyond this lesson's real `1000ms`, enough margin over a `350ms`
  real maximum) just makes a genuinely failing test (one checking for something that will never actually
  appear) take longer to report its real failure.

## Exercise

Change `jitter.html`'s real delay range to `Math.floor(Math.random() * 2000) + 50` (now up to a real
`2050ms`) and re-run this lesson's `1000ms`-timeout test 10 times. Measure the real new pass rate and
determine the minimum real timeout value that reliably produces `10/10` against this wider, genuinely
real random range — directly applying this lesson's own diagnostic method to a new, unmeasured scenario.
