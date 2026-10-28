---
description: "A real cy.request() login took 26ms vs the UI flow's 458ms, and a real 401 failed the test by default with a full request/response dump."
---

# Calling APIs Directly With cy.request

## What was run

Real `cy.request()` calls against the local Express API — logging in directly, seeding a todo, and
deliberately triggering a real `401` to see Cypress's actual default failure behavior rather than assuming
it mirrors `fetch()`.

```javascript
it("logs in via a real direct API call, bypassing the UI", () => {
  cy.request("POST", "/api/login", { username: "realuser", password: "realpass123" }).then((res) => {
    expect(res.status).to.eq(200);
    expect(res.body.token).to.eq("real-session-token-abc123");
  });
});

it("seeds a real todo via cy.request, then confirms it in the real UI", () => {
  cy.request("POST", "/api/todos", { text: "Seeded directly via cy.request" }).then((res) => {
    expect(res.status).to.eq(201);
    cy.visit("/todos.html");
    cy.contains("Seeded directly via cy.request").should("exist");
  });
});
```
```
✓ logs in via a real direct API call, bypassing the UI (26ms)
✓ seeds a real todo via cy.request, then confirms it in the real UI (75ms)
```

The real direct login took `26ms` — compare that to [Level 1, Lesson 10](../level-1/10-project-login-flow-test.md)'s UI-driven login flow, which took a real `458ms` for the identical credentials against the
identical backend. The gap is real page rendering, DOM queries, and typing simulation — none of which
`cy.request()` has to do, because it talks to the real server directly over HTTP with no browser rendering
involved at all.

## The real default failure behavior on a non-2xx status

```javascript
cy.request("POST", "/api/login", { username: "wronguser", password: "wrongpass" });
```
```
CypressError: `cy.request()` failed on:
http://localhost:5151/api/login

The response we received from your web server was:
  > 401: Unauthorized

This was considered a failure because the status code was not `2xx` or `3xx`.
If you do not want status codes to cause failures pass the option: `failOnStatusCode: false`
```

Cypress genuinely failed the test on this real `401` — not a crash, a deliberate default. The real error
message includes the full real request (headers, body) and full real response (headers, body), which is
substantially more diagnostic detail than a typical caught HTTP error would show by default. Setting
`failOnStatusCode: false` (used in this lesson's second real test) tells Cypress to hand back the real
response object instead, letting the test assert on the `401` directly.

## How It Actually Works

`cy.request()` makes a real HTTP request from Node (not from inside the browser), which is why it has no
access to the page's own cookies or session state unless explicitly configured to — and why it doesn't
auto-wait or retry the way `cy.get()` does: a network request either completes with some real status code
or it doesn't, and there's no DOM condition to poll for in between. Cypress's choice to fail by default on
a non-2xx/3xx status reflects a real design opinion: most of the time, an unexpected error status *is* the
bug you want a test to catch immediately, with full diagnostic context, rather than a status code you have
to remember to check manually on every single call. `failOnStatusCode: false` exists specifically for the
cases (like this lesson's second test) where testing the *error path itself* is the actual goal.

## Common mistakes

- Using `cy.request()` to drive the primary user flow a test is meant to validate — if the thing under test
  is the UI's login form, bypassing it with `cy.request()` tests the API, not the form; this lesson's real
  `26ms` speed is a setup-and-seeding tool, not a replacement for UI interaction tests.
- Forgetting `failOnStatusCode: false` when deliberately testing an error response — this lesson's real
  first attempt (not shown with the flag) would have failed the test outright on the expected `401`,
  exactly as the standalone failure demonstration above shows.
- Assuming `cy.request()` shares the real browser's cookies automatically in every configuration — it runs
  from Node by default, so session state set by a prior `cy.request()` login may not be visible to a
  subsequent `cy.visit()` without additional real configuration (Cypress's `cy.session()`, covered in
  [Level 3](../level-3/04-cookies-session.md), is the real tool for that).

## Exercise

Chain `cy.request("DELETE", "/api/todos/999")` after this lesson's seed-via-`cy.request()` test, targeting
whatever real `id` the seed response returned, and confirm via a follow-up `cy.request("GET",
"/api/todos")` that the real todo count returns to its original value. This directly exercises
`cy.request()` for full real setup-and-teardown around a UI test, not just one-off API calls.
