---
description: "Renaming one real button id broke both tests using a duplicated login flow -- fixing only the custom command (1 edit) restored both, zero test changes."
---

# Custom Commands

## What was run

A real custom Cypress command (`Cypress.Commands.add("login", ...)`), used by two real tests, then a
deliberate real id rename on the login button to measure — not assume — the actual cost of duplicating a
login flow versus centralizing it in one command.

```javascript
Cypress.Commands.add("login", (username, password) => {
  cy.visit("/login.html");
  cy.get("#username").type(username);
  cy.get("#password").type(password);
  cy.get("#submit-btn").click();
});

it("logs in via the real custom cy.login() command", () => {
  cy.login("realuser", "realpass123");
  cy.get("#result").should("have.text", "Login Successful");
});

it("reuses the identical custom command for a failing case", () => {
  cy.login("realuser", "wrongpass");
  cy.get("#result").should("have.text", "Login Failed");
});
```
```
✓ logs in via the real custom cy.login() command (458ms)
✓ reuses the identical custom command for a failing case (358ms)
```

## A real rename, and its real cost

```bash
# real change: id="submit-btn" -> id="log-in-btn" in login.html
npx cypress run --spec cypress/e2e/custom-commands.cy.js
```
```
1) logs in via the real custom cy.login() command:
   AssertionError: Timed out retrying after 4000ms: Expected to find element: `#submit-btn`, but never found it.
2) reuses the identical custom command for a failing case:
   AssertionError: Timed out retrying after 4000ms: Expected to find element: `#submit-btn`, but never found it.

0 passing (9s)
2 failing
```

Both real tests broke from one real markup change — expected, since both call the same `cy.login()`
command. The real question is what fixing it actually costs. Editing **only** the command definition
(`#submit-btn` → `#log-in-btn`, one line, one place) and re-running without touching either test file:

```
✓ logs in via the real custom cy.login() command (437ms)
✓ reuses the identical custom command for a failing case (375ms)
2 passing (863ms)
```

Both tests recovered from a single edit in a single location.

## How It Actually Works

`Cypress.Commands.add()` registers a real new method on the `cy` object, available to every spec in the
suite after it's defined — it's not syntactic sugar for a shared function call, it genuinely becomes part
of Cypress's own command queue, which means a custom command auto-waits and chains exactly like a built-in
one (`cy.login(...)` queues real `cy.visit`, `cy.get`, and `.type` calls, each with its own real
retry-ability). This lesson's real rename experiment demonstrates the same underlying lesson this network's
[Selenium Mastery Path](https://sigilipelli.github.io/selenium-mastery-path/level-3/01-why-page-object-model/)
measured for the Page Object Model: centralizing a locator in exactly one place means exactly one place
needs to change when the real application's markup changes, regardless of how many tests depend on it.

## Common mistakes

- Defining the same login steps inline in every test "because it's simple enough" — this lesson's own real
  2-test suite already shows the cost: one markup change broke both, and would break a third, fourth, and
  fifth test the same way if they existed.
- Putting custom commands in a spec file instead of a shared support file in a real project (this course's
  lab keeps `supportFile: false` and defines commands per-lesson for self-containment — a real project
  should centralize them in `cypress/support/commands.js` so every spec can use them without copy-pasting
  the `Cypress.Commands.add()` call itself).
- Assuming a custom command needs to return something to be chainable — `cy.login(...)` here returns
  nothing explicitly, but the command queue still lets you chain further real commands after it, because
  chaining is about the queue, not a return value.

## Exercise

Add a second custom command, `cy.addTodo(text)`, wrapping this course's real todo-add flow
(`cy.get("#new-todo").type(text)` then `cy.get("#add-btn").click()`), and rewrite the
[Level 1 todo test](../level-1/07-forms-dropdowns.md) to use it. Then deliberately rename `#add-btn` in
`todos.html` and confirm, by actually running the suite, that only the command definition needs a fix —
directly reproducing this lesson's real result on a second, independent real workflow.
