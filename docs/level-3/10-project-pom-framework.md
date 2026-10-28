---
description: "A real fluent Page Object class combined with cy.session passed 3/3 real tests, confirming the Level 3 architecture composes correctly."
---

# Project — a Real POM Framework With Custom Commands

## What was run

A real, fluent Page Object module (plain JS classes, not a Cypress-specific API) combined with
`cy.session()` caching, run as a 3-test capstone against the local test app.

```javascript
// cypress/support/pages.js
class LoginPage {
  visit() { cy.visit("/login.html"); return this; }
  fillUsername(text) { cy.get("#username").type(text); return this; }
  fillPassword(text) { cy.get("#password").type(text); return this; }
  submit() { cy.get("#submit-btn").click(); return this; }
  login(username, password) { return this.fillUsername(username).fillPassword(password).submit(); }
  getResult() { return cy.get("#result"); }
}
```

```javascript
it("logs in via a fluent page object chain", () => {
  new LoginPage().visit().login("realuser", "realpass123");
  new LoginPage().getResult().should("have.text", "Login Successful");
});

it("logs in and reuses a cached cy.session instead of repeating the UI flow", () => {
  cy.session("capstone-session", () => {
    new LoginPage().visit().login("realuser", "realpass123");
    new LoginPage().getResult().should("have.text", "Login Successful");
  });
  new TodosPage().visit();
  new TodosPage().getTodoItems().should("have.length.at.least", 2);
});
```
```
✓ logs in via a fluent page object chain (433ms)
✓ logs in and reuses a cached cy.session instead of repeating the UI flow (442ms)
✓ adds a real todo via the page object and confirms it renders (242ms)

3 passing (1s)
```

## How It Actually Works

This capstone's Page Object classes are *not* a Cypress feature — they're ordinary JavaScript classes
whose methods call real `cy` commands and `return this` for fluent chaining, the identical pattern this
network's own [Selenium Mastery Path](https://sigilipelli.github.io/selenium-mastery-path/level-3/03-fluent-page-objects/) used in Python. That portability is itself a real, useful fact: the Page
Object *pattern* is language- and tool-agnostic, built entirely from each language's own object-oriented
features, while the methods inside it call whatever commands the specific tool (Cypress here, Selenium
there) actually provides. The one genuinely Cypress-specific piece in this capstone is wrapping the login
flow in `cy.session()` ([Lesson 4](04-cookies-session.md)) — a page object encapsulates *how* to perform
the login, while `cy.session()` controls *how often* that encapsulated flow actually needs to run, and the
two compose cleanly because the session's setup callback is just calling the identical page-object methods
other tests call directly.

## Common mistakes

- Putting `cy.session()` calls inside the Page Object class itself — this capstone keeps session caching
  at the test level, wrapping a call to the page object's own `login()` method, which keeps the page object
  focused purely on "how to interact with this page" and leaves "how often to repeat this" as the test's
  own decision.
- Returning a raw Cypress chain (`cy.get(...)`) from a page-object method meant for fluent chaining (like
  `login()`), instead of `return this` — mixing the two return styles in the same class makes it unclear
  from the call site whether a chained `.method()` call continues the page object or a `cy` command.
- Assuming a Page Object needs to wrap *every* interaction — `getResult()` in this lesson's `LoginPage`
  deliberately returns the raw `cy.get(...)` chain rather than wrapping it in another method, because the
  caller needs Cypress's own assertion chain (`.should(...)`) available directly, not a page-object
  abstraction over it.

## Exercise

Add a `logout()` method to `LoginPage` that clears the real session (`cy.clearAllSessionStorage()` plus a
fresh `cy.visit("/login.html")`), and write a fourth capstone test that logs in via the cached
`cy.session()`, calls the new `logout()`, then attempts to visit `/todos.html` again. Determine from the
real result whether this test app's own current code actually enforces any authentication requirement on
that route, or whether — like several of this course's earlier real findings — the app simply doesn't
check, revealing a genuine gap rather than a Cypress limitation.
