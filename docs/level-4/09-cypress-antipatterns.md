---
description: "Nine real anti-patterns, each backed by a genuine bug or surprise this course itself hit and investigated -- not invented examples."
---

# Common Cypress Anti-Patterns

## What was run

No new code — a catalog of real anti-patterns, each one tied directly to a genuine bug or surprising
behavior this course actually hit and investigated earlier, rather than invented hypothetical examples.

## 1. Passing an empty string to `.type()`

[Level 2, Lesson 2](../level-2/02-fixtures-test-data.md)'s fixture-driven login test genuinely crashed on
its very first run: `` `cy.type()` cannot accept an empty string ``. The anti-pattern: assuming any string
value from a data source is always safe to pass to `.type()` without checking for the empty case.

## 2. Breaking retry-ability with a plain `.then()` before `.should()`

[Level 3, Lesson 3](../level-3/03-retry-ability-pitfalls.md) measured this directly: `.invoke("text").then((t) => parseInt(t)).should("eq", 5)` genuinely failed, even with `.should()` chained right after the
`.then()`, because the plain callback in between doesn't retry. The anti-pattern: assuming any `.should()`
later in a chain makes the whole chain retry-safe.

## 3. Assuming `cy.intercept()` applies to `cy.request()`

[Level 3, Lesson 5](../level-3/05-intercept-deep-dive.md) found the identical intercept handler had zero
effect on a `cy.request()` call — the real server genuinely answered it directly with a real `401` — while
fully controlling the same logical request made through the real browser. The anti-pattern: forgetting
that `cy.request()` bypasses the browser's network layer entirely, which is exactly what
`cy.intercept()` watches.

## 4. Trusting a DOM alias to behave like a cached Selenium `WebElement`

[Level 3, Lesson 2](../level-3/02-aliases.md) confirmed `cy.get("@alias")` re-queries the DOM fresh each
time, rather than erroring on a stale reference the way Selenium's `StaleElementReferenceException` does.
The anti-pattern: assuming an alias will fail loudly if the original element is gone, when it may instead
silently resolve to a different real element matching the same underlying query.

## 5. Using `cy.pause()`/`cy.debug()` and expecting them to do anything in `cypress run`

[Level 2, Lesson 9](../level-2/09-debugging-failing-tests.md) confirmed directly: `cy.pause()` completed
in a real `217ms` under headless `cypress run`, with zero stalling. The anti-pattern: assuming these
interactive-mode-only tools have any effect outside `cypress open`.

## 6. Copying `Cypress.env()` snippets from older tutorials or training data

[Level 2, Lesson 7](../level-2/07-env-variables-config.md) hit a real, immediate removal error:
`` `Cypress.env()` was removed in Cypress version 16.0.0 ``, and even the replacement `cy.env()` needed a
different real signature (an array of keys) than older documentation describes. The anti-pattern: trusting
remembered or AI-generated API usage without checking it against the actually-installed version.

## 7. Assuming `--parallel` works like `pytest-xdist`'s `-n`

[Level 3, Lesson 9](../level-3/09-parallelization-cloud.md) found `cypress run --parallel` refused to run
at all without `--record`, which itself refused without a real Cypress Cloud `projectId`. The anti-pattern:
assuming every modern test runner's "parallel" flag is a free, local feature.

## 8. Forgetting the same-origin constraint applies to Cypress's own test code

[Level 4, Lesson 3](03-multiple-origins.md) and [Lesson 4](04-same-origin-constraint.md) both confirmed
real failures — a command timeout after crossing origins without `cy.origin()`, and a `contentDocument`
read that genuinely "does not exist" against a cross-origin iframe. The anti-pattern: treating Cypress's
in-browser test code as exempt from the same security boundaries any other in-page JavaScript faces.

## 9. Clicking a disabled element and assuming it behaves like Selenium's silent no-op

[Level 1, Lesson 4](../level-1/04-interacting-with-elements.md) found Cypress's `.click()` on a real
disabled button genuinely refuses after retrying ~4 seconds, with a specific error — the opposite of
Selenium's own silent no-op on the identical element. The anti-pattern: assuming a behavior verified in one
tool automatically transfers to a different one that happens to look similar.

## How It Actually Works

Every entry above traces back to the same root cause this course has repeated since Level 1: Cypress will
do exactly what its real, current version actually does — not what an older tutorial, a different tool's
convention, or an assumption carried over from Selenium or plain JavaScript predicts. Several of these
anti-patterns (3, 4, 9) are specifically about assuming Cypress behaves like Selenium when it architecturally
doesn't; several others (1, 2, 6, 7) are about assuming documentation or memory is current when the
actually-installed version has moved on. This course's own repeated methodology — run the real thing, read
the real result, investigate when it's surprising — is the direct antidote to every anti-pattern on this
list, because each one was caught by exactly that process.

## Common mistakes

- Treating this list as exhaustive — it covers the real bugs this specific course hit; a different
  application under test, or a future Cypress version, will surface its own different real surprises using
  the identical investigative method.
- Memorizing the fixes without the underlying mechanism — e.g. remembering "use `cy.env([...])` with an
  array" without understanding *why* the API changed, which won't help diagnose the *next* breaking change
  in a future version.
- Assuming anti-patterns only matter at scale — several of these (the empty-string `.type()` crash, the
  disabled-button assumption) were real, first-run surprises in a single test, not failures that only
  appear under load.

## Exercise

Pick any two anti-patterns from this list whose root cause you haven't directly reproduced yourself. Rerun
the real underlying lesson's code locally, and before looking at its real output, write down your own
prediction. Compare your prediction against the real result — this is the exact process that caught every
one of these nine anti-patterns in the first place.
