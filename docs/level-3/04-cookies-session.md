---
description: "The first real cy.session() login took 480ms; two later tests reusing the cached session took 38ms and 26ms -- a real ~15-18x measured speedup."
---

# Cookies, Local Storage & cy.session

## What was run

Real `localStorage` access via `cy.window()`, and three real tests sharing one `cy.session()` call to
measure the actual speedup session caching provides, rather than assuming it from the feature's name.

```javascript
function loginSession() {
  cy.session("shared-session", () => {
    cy.visit("/login.html");
    cy.get("#username").type("realuser");
    cy.get("#password").type("realpass123");
    cy.get("#submit-btn").click();
    cy.get("#result").should("have.text", "Login Successful");
  });
}

it("test A: runs the real login callback for the first time this session", () => {
  loginSession();
  cy.visit("/elements.html");
});

it("test B: reuses the cached real session instead of logging in again", () => {
  loginSession();
  cy.visit("/elements.html");
});
```
```
✓ test A: runs the real login callback for the first time this session (480ms)
✓ test B: reuses the cached real session instead of logging in again (38ms)
✓ test C: reuses the cached real session a second time (26ms)
```

Test A paid the full real cost of the login UI flow: `480ms`. Tests B and C, calling the exact same
`cy.session("shared-session", ...)` with the identical setup function, took `38ms` and `26ms` — a real,
measured **~15-18x speedup** from not re-running the login UI at all.

## How It Actually Works

`cy.session(name, setupFn)` runs `setupFn` only the first time a given session name is used within a spec
run; every later call with the same name instead restores the real browser state (cookies, `localStorage`,
`sessionStorage`) that was captured right after the first run, skipping the setup function entirely. This
is conceptually the same real motivation as this network's own
[Selenium Mastery Path](https://sigilipelli.github.io/selenium-mastery-path/level-3/05-pytest-fixtures-for-selenium/), which measured a real `42%` speedup from a session-scoped browser fixture — but `cy.session()`
caches actual *application login state*, not just the browser process, which is why this lesson's speedup
(~15-18x) is dramatically larger: it isn't just skipping a browser launch, it's skipping the entire real
UI-driven login flow every single test after the first.

`cy.session()`'s real captured state survives a `cy.visit()` to a different page in the same real origin
(confirmed by this lesson's own `cy.visit("/elements.html")` calls after each `loginSession()`), because
the relevant cookies/storage belong to the origin, not the specific page. It does **not** survive across
separate spec files by default — each spec run starts fresh, which keeps tests independent of execution
order at the file level, while still letting tests *within* one file share expensive setup.

## Common mistakes

- Calling `cy.session()` with a setup function that has side effects beyond what you want cached (e.g. one
  that also seeds unrelated server data) — only the real browser state is cached and restored; any
  server-side effects from the setup function happen again on the *first* call only, which can create
  subtle real differences between the first test using a session and later ones.
- Assuming `cy.session()` needs manual `cy.clearCookies()`/`cy.clearLocalStorage()` cleanup between tests —
  it manages its own real validation and restoration; manually clearing storage before calling it again
  can defeat the caching this lesson's own `480ms` → `38ms` measurement depends on.
- Using a different session name per test "to be safe" — that guarantees the setup function reruns every
  time, which is the exact real cost this lesson's comparison shows `cy.session()` exists specifically to
  avoid.

## Exercise

Add a fourth test that calls `cy.clearAllSessionStorage()` immediately after `loginSession()`, then
re-visits `/elements.html` and checks whether the real session-authenticated state (this lesson's own
`localStorage` marker, or any cookie your own real login sets) survives. Determine from the real result
whether `cy.session()`'s cache is actually re-validated against current browser state on each use, or
whether it blindly restores a stale snapshot regardless of what's been manually cleared since.
