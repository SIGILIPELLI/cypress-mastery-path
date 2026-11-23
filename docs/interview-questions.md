---
title: "Cypress Interview Questions and Answers (179 Questions)"
description: "179 Cypress interview questions with answers and explanations, from beginner to advanced, organised by level. Free, from the Cypress Mastery Path."
---

# Cypress Interview Questions and Answers

**179 questions** with the correct answer and a short explanation, organised from beginner (Level 1) to advanced (Level 4). Every question comes from a lesson in the free Cypress Mastery Path, so you can open the lesson to go deeper.

!!! tip "Practise, don't just read"
    Each lesson also ends with an interactive quiz, and the [final quiz](quiz.md) tests all four levels. Try to answer each question below before reading the answer.

## Level 1 · Cypress Fundamentals

### [What Cypress Actually Is](level-1/01-what-cypress-actually-is.md)

**Q1.** What is Cypress's defining architectural choice?

**Answer:** Test code runs inside the same browser as the application under test

That gives direct access to the app's window and network layer.

**Q2.** Which kinds of tests is Cypress built for?

**Answer:** End-to-end tests of web apps (and component tests) written in JavaScript/TypeScript

It targets the browser.

**Q3.** Why do Cypress tests have direct access to the page's window object?

**Answer:** Because the test code executes in the same browser context as the app

Observable consequence of the architecture.

**Q4.** What's a downside of running inside the browser?

**Answer:** Constraints like same-origin rules and limited multi-tab support

Later lessons cover these limits.

### [Installing &amp; Configuring Cypress](level-1/02-installing-configuring.md)

**Q5.** What does `npm install cypress --save-dev` do?

**Answer:** Installs Cypress as a dev dependency of the project

The binary is downloaded on install.

**Q6.** Where do you set baseUrl and spec patterns?

**Answer:** cypress.config.js (or .ts)

Configuration lives there.

**Q7.** What does `baseUrl` let you do?

**Answer:** Use relative paths like cy.visit('/login')

Cleaner tests.

**Q8.** Why verify configuration by running tests rather than trusting defaults?

**Answer:** Defaults and docs can differ from actual behavior on your setup

The lesson tests what config actually controls.

### [Querying the DOM: cy.get, cy.contains, cy.find](level-1/03-querying-the-dom.md)

**Q9.** What does cy.get('.item') do?

**Answer:** Queries the DOM for elements matching the selector, retrying until found or timeout

Queries retry.

**Q10.** How does cy.contains('Login') differ from cy.get?

**Answer:** It finds an element by its text content

Useful for visible text.

**Q11.** What does `.find()` do?

**Answer:** Searches for descendants within the previously yielded subject

Chain from a parent.

**Q12.** What does Cypress show when a selector isn't found?

**Answer:** A specific timed-out error saying it failed to find the element after retrying

The lesson captures the real message.

### [Interacting With Elements: Type, Click, Check](level-1/04-interacting-with-elements.md)

**Q13.** What does .type('hello') do?

**Answer:** Types text into the focused or yielded input

Supports special keys like {enter}.

**Q14.** What happens when you .click() a disabled button?

**Answer:** Cypress fails by default, because it checks actionability

Use {force:true} only knowingly.

**Q15.** What does .check() act on?

**Answer:** Checkboxes and radio buttons

.uncheck() clears.

**Q16.** Why does Cypress check actionability before clicking?

**Answer:** To act like a real user: element must be visible, enabled and not covered

Avoids false passes.

### [Assertions &amp; should()](level-1/05-assertions.md)

**Q17.** What does `cy.get('h1').should('have.text','Todos')` do?

**Answer:** Retries until the heading's text matches or timeout

should is retryable.

**Q18.** When is `expect()` appropriate?

**Answer:** On values already extracted, e.g. inside .then()

expect doesn't retry.

**Q19.** What does `.and()` do?

**Answer:** Chains another assertion on the same subject (alias of should)

Readable.

**Q20.** Why can a should with a callback retry?

**Answer:** Cypress re-runs the callback until it passes or times out

Callbacks must be idempotent.

### [Auto-Waiting &amp; Retry-Ability](level-1/06-auto-waiting-retry-ability.md)

**Q21.** What does automatic retry-ability do for a 1.5s-delayed element?

**Answer:** Cypress keeps retrying the query/assertion until it appears (default timeout 4s)

No manual waits needed.

**Q22.** Why run the test 10 times in the lesson?

**Answer:** To measure real reliability rather than trust one lucky pass

Flakiness is statistical.

**Q23.** What's wrong with `cy.wait(2000)` for an element?

**Answer:** Arbitrary sleeps are slow and brittle; assert on the condition instead

Use assertions.

**Q24.** What's the default command timeout?

**Answer:** 4 seconds

Configurable via defaultCommandTimeout.

### [Forms &amp; Dropdowns](level-1/07-forms-dropdowns.md)

**Q25.** Which command picks a dropdown option?

**Answer:** cy.get('select').select('value or text')

Works with value or visible text.

**Q26.** How do you choose a radio option?

**Answer:** cy.get('input[type=radio]').check('value')

check handles radios.

**Q27.** What should a login-form test assert after submit?

**Answer:** The resulting UI or URL change, such as a welcome message

Verify outcomes.

**Q28.** Why prefer data-* attributes for selectors?

**Answer:** They're stable and independent of styling and structure

data-cy / data-testid.

### [Navigation: visit, reload, go](level-1/08-navigation.md)

**Q29.** What does cy.visit('/page') do?

**Answer:** Loads the page and waits for the load event

Fails on non-2xx by default.

**Q30.** Does text typed into an input survive cy.go('back') then forward?

**Answer:** It depends on browser form restoration, so test it rather than assume

The lesson checks real state.

**Q31.** What does cy.reload() do?

**Answer:** Reloads the current page

State resets except persisted storage.

**Q32.** What does cy.location('pathname') yield?

**Answer:** The current URL path for assertion

Use should to assert.

### [The Cypress App &amp; Headless Mode](level-1/09-cypress-app-headless-mode.md)

**Q33.** What is `cypress run`?

**Answer:** Headless execution, typically for CI

cypress open is the interactive runner.

**Q34.** Which browser does Cypress use by default in run mode?

**Answer:** Its bundled Electron

Use --browser chrome for others.

**Q35.** What does `cypress info` show?

**Answer:** Detected browsers and environment details

Helps debug setup.

**Q36.** Why run in headless mode on CI?

**Answer:** No display is needed and it's scriptable and faster

Videos and screenshots can still be saved.

### [Project — a Real Login Flow Test](level-1/10-project-login-flow-test.md)

**Q37.** What makes the login-flow project good practice?

**Answer:** Zero manual waits and assertions that retry

Combines the level's techniques.

**Q38.** How should invalid login be tested?

**Answer:** Assert the specific error message appears

Verify behavior.

**Q39.** Where should test data live?

**Answer:** In fixtures or config, not hard-coded across tests

Reusable.

**Q40.** What makes each test independent?

**Answer:** It sets up its own state and doesn't rely on previous tests

Independent tests.

## Level 2 · Intermediate Patterns

### [Custom Commands](level-2/01-custom-commands.md)

**Q41.** What does Cypress.Commands.add('login', ...) create?

**Answer:** A reusable custom command callable as cy.login()

Centralizes repeated flows.

**Q42.** What did renaming the login button's id show?

**Answer:** With a custom command only one place needs fixing instead of every test

The cost of duplication.

**Q43.** Where are custom commands usually defined?

**Answer:** cypress/support/commands.js

Loaded before specs.

**Q44.** When is a custom command a bad idea?

**Answer:** When it hides assertions or becomes a grab-bag; keep them focused

Prefer clarity.

### [Fixtures &amp; Test Data](level-2/02-fixtures-test-data.md)

**Q45.** What does cy.fixture('users') do?

**Answer:** Loads data from cypress/fixtures/users.json

Great for data-driven tests.

**Q46.** How do you drive the same flow with several data rows?

**Answer:** Loop over fixture rows and generate tests/iterations

Data-driven testing.

**Q47.** What's a benefit of JSON fixtures?

**Answer:** Separates data from test logic

Easier maintenance.

**Q48.** Can fixtures also stub network responses?

**Answer:** Yes, via cy.intercept(..., { fixture: 'file.json' })

Combines two features.

### [Network Stubbing With cy.intercept](level-2/03-network-stubbing-intercept.md)

**Q49.** What does cy.intercept() do?

**Answer:** Spies on or stubs network requests

Matches by method and URL.

**Q50.** What's the difference between spying and stubbing?

**Answer:** Spying observes real requests; stubbing returns a fake response

Choose by need.

**Q51.** Why stub a server error response?

**Answer:** To test the UI's error handling deterministically

Hard to trigger otherwise.

**Q52.** How do you wait for an intercepted request to complete?

**Answer:** Alias it and cy.wait('@alias')

Wait on the route, not time.

### [Calling APIs Directly With cy.request](level-2/04-cy-request.md)

**Q53.** What does cy.request() do?

**Answer:** Makes an HTTP request directly from Cypress, bypassing the browser UI

Useful for setup.

**Q54.** What happens on a 401 from cy.request by default?

**Answer:** The command fails unless failOnStatusCode is false

Unlike fetch.

**Q55.** Why log in via cy.request?

**Answer:** Faster and more reliable than UI login for setup

Test the login UI separately.

**Q56.** Why seed data via API?

**Answer:** Quick known state without clicking through the UI

Isolation.

### [Iframes: a Real Cypress Limitation](level-2/05-iframes-real-limitation.md)

**Q57.** Why can't cy.get see inside an iframe by default?

**Answer:** The iframe has a separate document; cy.get queries the top document

A real limitation.

**Q58.** What's the usual workaround?

**Answer:** Access iframe.contentDocument body and wrap it, or use a plugin

Custom command.

**Q59.** What does the course say about the iframe limitation?

**Answer:** It's reported honestly as a real limitation

Accuracy.

**Q60.** Cross-origin iframes add what difficulty?

**Answer:** Same-origin policy blocks access to their DOM

Browser-enforced.

### [File Upload With selectFile](level-2/06-file-upload.md)

**Q61.** How do you upload a file in Cypress?

**Answer:** cy.get('input[type=file]').selectFile('path')

selectFile is built in.

**Q62.** Where do upload fixtures live?

**Answer:** cypress/fixtures

Reference by path.

**Q63.** What should you assert after upload?

**Answer:** The UI shows the file name or success message

Verify outcome.

**Q64.** What does selectFile's drag-drop action do?

**Answer:** Simulates dropping the file onto a drop zone

{ action: 'drag-drop' }.

### [Environment Variables &amp; Configuration](level-2/07-env-variables-config.md)

**Q65.** How do you read an environment variable in a test?

**Answer:** Cypress.env('NAME')

Set via config, env file or CLI.

**Q66.** How do you pass env vars on the command line?

**Answer:** --env KEY=value

CI friendly.

**Q67.** Why keep secrets out of cypress.config.js committed files?

**Answer:** They'd leak in source control; use CI secrets

Never commit secrets.

**Q68.** What's baseUrl per environment used for?

**Answer:** Pointing the same tests at staging or local

Override in CLI.

### [Hooks: before, beforeEach, after, afterEach](level-2/08-hooks.md)

**Q69.** Which hook runs before each test?

**Answer:** beforeEach

before runs once per describe.

**Q70.** What belongs in beforeEach?

**Answer:** Setup that gives each test a fresh known state

Isolation.

**Q71.** Why avoid relying on test order?

**Answer:** Tests should be independent and runnable alone

Order dependence causes flakes.

**Q72.** What does `afterEach` risk?

**Answer:** Cleanup may not run if the browser crashes; prefer cleaning in setup

Cypress guidance.

### [Debugging Failing Tests](level-2/09-debugging-failing-tests.md)

**Q73.** What does cy.pause() or .debug() help with?

**Answer:** Stopping the run to inspect state

Debugging.

**Q74.** What does the Command Log show?

**Answer:** Each command and its snapshot you can time-travel through

Hover to see DOM.

**Q75.** How do you see a failing test's page state in CI?

**Answer:** Screenshots and videos saved on failure

Configured by default for run.

**Q76.** What's a good first step for a failing selector?

**Answer:** Check the DOM snapshot at that step to see what exists

Time-travel debugging.

### [Project — a Real Network-Stubbed Suite](level-2/10-project-network-stubbed-suite.md)

**Q77.** What does the network-stubbed suite achieve?

**Answer:** Fast deterministic tests independent of the backend

Stubs control responses.

**Q78.** What should still be tested without stubs?

**Answer:** A few real end-to-end flows to catch integration issues

Balance.

**Q79.** What does asserting on the request body check?

**Answer:** That the UI sends the right payload

Use the intercept alias.

**Q80.** What risk comes from heavily stubbed tests?

**Answer:** Stubs can drift from the real API, hiding breakage

Contract tests help.

## Level 3 · Architecture & Advanced Techniques

### [The Command Queue: Why Cypress Code Isn't Actually Async](level-3/01-the-command-queue.md)

**Q81.** How does Cypress execute commands?

**Answer:** Queues them and runs asynchronously in order; commands don't return values directly

The command queue.

**Q82.** Why does `const el = cy.get('x')` not give an element?

**Answer:** cy commands return chainables, not elements; use .then() or aliases

Async queue.

**Q83.** When does the queued command actually run?

**Answer:** After the test function finishes enqueueing, in order

Enqueue first.

**Q84.** Why does mixing plain JS async/await with cy commands cause trouble?

**Answer:** Cypress commands aren't promises and don't work with await

Use .then.

### [Aliases: as() and @alias](level-3/02-aliases.md)

**Q85.** What does .as('todos') do?

**Answer:** Creates an alias for later reference

Use @todos.

**Q86.** How do you reference an element alias?

**Answer:** cy.get('@todos')

For intercepts too.

**Q87.** Why use aliases for intercepts?

**Answer:** To wait on and assert about specific requests

cy.wait('@alias').

**Q88.** What does `this.alias` require?

**Answer:** Using function() rather than an arrow function and a context

this binding.

### [Retry-Ability Pitfalls](level-3/03-retry-ability-pitfalls.md)

**Q89.** What does retry-ability retry?

**Answer:** Queries and assertions, not actions like click

Only the last query + assertions.

**Q90.** Why is `cy.get('x').click().should('...')` risky?

**Answer:** Click isn't retried, so a re-render between can break the chain

Re-query after actions.

**Q91.** What causes detached DOM errors?

**Answer:** Element re-rendered between query and action

Re-query.

**Q92.** How do you make assertions robust?

**Answer:** Assert on the stable end state, not intermediate steps

Simplify.

### [Cookies, Local Storage &amp; cy.session](level-3/04-cookies-session.md)

**Q93.** What does cy.session() do?

**Answer:** Caches and restores login state (cookies/localStorage) across tests

Speeds up auth.

**Q94.** What does cy.clearCookies() do?

**Answer:** Removes cookies for isolation

Cypress clears automatically between tests.

**Q95.** Why log in once per spec via session?

**Answer:** Saves time versus UI login before every test

Validate with a function.

**Q96.** How do you set a cookie?

**Answer:** cy.setCookie(name, value)

Quick auth.

### [cy.intercept Deep Dive: Stub vs Spy](level-3/05-intercept-deep-dive.md)

**Q97.** What can cy.intercept's routeHandler do?

**Answer:** Modify request or response, delay or fail it

Full control.

**Q98.** How do you simulate a slow network?

**Answer:** Use delay in a stub or throttle the response

Deterministic.

**Q99.** How do you assert a request body?

**Answer:** Intercept with alias then cy.wait('@alias').its('request.body')

Inspect the interception.

**Q100.** What happens with overlapping intercept routes?

**Answer:** Handlers run in reverse definition order until one responds

Most recent first.

### [Screenshots &amp; Videos on Failure](level-3/06-screenshots-videos.md)

**Q101.** When are screenshots taken automatically?

**Answer:** On failure in cypress run

Configurable.

**Q102.** Where are videos useful?

**Answer:** Diagnosing CI failures you can't watch live

Run mode records them.

**Q103.** What does cy.screenshot() do?

**Answer:** Captures the page or an element on demand

Visual evidence.

**Q104.** Why keep artifacts as CI outputs?

**Answer:** Developers can inspect failures after the run

Upload them.

### [Flaky Test Diagnosis](level-3/07-flaky-test-diagnosis.md)

**Q105.** What's a first step in diagnosing a flaky test?

**Answer:** Run it many times and look at the failure pattern

Evidence first.

**Q106.** Common causes of flakiness include...

**Answer:** Race conditions, shared state and unstable selectors

Fix root cause.

**Q107.** What does Cypress's test retries feature do?

**Answer:** Reruns failing tests up to N times

Use sparingly; track flaky tests.

**Q108.** Why not just add cy.wait(ms)?

**Answer:** It slows tests and masks the real race

Wait on conditions.

### [CI Integration](level-3/08-ci-integration.md)

**Q109.** What does `cypress run --browser chrome` do?

**Answer:** Runs the suite headlessly in Chrome

CI mode.

**Q110.** Why cache node_modules and the Cypress binary in CI?

**Answer:** To speed up installs

Binary cache folder.

**Q111.** What should CI start before tests?

**Answer:** The app under test, and wait until it's ready

Use start-server-and-test.

**Q112.** What should fail the CI job?

**Answer:** Any failing spec (non-zero exit code)

Gate merges.

### [Parallelization &amp; Cypress Cloud](level-3/09-parallelization-cloud.md)

**Q113.** What does parallelization do?

**Answer:** Splits specs across machines to shorten total time

Needs recording/grouping.

**Q114.** What balances parallel runs?

**Answer:** Spec duration history when using the cloud service

Load balancing.

**Q115.** What's a requirement for parallelizing specs?

**Answer:** Specs must be independent

No cross-spec dependencies.

**Q116.** What's an alternative to the cloud for splitting?

**Answer:** Manually shard spec lists across CI jobs

Plain CI matrices.

### [Project — a Real POM Framework With Custom Commands](level-3/10-project-pom-framework.md)

**Q117.** What is the Page Object Model?

**Answer:** Encapsulating page selectors and actions in classes/modules

Reduces duplication.

**Q118.** What does POM improve?

**Answer:** Maintenance: UI changes touch one place

Like custom commands.

**Q119.** What should page objects avoid?

**Answer:** Embedding assertions that hide test intent

Keep assertions in tests.

**Q120.** Cypress's own guidance on page objects?

**Answer:** They're optional; custom commands and app actions may be simpler

Choose pragmatically.

## Level 4 · Component Testing, Scaling & Comparisons

### [Why Cypress's Architecture Is Different](level-4/01-why-architecture-is-different.md)

**Q121.** Why is Cypress's architecture different from Selenium's?

**Answer:** Tests run in the browser rather than controlling it over a remote protocol

Direct access.

**Q122.** What advantage follows from running in the browser?

**Answer:** Fast, reliable access to DOM, network and app state

Automatic waiting.

**Q123.** What trade-off follows?

**Answer:** Language support is JS/TS and some browser limitations apply

Honest trade-offs.

**Q124.** What is the architecture's effect on flakiness?

**Answer:** Fewer timing issues because Cypress knows the app's state and retries

Auto-wait.

### [Component Testing](level-4/02-component-testing.md)

**Q125.** What is Cypress component testing?

**Answer:** Mounting individual components in a real browser and testing them in isolation

cy.mount.

**Q126.** When choose component tests over e2e?

**Answer:** For fast, isolated checks of UI logic

Complementary.

**Q127.** What does cy.mount() require?

**Answer:** A framework adapter (React, Vue, Angular, etc.)

Bundler config.

**Q128.** What do component tests not cover?

**Answer:** Full-stack integration between services

Use e2e for that.

### [Multiple Origins &amp; cy.origin](level-4/03-multiple-origins.md)

**Q129.** What does cy.origin() enable?

**Answer:** Running commands against a second origin in one test

Needed for cross-domain flows.

**Q130.** When do you need it?

**Answer:** When the test navigates to a different superdomain (e.g. SSO)

Browser security.

**Q131.** What's a constraint of cy.origin callbacks?

**Answer:** Variables must be passed explicitly, they don't capture outer scope

Serializable args.

**Q132.** Why was multi-origin once impossible?

**Answer:** Cypress enforced a single superdomain per test

Historic limitation.

### [The Same-Origin Constraint, Investigated](level-4/04-same-origin-constraint.md)

**Q133.** What is the same-origin constraint?

**Answer:** Within a test Cypress couldn't visit different superdomains without cy.origin

Browser security.

**Q134.** Why does it exist?

**Answer:** Cypress runs inside the browser and must obey its security model

Same-origin policy.

**Q135.** How do you avoid it for login?

**Answer:** Log in via cy.request or session instead of UI at the third party

Workarounds.

**Q136.** What does the lesson say about honesty?

**Answer:** These constraints are real and should be understood

Accuracy.

### [Plugins &amp; cy.task](level-4/05-plugins-cy-task.md)

**Q137.** What does cy.task() do?

**Answer:** Runs Node.js code in the Cypress process (e.g. DB seeding)

Defined in setupNodeEvents.

**Q138.** Why can't tests access the filesystem directly?

**Answer:** They run in the browser; use cy.readFile or cy.task

Node vs browser.

**Q139.** Where are plugins/tasks configured?

**Answer:** setupNodeEvents in cypress.config.js

Runs in Node.

**Q140.** What must a cy.task handler return?

**Answer:** A value or promise (not undefined)

null is allowed.

### [Performance: Measuring Real Command Timing](level-4/06-performance-measuring.md)

**Q141.** How should you measure test run time fairly?

**Answer:** Repeat runs and compare distributions, noting caching and machine differences

Honest measurement.

**Q142.** What typically dominates Cypress test time?

**Answer:** Page loads and UI interactions, not command overhead

Reduce UI steps via API setup.

**Q143.** How can you speed suites up?

**Answer:** API-based setup, session caching and parallelization

Measure first.

**Q144.** Why not optimize without measuring?

**Answer:** You may speed up the wrong thing

Evidence.

### [Cypress vs Selenium: a Real, Honest Comparison](level-4/07-cypress-vs-selenium.md)

**Q145.** A big difference of Cypress vs Selenium?

**Answer:** Cypress runs in-browser with auto-waiting; Selenium drives browsers via WebDriver and supports more languages and browsers

Choose by needs.

**Q146.** When might Selenium be the better choice?

**Answer:** Broad browser/language support or testing non-web contexts

Trade-offs.

**Q147.** What's easier in Cypress?

**Answer:** Debugging with time-travel snapshots and auto-retry

Developer experience.

**Q148.** What is a Selenium Grid analogue?

**Answer:** Cypress Cloud parallelization or CI sharding

Scaling.

### [Cypress vs Playwright: a Real, Honest Comparison](level-4/08-cypress-vs-playwright.md)

**Q149.** A difference between Cypress and Playwright?

**Answer:** Playwright controls browsers out of process with multi-tab/browser support; Cypress runs inside the browser

Architectural contrast.

**Q150.** Which supports multiple browser engines natively?

**Answer:** Playwright (Chromium, Firefox, WebKit)

Cypress WebKit is experimental.

**Q151.** Which has built-in parallelism without a paid service?

**Answer:** Playwright runs workers in parallel by default

Cypress parallelization via Cloud or manual sharding.

**Q152.** Why compare tools honestly?

**Answer:** Choose by requirements, not hype

Context matters.

### [Common Cypress Anti-Patterns](level-4/09-cypress-antipatterns.md)

**Q153.** Which is a Cypress antipattern?

**Answer:** cy.wait(5000) instead of asserting on a condition

Brittle and slow.

**Q154.** Why avoid tests depending on each other?

**Answer:** They fail unpredictably when run alone or reordered

Independent tests.

**Q155.** Why avoid conditional testing (if element exists)?

**Answer:** The DOM can change after the check so it's flaky; make state deterministic

Control the app state.

**Q156.** Why not use brittle selectors like nth-child chains?

**Answer:** They break on layout changes; use data-* attributes

Stable selectors.

### [Capstone — a Real, Complete Test Framework](level-4/10-capstone-complete-test-framework.md)

**Q157.** What does the capstone combine?

**Answer:** Custom commands, fixtures, intercepts, sessions, CI and reports

A complete framework.

**Q158.** What makes a framework maintainable?

**Answer:** Clear structure, stable selectors, reusable commands and independent tests

Simplicity.

**Q159.** How do you prove the framework is reliable?

**Answer:** Run repeatedly in CI and track flake rates

Measure.

**Q160.** What should be documented?

**Answer:** How to run, add tests and interpret failures

Onboarding.

## Mixed questions across all levels

**Q161.** A Cypress test types into an input after the page loads and the field renders 1.5 s late. What's the right approach?

**Answer:** Rely on automatic retry: cy.get(...).type(...) with no manual wait

Queries retry until the element exists or the timeout hits.

**Q162.** Why does `const btn = cy.get('button'); btn.click()` not behave like normal JavaScript?

**Answer:** cy commands enqueue and return chainables, not elements; the DOM query hasn't run yet

Use chaining, .then() or aliases.

**Q163.** Which part of `cy.get('.row').should('have.length', 3)` is retried?

**Answer:** The query and the assertion together until they pass or time out

Retry-ability covers queries and assertions.

**Q164.** A test does `cy.get('.btn').click().should('be.visible')` and fails with a detached element after a re-render. What's the fix?

**Answer:** Re-query after the action and assert on a stable end state

Actions aren't retried.

**Q165.** Which is the best selector for a Save button?

**Answer:** [data-cy=save]

Stable, intent-revealing hooks survive refactors.

**Q166.** A test depends on a todo created by the previous test. What's wrong?

**Answer:** Tests must be independent; create state in beforeEach or via API

Order dependence is a flakiness source.

**Q167.** Login via the UI before each of 50 tests is slow. What do you do?

**Answer:** Use cy.session() or cy.request to authenticate quickly, and test the login UI once

Separate setup from what you test.

**Q168.** You want to test the UI's error state when the API returns 500. What do you use?

**Answer:** cy.intercept to stub a 500 response

Deterministic failure injection.

**Q169.** How do you wait for an API call before asserting on the UI it updates?

**Answer:** Alias the intercept and cy.wait('@alias')

Wait on the request.

**Q170.** Why does `cy.request()` returning 401 fail the test by default?

**Answer:** failOnStatusCode is true; set it false to assert on error responses

Different from fetch.

**Q171.** A test needs to read a file and seed a database. Which Cypress feature runs Node code?

**Answer:** cy.task()

Tasks run in Node via setupNodeEvents.

**Q172.** The test visits your app, clicks 'Login with SSO' and lands on another domain. What do you need?

**Answer:** cy.origin() for commands against the other origin

Same-origin constraint.

**Q173.** Content inside an iframe isn't found by cy.get. Why?

**Answer:** The iframe has a separate document; get the iframe's body and wrap it

A real limitation, solved with a helper command.

**Q174.** A test fails only in CI. What's the most useful evidence?

**Answer:** Screenshots, video and the command log from the failed run

Collect artifacts.

**Q175.** Your suite has 30 specs taking 40 minutes. How do you speed it up safely?

**Answer:** Shard or parallelize independent specs and move setup to API calls

Independence is required.

**Q176.** A flaky test passes after Cypress test retries. What should you still do?

**Answer:** Track it as flaky and fix the root cause

Retries mask problems.

**Q177.** When would you choose Playwright or Selenium over Cypress?

**Answer:** Multi-tab or multi-browser needs (e.g. WebKit) or non-JS languages

Cypress runs in-browser with trade-offs.

**Q178.** Which is an antipattern?

**Answer:** Using `cy.wait(5000)` to wait for an element

Assert on the condition.

**Q179.** Component tests vs e2e: which fits checking a form component's validation messages quickly?

**Answer:** A component test mounting the component in isolation

Fast and isolated; keep a few e2e flows too.

## Practical coding questions

### Model retry-ability

Implement retry(check, {timeout, interval}) using a fake clock: call check(elapsed) repeatedly, advancing elapsed by `interval`, until check returns {ok:true} (return {passed:true, elapsed}) or the next step would exceed `timeout` (return {passed:false, elapsed, error}). Use it to compare an element that appears after 1.5 s and one after 6 s.

Starter code:

```
// A fake clock makes the simulation deterministic.
function retry(check, { timeout = 4000, interval = 50 } = {}) {
  // call check(elapsedMs) repeatedly, advancing a fake clock by `interval`,
  // until it returns {ok:true} (return {passed:true, elapsed}) or the next step
  // would exceed `timeout` (return {passed:false, elapsed, error}).
}

// The element appears after 1500 ms (like the 1.5 s delayed element).
const appears = (ms) => (t) =>
  t >= ms ? { ok: true } : { ok: false, error: "Expected to find element, but never found it" };

console.log(retry(appears(1500)));
console.log(retry(appears(6000)));
console.log(retry(appears(1500), { timeout: 1000 }));
```

Expected output:

```text
{ passed: true, elapsed: 1500 }
{
  passed: false,
  elapsed: 4000,
  error: 'Expected to find element, but never found it'
}
{
  passed: false,
  elapsed: 1000,
  error: 'Expected to find element, but never found it'
}
```

??? success "Show a solution"

    ```
    // A fake clock makes the simulation deterministic.
    function retry(check, { timeout = 4000, interval = 50 } = {}) {
      let elapsed = 0;
      while (true) {
        const result = check(elapsed);
        if (result.ok) return { passed: true, elapsed };
        if (elapsed + interval > timeout) return { passed: false, elapsed, error: result.error };
        elapsed += interval;
      }
    }

    // The element appears after 1500 ms (like the 1.5 s delayed element).
    const appears = (ms) => (t) =>
      t >= ms ? { ok: true } : { ok: false, error: "Expected to find element, but never found it" };

    console.log(retry(appears(1500)));
    console.log(retry(appears(6000)));
    console.log(retry(appears(1500), { timeout: 1000 }));
    ```

    Cypress doesn't sleep for a fixed time: it re-runs the query and assertions until they pass or the 4 s default timeout expires, so the delayed element passes at 1500 ms and the 6 s one fails with the last error. This is a model of that behavior, not Cypress code. Verified by running the solution in Node.

### Model the command queue

Complete the tiny command queue: each command should enqueue a function and return `cy` so calls chain, and nothing runs until runQueue(). Observe that the test body's own log line runs before any enqueued command, just like in Cypress.

Starter code:

```
// Complete the tiny command queue: every command should ENQUEUE a function and return `cy`
// so calls chain; nothing runs until runQueue() is called (like Cypress after the test body).
const queue = [];
const log = [];
const cy = {
  visit(url) { /* enqueue: log.push("visit " + url) */ return cy; },
  get(sel) { /* enqueue: log.push("get " + sel) */ return cy; },
  click() { /* enqueue: log.push("click") */ return cy; },
  then(fn) { /* enqueue: call fn */ return cy; },
};

function runQueue() { while (queue.length) queue.shift()(); }

cy.visit("/login").get("#user").click();
log.push("test body finished");
cy.then(() => log.push("then callback"));

console.log("queued:", queue.length);
runQueue();
console.log(log.join(" | "));
```

Expected output:

```text
queued: 4
test body finished | visit /login | get #user | click | then callback
```

??? success "Show a solution"

    ```
    // A tiny model of Cypress's command queue: commands are enqueued while the test
    // function runs, then executed later, in order.
    const queue = [];
    const log = [];
    const cy = {
      visit(url) { queue.push(() => log.push("visit " + url)); return cy; },
      get(sel) { queue.push(() => log.push("get " + sel)); return cy; },
      click() { queue.push(() => log.push("click")); return cy; },
      then(fn) { queue.push(() => fn()); return cy; },
    };

    function runQueue() { while (queue.length) queue.shift()(); }

    // "Test body": runs synchronously and only ENQUEUES.
    cy.visit("/login").get("#user").click();
    log.push("test body finished");        // runs BEFORE any command above
    cy.then(() => log.push("then callback"));

    console.log("queued:", queue.length);
    runQueue();
    console.log(log.join(" | "));
    ```

    Cypress commands are enqueued while your test function runs and executed afterwards in order, which is why plain JS around them runs first and why you can't assign a command's result to a variable. This is a teaching model of the idea, not Cypress internals. Verified by running the solution in Node.

---

More free practice: the [Cypress Mastery Path](index.md) curriculum, the [Interview Prep Pack](https://learnmasterypath.com/interview-pack/) and the [free resume ATS check](https://learnmasterypath.com/resume/).
