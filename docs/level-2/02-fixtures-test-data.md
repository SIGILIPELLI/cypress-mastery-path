---
description: "The real empty-credentials fixture case crashed with 'cy.type() cannot accept an empty string' -- discovered by running the test, not assumed, then fixed."
---

# Fixtures & Test Data

## What was run

A real 4-case JSON fixture file, loaded with `cy.fixture()` and iterated to drive the same login flow with
different real data — the same data-driven approach this network's own Selenium course used with plain JSON
files, now via Cypress's own built-in fixture loading.

```json
[
  { "name": "valid credentials", "username": "realuser", "password": "realpass123", "expected": "Login Successful" },
  { "name": "wrong password", "username": "realuser", "password": "wrongpass", "expected": "Login Failed" },
  { "name": "wrong username", "username": "wronguser", "password": "realpass123", "expected": "Login Failed" },
  { "name": "empty credentials", "username": "", "password": "", "expected": "Login Failed" }
]
```

```javascript
cy.fixture("login-cases.json").then((cases) => {
  expect(cases.length).to.eq(4);
  cases.forEach((c) => {
    cy.visit("/login.html");
    cy.get("#username").type(c.username);
    cy.get("#password").type(c.password);
    cy.get("#submit-btn").click();
    cy.get("#result").should("have.text", c.expected);
  });
});
```

## A real bug, caught on the very first run

```
CypressError: `cy.type()` cannot accept an empty string. You need to actually type something.
```

The "empty credentials" case's `username: ""` genuinely crashed the test — not because the login logic is
broken, but because `.type("")` is a real, deliberate Cypress restriction: typing nothing isn't a
meaningful user action, so Cypress refuses it outright rather than silently doing nothing. The real fix
was to skip the `.type()` call entirely when the fixture value is empty (the input already starts empty, so
there's nothing to type):

```javascript
if (c.username) cy.get("#username").type(c.username);
if (c.password) cy.get("#password").type(c.password);
```
```
✓ loads the real fixture file and iterates its real cases (1265ms)
1 passing (1s)
```

## How It Actually Works

`cy.fixture()` reads a real file from `cypress/fixtures/` and returns its parsed contents (JSON is parsed
automatically; other file types come back as text or base64) — it's a real file read, not a mocked or
in-memory data structure, which is why the fixture file genuinely needs to exist at that real path. Once
loaded, fixture data is just a normal JavaScript value — the `.forEach()` loop driving four separate real
login attempts isn't a Cypress-specific looping construct, it's ordinary JS iterating over real data that
happened to come from `cy.fixture()`. This lesson's real `.type("")` crash is a deliberate Cypress design
choice (not a fixture-specific bug) that exists specifically to catch tests that pass genuinely malformed
or empty data without the test author noticing — in this case, correctly flagging that "empty string"
needs different handling than "type this string," rather than silently treating them the same way.

## Common mistakes

- Assuming fixture data can always be passed straight into `.type()` without checking for empty values —
  this lesson's real crash is exactly what happens when a data-driven test's fixture includes an edge case
  (empty input) that needs different handling, not identical handling, to the normal cases.
- Hardcoding fixture file paths with a leading `cypress/fixtures/` — `cy.fixture("login-cases.json")`
  already resolves relative to that real directory; adding the prefix looks for a nonexistent nested path.
- Loading a fixture inside every single `it()` block when the data doesn't change between tests — this
  lesson's single `cy.fixture().then()` call loads the real file once and iterates all 4 real cases inside
  it, which is both faster and avoids 4 separate real file reads for identical data.

## Exercise

Add a fifth case to `login-cases.json` with `"password": null` instead of an empty string, and re-run this
lesson's test without additional changes. Determine, from the real observed Cypress error (or lack of one)
rather than assumption, whether `.type(null)` fails the same way `.type("")` did, differently, or not at
all — and adjust the `if (c.password)` guard if the real result shows it's needed.
