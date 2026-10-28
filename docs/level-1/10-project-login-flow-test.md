---
description: "The 3-test real login-flow capstone passed 3/3 on its first run, zero manual waits -- opposite of Selenium's capstone, which hit a real race condition."
---

# Project — a Real Login Flow Test

## What was run

A real, 3-test Cypress suite combining every technique from this level — `.type()`, `.click()`,
`.should()`, `cy.visit()` — into one small, genuine login-flow project against the local test app, with
zero manual waits anywhere in the code.

```javascript
it("rejects an invalid login with the real error message", () => {
  cy.visit("/login.html");
  cy.get("#username").type("wronguser");
  cy.get("#password").type("wrongpass");
  cy.get("#submit-btn").click();
  cy.get("#result").should("have.text", "Login Failed");
});

it("logs in successfully and the result reflects it immediately", () => {
  cy.visit("/login.html");
  cy.get("#username").type("realuser");
  cy.get("#password").type("realpass123");
  cy.get("#submit-btn").click();
  cy.get("#result").should("have.text", "Login Successful");
});

it("navigates from login to the elements page and confirms a fresh state", () => {
  cy.visit("/login.html");
  cy.get("#username").type("realuser");
  cy.get("#password").type("realpass123");
  cy.get("#submit-btn").click();
  cy.get("#result").should("have.text", "Login Successful");
  cy.visit("/elements.html");
  cy.get("#text-input").should("have.value", "");
});
```
```
✓ rejects an invalid login with the real error message (458ms)
✓ logs in successfully and the result reflects it immediately (373ms)
✓ navigates from login to the elements page and confirms a fresh state (378ms)

3 passing (1s)
```

## The real, direct contrast with this network's Selenium capstone

This network's own [Selenium Mastery Path](https://sigilipelli.github.io/selenium-mastery-path/level-1/10-project-login-flow-test/) capstone — an almost identical real login-flow test — hit a genuine
`NoSuchElementException` race condition on its very first real run, because `find_element()` checked the
real DOM once, immediately after a click, before the app's real response had finished rendering. This
lesson's Cypress capstone hit **no such failure**, on the first real run, with no manual wait of any kind.
That difference isn't luck — it's [Lesson 6](06-auto-waiting-retry-ability.md)'s auto-waiting doing exactly
what it's designed to do: `.should("have.text", "Login Failed")` genuinely retried until the real
`fetch()`-driven DOM update finished, the same way it retried for the real `1547ms` delay measured in
[Lesson 5](05-assertions.md).

## How It Actually Works

Every command in this capstone is one already covered individually in this level — what's new here is
only the composition: a full real user flow (type credentials, submit, read the result, navigate again)
built entirely from commands that each auto-wait on their own. That compounding matters: if even one
command in a chain didn't retry, the whole flow would be only as reliable as its weakest link, exactly as
Selenium's sibling capstone discovered the hard way. Cypress's retry-ability isn't a feature you opt into
per-test — it's the default behavior of `cy.get()` and `.should()`, which is why a first-draft script with
zero waits already behaved correctly here.

## Common mistakes

- Concluding from this lesson's clean first run that Cypress tests are simply immune to failure — they're
  immune to exactly the race-condition category Selenium's capstone hit, not to every kind of flakiness;
  [Level 3](../level-3/index.md) of this course covers the specific situations where Cypress's own
  retry-ability has real limits.
- Writing a Cypress test the way you'd write a Selenium one out of habit — adding `cy.wait(1000)` after
  every action "just in case" undoes exactly the reliability auto-waiting already provides, and only slows
  the suite down.
- Treating this capstone's 3 tests as fully independent — each one re-visits `/login.html` and logs in
  itself rather than relying on a previous test's state, which is deliberate: Cypress test order isn't
  guaranteed to matter for isolation the way shared browser state might suggest.

## Exercise

Add a fourth test to this capstone that logs in successfully, then immediately (same test, no `cy.visit()`
in between) tries to submit the login form a second time without reloading the page. Run it against the
real app and determine, from the real observed result rather than assumption, whether the real Express
backend and the real page's own JS genuinely support a second submission attempt, or whether the first
successful response already replaced the form element the second attempt needs.
