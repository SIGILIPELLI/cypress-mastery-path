---
description: "The identical dynamic cy.intercept handler had zero effect on a real cy.request() call but fully controlled the same request via the real browser."
---

# cy.intercept Deep Dive: Stub vs Spy

## What was run

Deeper real `cy.intercept()` capabilities beyond the basic stub/spy split from
[Level 2, Lesson 3](../level-2/03-network-stubbing-intercept.md): inspecting a real outgoing request body,
delaying a real response to observe a loading state, and returning a dynamically computed response — plus
a genuine real discovery about exactly which requests `cy.intercept()` can actually see.

```javascript
it("inspects a real outgoing request body before letting it continue", () => {
  cy.intercept("POST", "/api/todos", (req) => {
    expect(req.body.text).to.eq("real request body check");
    req.continue();
  }).as("createTodo");
  cy.visit("/todos.html");
  cy.get("#new-todo").type("real request body check");
  cy.get("#add-btn").click();
  cy.wait("@createTodo");
});

it("delays a real response to observe the real loading state", () => {
  cy.intercept("GET", "/api/todos", (req) => {
    req.on("response", (res) => { res.setDelay(800); });
  }).as("delayedTodos");
  cy.visit("/todos.html");
  cy.get("#loading").should("be.visible");
  cy.wait("@delayedTodos");
  cy.get("#loading").should("not.be.visible");
});
```
```
✓ inspects a real outgoing request body before letting it continue (305ms)
✓ delays a real response to observe the real loading state (855ms)
```

The real `800ms` artificial delay genuinely made `#loading` visible for that entire window, then hide once
the delayed response actually arrived — a direct, real test of a loading-state UI pattern that's otherwise
awkward to trigger reliably against a fast real local server.

## A real, important discovery: cy.intercept doesn't see cy.request at all

```javascript
it("finds cy.intercept has no effect on cy.request, which bypasses the browser entirely", () => {
  cy.intercept("POST", "/api/login", (req) => {
    req.reply({ success: true, token: "dynamic-token-should-not-apply-here" });
  }).as("dynamicLogin");
  cy.request({ method: "POST", url: "/api/login", body: { username: "realuser", password: "anything" }, failOnStatusCode: false })
    .then((res) => { expect(res.status).to.eq(401); });
});

it("returns a real dynamic response when the SAME request goes through the real browser", () => {
  cy.intercept("POST", "/api/login", (req) => {
    if (req.body.username === "realuser") req.reply({ success: true, token: "dynamic-token-for-realuser" });
    else req.reply({ success: false, message: "dynamic rejection" });
  }).as("dynamicLoginUI");
  cy.visit("/login.html");
  cy.get("#username").type("realuser");
  cy.get("#password").type("anything");
  cy.get("#submit-btn").click();
  cy.wait("@dynamicLoginUI");
  cy.get("#result").should("have.text", "Login Successful");
});
```
```
✓ finds cy.intercept has no effect on cy.request, which bypasses the browser entirely (25ms)
✓ returns a real dynamic response when the SAME request goes through the real browser (378ms)
```

The *exact same* intercept handler, registered the exact same way, had **zero effect** on the
`cy.request()` call — the real server genuinely rejected it with a real `401` (the password really was
wrong) — but fully controlled the identical login request when it went through the real browser's own
`fetch()` call instead. This wasn't assumed; it was found by first writing the dynamic-intercept test
against `cy.request()`, watching it fail with the real server's own real `401`, and investigating why.

## How It Actually Works

`cy.intercept()` registers its handler against the real browser's own network layer — it genuinely
intercepts `fetch()`/`XMLHttpRequest` calls made from inside the page. `cy.request()`, confirmed back in
[Lesson 4 of Level 2](../level-2/04-cy-request.md), makes its real HTTP call from Node, entirely outside
the browser — so there's no browser-level network event for `cy.intercept()` to ever catch. This is a real,
load-bearing architectural boundary, not an edge case: any test combining `cy.request()` for setup with
`cy.intercept()` for stubbing needs to know which of its calls actually goes through the browser, or a stub
meant to apply everywhere will silently apply to half of what the test expected.

The request-inspection and dynamic-response capabilities themselves (`req.continue()`, `req.reply()`,
`res.setDelay()`) all operate on that same real, intercepted browser-level traffic — `req.continue()` lets
the real request proceed to the real server after inspection (as in the first test), while `req.reply()`
replaces the response entirely, computed from whatever the real request's own body or headers contain.

## Common mistakes

- Assuming a `cy.intercept()` stub applies universally once registered, regardless of how a later request
  is made — this lesson's own real result shows it depends entirely on *how* that request travels (browser
  fetch vs. Node-side `cy.request()`), not just the URL pattern matching.
- Using `req.reply(staticValue)` when the response genuinely needs to depend on the real request's content
  — a static stub can't express "different users get different responses," which this lesson's dynamic
  `req.body.username` check handles correctly.
- Testing a real loading spinner against an unmodified fast local server and concluding it "doesn't work"
  when it never had time to actually appear — this lesson's real `setDelay(800)` is the correct tool for
  deliberately slowing a response just long enough to observe a transient UI state.

## Exercise

Add a `req.on("response", (res) => { res.setDelay(5000); })` to the todos intercept and re-run the
loading-state test with the real app's existing code unmodified. Determine from the real result whether the
test's own `cy.wait("@delayedTodos")` (which uses the suite's default command timeout) needs an explicit
longer timeout to avoid a false failure — directly connecting this lesson's delay mechanism back to
[Level 1, Lesson 6](../level-1/06-auto-waiting-retry-ability.md)'s real configurable timeout ceiling.
