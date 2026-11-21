---
title: "Final Quiz"
description: "Test what you have learned across all four levels of Cypress Mastery Path with a 19-question final quiz."
---

# Final Quiz

19 questions drawn from all four levels of this course. You need **70% (14 out of 19)** to pass —
you can retake it as many times as you like. Your best result is saved in this browser.

<div id="mp-quiz-root"></div>

<script type="application/json" id="mp-quiz-data">
[
  {
    "question": "A Cypress test types into an input after the page loads and the field renders 1.5 s late. What's the right approach?",
    "options": [
      "A setTimeout",
      "A try/catch loop",
      "Rely on automatic retry: cy.get(...).type(...) with no manual wait",
      "cy.wait(3000)"
    ],
    "correctIndex": 2,
    "explanation": "Queries retry until the element exists or the timeout hits."
  },
  {
    "question": "Why does `const btn = cy.get('button'); btn.click()` not behave like normal JavaScript?",
    "options": [
      "It's a bug",
      "get returns null",
      "Cypress is slow",
      "cy commands enqueue and return chainables, not elements; the DOM query hasn't run yet"
    ],
    "correctIndex": 3,
    "explanation": "Use chaining, .then() or aliases."
  },
  {
    "question": "Which part of `cy.get('.row').should('have.length', 3)` is retried?",
    "options": [
      "Only click",
      "The query and the assertion together until they pass or time out",
      "Nothing",
      "Only visit"
    ],
    "correctIndex": 1,
    "explanation": "Retry-ability covers queries and assertions."
  },
  {
    "question": "A test does `cy.get('.btn').click().should('be.visible')` and fails with a detached element after a re-render. What's the fix?",
    "options": [
      "Use force:true",
      "Re-query after the action and assert on a stable end state",
      "Add cy.wait(1000)",
      "Retry the click"
    ],
    "correctIndex": 1,
    "explanation": "Actions aren't retried."
  },
  {
    "question": "Which is the best selector for a Save button?",
    "options": [
      "div > div > button",
      ".btn:nth-child(3)",
      "#root button",
      "[data-cy=save]"
    ],
    "correctIndex": 3,
    "explanation": "Stable, intent-revealing hooks survive refactors."
  },
  {
    "question": "A test depends on a todo created by the previous test. What's wrong?",
    "options": [
      "Nothing",
      "Law",
      "Speed",
      "Tests must be independent; create state in beforeEach or via API"
    ],
    "correctIndex": 3,
    "explanation": "Order dependence is a flakiness source."
  },
  {
    "question": "Login via the UI before each of 50 tests is slow. What do you do?",
    "options": [
      "Add sleeps",
      "Skip login",
      "Cache cookies by hand",
      "Use cy.session() or cy.request to authenticate quickly, and test the login UI once"
    ],
    "correctIndex": 3,
    "explanation": "Separate setup from what you test."
  },
  {
    "question": "You want to test the UI's error state when the API returns 500. What do you use?",
    "options": [
      "Break the server",
      "cy.wait",
      "cy.intercept to stub a 500 response",
      "A fixture only"
    ],
    "correctIndex": 2,
    "explanation": "Deterministic failure injection."
  },
  {
    "question": "How do you wait for an API call before asserting on the UI it updates?",
    "options": [
      "setTimeout",
      "Alias the intercept and cy.wait('@alias')",
      "cy.wait(2000)",
      "Reload"
    ],
    "correctIndex": 1,
    "explanation": "Wait on the request."
  },
  {
    "question": "Why does `cy.request()` returning 401 fail the test by default?",
    "options": [
      "It's a bug",
      "Law",
      "Cypress hates APIs",
      "failOnStatusCode is true; set it false to assert on error responses"
    ],
    "correctIndex": 3,
    "explanation": "Different from fetch."
  },
  {
    "question": "A test needs to read a file and seed a database. Which Cypress feature runs Node code?",
    "options": [
      "cy.task()",
      "cy.request",
      "cy.fixture",
      "cy.exec only"
    ],
    "correctIndex": 0,
    "explanation": "Tasks run in Node via setupNodeEvents."
  },
  {
    "question": "The test visits your app, clicks 'Login with SSO' and lands on another domain. What do you need?",
    "options": [
      "A longer timeout",
      "cy.origin() for commands against the other origin",
      "cy.wait",
      "cy.intercept"
    ],
    "correctIndex": 1,
    "explanation": "Same-origin constraint."
  },
  {
    "question": "Content inside an iframe isn't found by cy.get. Why?",
    "options": [
      "Cypress is broken",
      "Wrong selector",
      "The iframe has a separate document; get the iframe's body and wrap it",
      "Timeout"
    ],
    "correctIndex": 2,
    "explanation": "A real limitation, solved with a helper command."
  },
  {
    "question": "A test fails only in CI. What's the most useful evidence?",
    "options": [
      "Re-run until green",
      "Delete it",
      "Screenshots, video and the command log from the failed run",
      "Add retries only"
    ],
    "correctIndex": 2,
    "explanation": "Collect artifacts."
  },
  {
    "question": "Your suite has 30 specs taking 40 minutes. How do you speed it up safely?",
    "options": [
      "Run fewer asserts",
      "Shard or parallelize independent specs and move setup to API calls",
      "Skip failing",
      "Add cy.wait"
    ],
    "correctIndex": 1,
    "explanation": "Independence is required."
  },
  {
    "question": "A flaky test passes after Cypress test retries. What should you still do?",
    "options": [
      "Ignore it",
      "Delete assertions",
      "Track it as flaky and fix the root cause",
      "Raise retries to 10"
    ],
    "correctIndex": 2,
    "explanation": "Retries mask problems."
  },
  {
    "question": "When would you choose Playwright or Selenium over Cypress?",
    "options": [
      "For speed only",
      "Never",
      "For assertions",
      "Multi-tab or multi-browser needs (e.g. WebKit) or non-JS languages"
    ],
    "correctIndex": 3,
    "explanation": "Cypress runs in-browser with trade-offs."
  },
  {
    "question": "Which is an antipattern?",
    "options": [
      "Using aliases",
      "Using fixtures",
      "Using data-cy",
      "Using `cy.wait(5000)` to wait for an element"
    ],
    "correctIndex": 3,
    "explanation": "Assert on the condition."
  },
  {
    "question": "Component tests vs e2e: which fits checking a form component's validation messages quickly?",
    "options": [
      "A component test mounting the component in isolation",
      "Manual",
      "Full e2e",
      "Load test"
    ],
    "correctIndex": 0,
    "explanation": "Fast and isolated; keep a few e2e flows too."
  }
]
</script>

## Coding challenges

Try these on your own computer, then compare your output. Nothing you write is sent anywhere.

### Model retry-ability

Implement retry(check, {timeout, interval}) using a fake clock: call check(elapsed) repeatedly, advancing elapsed by `interval`, until check returns {ok:true} (return {passed:true, elapsed}) or the next step would exceed `timeout` (return {passed:false, elapsed, error}). Use it to compare an element that appears after 1.5 s and one after 6 s.

**Starter code**

```javascript
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

**Expected output**

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

<details><summary>Show solution and explanation</summary>

<pre><code>// A fake clock makes the simulation deterministic.
function retry(check, { timeout = 4000, interval = 50 } = {}) {
  let elapsed = 0;
  while (true) {
    const result = check(elapsed);
    if (result.ok) return { passed: true, elapsed };
    if (elapsed + interval &gt; timeout) return { passed: false, elapsed, error: result.error };
    elapsed += interval;
  }
}

// The element appears after 1500 ms (like the 1.5 s delayed element).
const appears = (ms) =&gt; (t) =&gt;
  t &gt;= ms ? { ok: true } : { ok: false, error: &quot;Expected to find element, but never found it&quot; };

console.log(retry(appears(1500)));
console.log(retry(appears(6000)));
console.log(retry(appears(1500), { timeout: 1000 }));
</code></pre>

<p>Cypress doesn&#x27;t sleep for a fixed time: it re-runs the query and assertions until they pass or the 4 s default timeout expires, so the delayed element passes at 1500 ms and the 6 s one fails with the last error. This is a model of that behavior, not Cypress code. Verified by running the solution in Node.</p>
</details>

### Model the command queue

Complete the tiny command queue: each command should enqueue a function and return `cy` so calls chain, and nothing runs until runQueue(). Observe that the test body's own log line runs before any enqueued command, just like in Cypress.

**Starter code**

```javascript
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

**Expected output**

```text
queued: 4
test body finished | visit /login | get #user | click | then callback
```

<details><summary>Show solution and explanation</summary>

<pre><code>// A tiny model of Cypress&#x27;s command queue: commands are enqueued while the test
// function runs, then executed later, in order.
const queue = [];
const log = [];
const cy = {
  visit(url) { queue.push(() =&gt; log.push(&quot;visit &quot; + url)); return cy; },
  get(sel) { queue.push(() =&gt; log.push(&quot;get &quot; + sel)); return cy; },
  click() { queue.push(() =&gt; log.push(&quot;click&quot;)); return cy; },
  then(fn) { queue.push(() =&gt; fn()); return cy; },
};

function runQueue() { while (queue.length) queue.shift()(); }

// &quot;Test body&quot;: runs synchronously and only ENQUEUES.
cy.visit(&quot;/login&quot;).get(&quot;#user&quot;).click();
log.push(&quot;test body finished&quot;);        // runs BEFORE any command above
cy.then(() =&gt; log.push(&quot;then callback&quot;));

console.log(&quot;queued:&quot;, queue.length);
runQueue();
console.log(log.join(&quot; | &quot;));
</code></pre>

<p>Cypress commands are enqueued while your test function runs and executed afterwards in order, which is why plain JS around them runs first and why you can&#x27;t assign a command&#x27;s result to a variable. This is a teaching model of the idea, not Cypress internals. Verified by running the solution in Node.</p>
</details>
