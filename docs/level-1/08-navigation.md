---
description: "Typed text genuinely survived a real cy.go('back') after navigating away -- the identical input was completely wiped by cy.reload(), a real contrast."
---

# Navigation: visit, reload, go

## What was run

Three real tests exercising `cy.visit()`, `cy.go()`, and `cy.reload()` against the local test app, checking
exactly what state survives each kind of real navigation rather than assuming.

```javascript
it("checks whether typed text survives a real cy.go(back)", () => {
  cy.visit("/elements.html");
  cy.get("#text-input").type("will it survive back");
  cy.visit("/");
  cy.go("back");
  cy.get("#text-input").should("have.value", "will it survive back");
});

it("checks whether typed text survives a real cy.reload()", () => {
  cy.visit("/elements.html");
  cy.get("#text-input").type("will it survive reload");
  cy.reload();
  cy.get("#text-input").should("have.value", "");
});
```
```
✓ checks whether typed text survives a real cy.go(back) (324ms)
✓ checks whether typed text survives a real cy.reload() (166ms)
✓ navigates forward and back with cy.go (293ms)
```

Both tests passed — but they prove opposite things. Typed text genuinely survived navigating away and
coming back via `cy.go("back")`; the identical typed text was completely wiped by `cy.reload()`. This
matches this network's own [Selenium Mastery Path](https://sigilipelli.github.io/selenium-mastery-path/level-1/08-browser-navigation/)'s real finding using Selenium's `.back()` and `.refresh()` against an
analogous page — the same real browser-level behavior, now confirmed under Cypress's own navigation
commands too.

## How It Actually Works

`cy.go("back")` drives the real browser's native history navigation — the same mechanism as a person
clicking their browser's back button — and browsers commonly restore a form's in-memory DOM state
(including typed-but-unsubmitted input values) when navigating back to a page still held in session
history, rather than reloading it from scratch. `cy.reload()`, by contrast, genuinely tears down and
re-fetches the page from the server, which is why every real in-memory DOM state (the typed text, any JS
variables, this course's own `window.appInternalCounter` from [Lesson 1](01-what-cypress-actually-is.md))
is gone afterward — a fresh page load has no memory of the previous one. `cy.go("forward")` works the same
way as `"back"`, just traversing the real session history in the other real direction, confirmed in this
lesson's third test by checking `cy.url()` after each real navigation.

## Common mistakes

- Assuming `cy.reload()` and "navigate away then come back" are interchangeable ways to "reset" a page for
  the next test — they produce genuinely different real results, as this lesson's two contrasting tests
  demonstrate directly.
- Relying on back/forward history state surviving in a test, then being surprised when a *different*
  browser or a production deployment's caching behavior doesn't restore the same fields — browser history
  restoration isn't a Cypress feature or a guarantee; it's ordinary, somewhat implementation-specific
  browser behavior that this lesson happened to observe working in this environment.
- Using `cy.go(-1)` / `cy.go(1)` and the string forms `"back"` / `"forward"` inconsistently across a suite
  — both work, but picking one convention makes a suite's navigation intent easier to read at a glance.

## Exercise

Add a real `beforeunload` handler to `elements.html` that logs a message to the server via `fetch()`
whenever the page is about to be torn down, then trigger it with `cy.reload()` versus `cy.go("back")`.
Check the real server log afterward to determine, directly from observed evidence rather than assumption,
whether `cy.go("back")`'s history restoration in this environment counts as a real page teardown at all, or
something more like resuming a cached page.
