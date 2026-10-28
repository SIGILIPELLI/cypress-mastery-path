---
description: "Stubbing a real 500 response crashed the app with 'todos.forEach is not a function' -- cy.intercept exposed a real missing error-handling path."
---

# Network Stubbing With cy.intercept

## What was run

Three real `cy.intercept()` tests against the local todos page: fully stubbing a response with fake data,
spying on a real unstubbed request without changing it, and stubbing a simulated server error — the third
one producing a genuinely useful real failure.

```javascript
it("stubs the real /api/todos response with fake data", () => {
  cy.intercept("GET", "/api/todos", [
    { id: 999, text: "Stubbed todo, not from the real server", done: false },
  ]).as("getTodos");
  cy.visit("/todos.html");
  cy.wait("@getTodos");
  cy.get(".todo-item").should("have.length", 1);
  cy.contains("Stubbed todo, not from the real server").should("exist");
});

it("spies on the real unstubbed request without changing its response", () => {
  cy.intercept("GET", "/api/todos").as("getTodosReal");
  cy.visit("/todos.html");
  cy.wait("@getTodosReal").its("response.statusCode").should("eq", 200);
  cy.get(".todo-item").should("have.length", 2);
});
```
```
✓ stubs the real /api/todos response with fake data (93ms)
✓ spies on the real unstubbed request without changing its response (41ms)
```

The first test's real rendered list came entirely from the stub — `999`/`"Stubbed todo, not from the real
server"` never touched the real Express server at all. The second test proves `cy.intercept()` can observe
a request (confirmed via `cy.wait("@getTodosReal").its("response.statusCode")`) without stubbing its
response — the real server still answered, and the real 2 seed todos still rendered.

## A real bug cy.intercept's error-simulation genuinely exposed

```javascript
cy.intercept("GET", "/api/todos", { statusCode: 500, body: { error: "Simulated real server error" } }).as("getTodosError");
cy.visit("/todos.html");
cy.wait("@getTodosError").its("response.statusCode").should("eq", 500);
```
```
TypeError: The following error originated from your application code, not from Cypress.
  todos.forEach is not a function
at loadTodos (http://localhost:5151/todos.html:19:9)
```

This is not a Cypress bug — it's a real, genuine gap in the test app's own frontend code, caught precisely
*because* `cy.intercept()` made a real 500 response reachable in the first place. The real `loadTodos()`
function in `todos.html` always calls `res.json()` then `todos.forEach(...)`, with no check for whether the
response was actually an error object (`{ error: "..." }`) instead of a real todo array. Calling
`.forEach()` on that object crashed with a real, specific `TypeError`, and Cypress correctly flagged it as
an uncaught application exception rather than silently swallowing it.

## How It Actually Works

`cy.intercept()` sits between the real browser and the real network, matching requests by method and URL
pattern. Passing a static value (the fake todo array) as its third argument tells Cypress to respond
directly with that stub, never letting the real request reach the real server at all — confirmed here by
the stubbed todo appearing even though it was never seeded in the real Express app's in-memory `todos`
array. Omitting that third argument (the spy case) makes `cy.intercept()` purely observational: the real
request still goes to the real server, and `cy.wait("@alias")` lets a test pause until that real
request/response cycle actually completes, exposing its real status code and body for assertions. This dual
capability — stub or spy, same command — is Cypress's own defining network-testing feature, and it's what
let this lesson simulate a server condition (a real `500`) that the actual Express app was never asked to
produce, surfacing a real gap in frontend error handling that a happy-path test against the real server
would never have found.

## Common mistakes

- Treating every `cy.intercept()` as a stub — omitting the response argument (as in this lesson's spy test)
  is a real, distinct, valid use: observing and asserting on a real network call without altering it.
- Assuming a stubbed error response will be handled gracefully just because the real server would never
  send malformed data — this lesson's real crash proves the opposite: stubbing is exactly how you discover
  whether your frontend code *actually* handles conditions the real backend happens not to produce today.
- Forgetting to `.as()` an intercept and then trying to `cy.wait()` on it — `cy.wait()` needs a real alias
  name to know which intercepted request to pause for.

## Exercise

Fix the real bug this lesson found: add a check in `todos.html`'s `loadTodos()` for whether the parsed
response is actually an array before calling `.forEach()`, and render a real, visible error message
(`#todo-error`) when it isn't. Re-run this lesson's third test and confirm the real stubbed 500 now
produces a clean, asserted error state instead of an uncaught exception — directly closing the gap
`cy.intercept()` found.
