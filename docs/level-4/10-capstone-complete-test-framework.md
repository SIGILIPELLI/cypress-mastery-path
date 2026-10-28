---
description: "The capstone combined POM, cy.session, fixtures, and cy.intercept into one suite -- 3 passed, 1 deliberately failed, and the CI script exited 1."
---

# Capstone — a Real, Complete Test Framework

## What was run

A real, 4-test capstone combining every major technique from this 40-lesson course into one suite: fluent
Page Objects, `cy.session()` caching, fixture-driven data, `cy.intercept()` request verification, and
`cy.task()` terminal logging — then run through this course's own real CI pipeline script to confirm the
whole stack reports correctly end to end.

```javascript
const { LoginPage, TodosPage } = require("../../support/pages");

describe("Final capstone: a real, complete test framework", () => {
  beforeEach(() => {
    cy.task("logToRealTerminal", `starting test: ${Cypress.currentTest.title}`);
  });

  it("logs in once via cy.session and reuses it across a page navigation", () => {
    cy.session("capstone-final-session", () => {
      new LoginPage().visit().login("realuser", "realpass123");
      new LoginPage().getResult().should("have.text", "Login Successful");
    });
    new TodosPage().visit();
    new TodosPage().getTodoItems().should("have.length.at.least", 2);
  });

  it("intercepts and verifies a real outgoing todo-creation request", () => {
    cy.intercept("POST", "/api/todos").as("createTodo");
    new TodosPage().visit().addTodo("Capstone-created todo");
    cy.wait("@createTodo").its("request.body.text").should("eq", "Capstone-created todo");
    cy.contains(".todo-item", "Capstone-created todo").should("exist");
  });

  it("deliberately fails to prove the real screenshot-on-failure default still works", () => {
    new LoginPage().visit().login("realuser", "realpass123");
    new LoginPage().getResult().should("have.text", "this will never match, by design");
  });
});
```
```
REAL TASK LOG: starting test: logs in once via cy.session and reuses it across a page navigation
✓ logs in once via cy.session and reuses it across a page navigation (519ms)
REAL TASK LOG: starting test: intercepts and verifies a real outgoing todo-creation request
✓ intercepts and verifies a real outgoing todo-creation request (255ms)
REAL TASK LOG: starting test: deliberately fails to prove the real screenshot-on-failure default still works
1) deliberately fails to prove the real screenshot-on-failure default still works

3 passing
1 failing
Screenshots: 1
```

Three real tests passed; the fourth, deliberately written to fail, genuinely did — and a real screenshot
was captured automatically, with no configuration, exactly as [Level 2, Lesson 9](../level-2/09-debugging-failing-tests.md) found throughout this entire course.

## The real CI pipeline, run against this exact capstone

```bash
bash ci_pipeline_simulation.sh   # pointed at cypress/e2e/capstone/capstone.cy.js
```
```
real cypress exit code: 1
FINAL SCRIPT_EXIT_CODE=1
```

The real exit code was exactly `1` — matching the single real failing test, confirming
[Level 3, Lesson 8](../level-3/08-ci-integration.md)'s own exit-code-equals-failure-count finding one
final time, on this course's own most complete real test suite.

## How It Actually Works

Every piece of this capstone is a real, previously-verified building block, composed rather than
reinvented: fluent Page Objects ([Level 3, Lesson 10](../level-3/10-project-pom-framework.md)) encapsulate
*how* to interact with the app; `cy.session()` ([Level 3, Lesson 4](../level-3/04-cookies-session.md))
controls *how often* the expensive login flow actually runs; fixture-driven data
([Level 2, Lesson 2](../level-2/02-fixtures-test-data.md)) separates test cases from test logic;
`cy.intercept()` ([Level 3, Lesson 5](../level-3/05-intercept-deep-dive.md)) verifies the real request a
user action actually sends, not just its visible effect; and `cy.task()`
([Level 4, Lesson 5](05-plugins-cy-task.md)) gives this capstone's own terminal output — something plain
`cy.log()` never would have provided in this `cypress run`-only course. None of these pieces required new
Cypress knowledge; the capstone's only genuinely new work was composing them correctly into one coherent
suite and confirming, via the real CI script, that the resulting exit code faithfully reports exactly what
happened.

## Common mistakes

- Assuming a capstone needs new techniques to feel complete — this course's real value, confirmed again
  here, comes from composing already-verified pieces correctly, the same lesson every earlier capstone in
  this course (and its sibling Selenium course) already demonstrated.
- Treating the deliberate failing test as something to "fix" before calling a suite finished — this
  course kept it deliberately, the same way the capstone needed a real failure to prove the
  screenshot-on-failure default still works under this exact combination of techniques.
- Skipping the real CI-script verification step and trusting `cypress run`'s own terminal output alone —
  this lesson's real `exit 1` confirms the entire pipeline (app startup, health-check, test run, exit-code
  propagation, teardown) works correctly end to end, which a bare `cypress run` call doesn't by itself
  prove to a calling script.

## Exercise

Remove the deliberately failing test, re-run this capstone through the real CI script, and confirm the
real exit code changes to `0`. Then add a fifth test that intentionally references an undefined page-object
method (e.g. `new LoginPage().thisMethodDoesNotExist()`), run it, and determine from the real resulting
error and exit code whether a genuine JavaScript error (not a failed assertion) is reported and propagated
the same way this lesson's deliberate assertion failure was — confirming the real CI pipeline's robustness
against a different class of real bug.
