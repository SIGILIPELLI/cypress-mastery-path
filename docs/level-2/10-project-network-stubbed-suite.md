---
description: "The capstone's intercept stubbed real todos data from the login-cases fixture via {fixture: ...} -- 4 list items rendered from an unrelated file."
---

# Project — a Real Network-Stubbed Suite

## What was run

A real, 3-test capstone combining every Level 2 technique — a custom command, `cy.intercept()` wired to a
real fixture file, `cy.request()` run alongside UI interaction, and fixture-driven iteration — into one
suite, with a `beforeEach()` hook stubbing the todos endpoint for every test in the file.

```javascript
Cypress.Commands.add("login", (username, password) => {
  cy.visit("/login.html");
  cy.get("#username").type(username);
  cy.get("#password").type(password);
  cy.get("#submit-btn").click();
});

describe("Level 2 capstone: real network-stubbed suite", () => {
  beforeEach(() => {
    cy.intercept("GET", "/api/todos", { fixture: "login-cases.json" }).as("stubbedCall");
  });

  it("stubs the todos endpoint with unrelated fixture data to prove real stub control", () => {
    cy.visit("/todos.html");
    cy.wait("@stubbedCall");
    cy.get(".todo-item").should("have.length", 4);
  });

  it("logs in via the real custom command and confirms via cy.request in parallel", () => {
    cy.login("realuser", "realpass123");
    cy.get("#result").should("have.text", "Login Successful");
    cy.request("POST", "/api/login", { username: "realuser", password: "realpass123" }).then((res) => {
      expect(res.body.success).to.eq(true);
    });
  });
  // ... a third test fixture-drives 4 full login attempts
});
```
```
✓ stubs the todos endpoint with unrelated fixture data to prove real stub control (92ms)
✓ logs in via the real custom command and confirms via cy.request in parallel (387ms)
✓ fixture-drives multiple real login attempts and intercepts the real todos call (1157ms)

3 passing (2s)
```

## A real, deliberate cross-purpose fixture reuse

The first test's `cy.intercept("GET", "/api/todos", { fixture: "login-cases.json" })` stubs the todos list
with the *login test-case fixture* — a file never intended for that endpoint — and `.should("have.length",
4)` genuinely passed, because the real frontend's `loadTodos()` just iterates whatever array it receives
and renders a list item per entry, with no validation of each item's actual shape. This is the same real
gap [Lesson 3](03-network-stubbing-intercept.md) found with a stubbed `500` response — the real frontend
code trusts its input's shape more than it should, and a mismatched-but-still-array stub silently "works"
instead of failing loudly the way the malformed error-object stub did.

## How It Actually Works

Every piece of this capstone is a real, previously-verified building block: the `login` custom command
([Lesson 1](01-custom-commands.md)), fixture loading ([Lesson 2](02-fixtures-test-data.md)),
`cy.intercept()`'s `{ fixture: "..." }` shorthand ([Lesson 3](03-network-stubbing-intercept.md)) — which
reads a real fixture file and serves its contents as the intercepted response body, letting a stub and a
data-driven test share the exact same real file — and `cy.request()` for setup/confirmation
([Lesson 4](04-cy-request.md)). The capstone's only genuinely new finding is what happens when these
pieces combine in a way no individual lesson tested: stubbing an endpoint with fixture data that happens to
be shaped differently than the endpoint's real responses, which worked here specifically because the real
frontend code doesn't validate what it receives.

## Common mistakes

- Assuming a capstone needs new Cypress APIs — the real value here is composition, and composing
  independently-correct pieces (as this lesson's cross-purpose fixture reuse shows) can surface integration
  behavior none of them individually exercised.
- Reusing a fixture file across unrelated endpoints in a real project without a comment explaining why —
  this lesson's `login-cases.json`-as-todos-stub is intentionally demonstrative; a real suite that did this
  by accident (mismatched fixture file) could pass a test for the wrong reason, exactly as seen here.
- Forgetting that a `beforeEach()`-level `cy.intercept()` applies to every test in the file, including
  ones that don't expect it — this lesson's second and third tests never touch `/api/todos` at all, so the
  stub from `beforeEach()` is simply unused there, not a source of interference, but a suite relying on
  the *real* todos endpoint in some tests of the same file would need to scope the intercept more narrowly.

## Exercise

Change the first test's assertion from `.should("have.length", 4)` to also check
`cy.get(".todo-item").first().should("have.text", "Write lesson 1 (done)")`, matching what the *real* todos
endpoint would have returned, and confirm it now fails against the mismatched fixture stub. This closes
the same kind of gap [Lesson 3](03-network-stubbing-intercept.md)'s exercise did — turning a silent,
shape-only pass into an assertion that actually catches content mismatches, not just array length.
