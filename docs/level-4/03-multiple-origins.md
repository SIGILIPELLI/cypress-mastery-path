---
description: "A real navigation to a different origin (port 5152) made the next command time out after 4000ms -- cy.origin() fixed it in 361ms."
---

# Multiple Origins & cy.origin

## What was run

A real second Express server, started on a genuinely different port (`localhost:5152`, a different real
origin from the main app's `localhost:5151`), to test exactly what happens when a Cypress test crosses an
origin boundary — first without any special handling, then with `cy.origin()`.

```javascript
it("fails without cy.origin() when visiting a genuinely different real origin", () => {
  cy.visit("http://localhost:5151/");
  cy.visit("http://localhost:5152/");
  cy.get("#second-origin-heading").should("exist");
});

it("works correctly using cy.origin() to cross the real origin boundary", () => {
  cy.visit("http://localhost:5151/");
  cy.origin("http://localhost:5152", () => {
    cy.visit("http://localhost:5152/");
    cy.get("#second-origin-heading").should("have.text", "Second Origin Page");
  });
});
```
```
1) fails without cy.origin() ...
   CypressError: Timed out retrying after 4000ms: The command was expected to run against origin
   `http://localhost:5151` but the application is at origin `http://localhost:5152`.
   Use `cy.origin()` to wrap the commands run on `http://localhost:5152`.

✓ works correctly using cy.origin() to cross the real origin boundary (361ms)
```

The real error names both origins explicitly and tells you the exact fix — and the `cy.origin()`-wrapped
version genuinely worked, in `361ms`, against the real second server.

## How It Actually Works

This is the real, concrete consequence of this course's own architectural thread
([Level 4, Lesson 1](01-why-architecture-is-different.md)): Cypress's test code runs *inside* the browser,
subject to the real browser's own same-origin security boundaries, the same way any ordinary page's own
JavaScript would be. After `cy.visit("http://localhost:5152/")` genuinely navigated the real browser tab
to the second origin, Cypress's own top-level test code — still running in the context of the *original*
origin — could no longer directly command what was happening in a different-origin page, exactly the
restriction a real browser enforces on any cross-origin script access. `cy.origin(url, callback)` works
around this by injecting the callback's commands to run *inside* the target origin's own context
specifically, rather than trying to reach across the boundary from outside it — which is why the exact
same `cy.get("#second-origin-heading")` call succeeds inside `cy.origin()` but times out waiting to
"reconnect" outside it.

Selenium and Playwright don't share this specific real constraint in the same way, because their client
code runs entirely outside any browser's JS context ([Level 1, Lesson 1](../level-1/01-what-cypress-actually-is.md)) — a wire-protocol client isn't itself bound by the same-origin
policy the way in-browser code genuinely is.

## Common mistakes

- Assuming a simple `cy.visit()` to a different real origin "just works" the way navigating in a normal
  browser tab does — this lesson's real timeout proves otherwise; Cypress's own command execution context
  doesn't automatically follow the navigation across origins.
- Wrapping too little inside `cy.origin()` — only commands genuinely needed to interact with the
  different-origin page belong inside the callback; commands meant for the original origin (including
  anything using an outer-scope JS variable, which `cy.origin()`'s callback can't directly close over)
  need to stay outside it.
- Confusing "different origin" with "different page on the same origin" — this course's own earlier
  multi-page navigation (`cy.visit()` between `/login.html` and `/elements.html` in
  [Level 1](../level-1/08-navigation.md)) never needed `cy.origin()`, because `localhost:5151` is the
  identical real origin throughout; the real boundary is scheme+host+port, not the specific path.

## Exercise

Change this lesson's second server to listen on the same port (`5151`) but a different real hostname
(`127.0.0.1` instead of `localhost`) rather than a different port, and re-run both tests unmodified except
for the URL. Determine from the real result whether Cypress's same-origin check treats `localhost` and
`127.0.0.1` as the same real origin or a genuinely different one — directly testing the actual boundary of
what counts as "cross-origin" here, rather than assuming hostname and loopback IP are always
interchangeable.
