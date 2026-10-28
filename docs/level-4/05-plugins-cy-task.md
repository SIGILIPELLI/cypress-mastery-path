---
description: "cy.task('logToRealTerminal', ...) genuinely printed to the actual shell running cypress run -- something cy.log() never does."
---

# Plugins & cy.task

## What was run

A real Node-side task, registered via `setupNodeEvents` in `cypress.config.js`, that writes to the actual
terminal and performs a genuine filesystem read/write — both things Cypress's in-browser test code
([Level 4, Lesson 1](01-why-architecture-is-different.md)) cannot do directly, confirmed by actually
running them rather than assuming the capability boundary.

```javascript
// cypress.config.js
setupNodeEvents(on, config) {
  on("task", {
    logToRealTerminal(message) {
      console.log("REAL TASK LOG:", message);
      return null;
    },
    writeRealFile({ path, contents }) {
      fs.writeFileSync(path, contents);
      return null;
    },
    readRealFile(path) {
      return fs.readFileSync(path, "utf8");
    },
  });
  return config;
},
```

```javascript
it("logs a real message to the actual terminal, not just the browser console", () => {
  cy.task("logToRealTerminal", "hello from a real cy.task call");
});

it("writes and reads a real file on disk via Node, not the browser", () => {
  const path = "/tmp/real_cy_task_test.txt";
  cy.task("writeRealFile", { path, contents: "real content written via cy.task" });
  cy.task("readRealFile", path).should("eq", "real content written via cy.task");
});
```
```
Running:  plugins-task.cy.js
REAL TASK LOG: hello from a real cy.task call
2 passing (44ms)
```

That `REAL TASK LOG: ...` line genuinely appeared in the actual shell running `cypress run` — not inside
Cypress's own command log, not in a browser console. The file write/read round-trip also genuinely
succeeded against the real filesystem, something a sandboxed browser tab (even Cypress's own in-browser
test code) cannot do directly via any `cy` command.

## How It Actually Works

`setupNodeEvents` runs in the real Node.js process behind Cypress — the same real process this course has
confirmed, via process inspection in [Lesson 1](01-why-architecture-is-different.md), sits alongside (but
separate from) the browser's own renderer process. `on("task", {...})` registers real Node functions that
`cy.task("name", arg)` can invoke from a test, crossing from the browser's sandboxed JS context into
genuine, unrestricted Node code — real filesystem access, real database queries, real `console.log` output
to the actual terminal, anything a plain Node script could do. This is the real, structural reason
`cy.task()` exists at all: Cypress's own in-browser commands are deliberately sandboxed the same way any
browser tab's JavaScript is, and `cy.task()` is the one sanctioned bridge back out to the unrestricted
Node environment for the specific, narrow set of things a test genuinely needs that a browser can't do on
its own.

This also explains a real, useful contrast with `cy.log()`: `cy.log()` writes to Cypress's own in-browser
command log, visible in the interactive Test Runner's UI, but it never appears in a `cypress run` terminal
session the way this lesson's `cy.task()` output does — confirmed throughout this course's earlier
lessons, where `cy.log()` calls never once showed up in any captured terminal output.

## Common mistakes

- Reaching for `cy.log()` when debugging a headless `cypress run` and being confused that nothing prints
  to the terminal — this lesson's real result shows `cy.task()` with a Node-side `console.log` is the
  correct tool for that, not `cy.log()`.
- Putting heavy real logic (complex data transformation, slow real I/O) directly inside `setupNodeEvents`
  without considering it blocks that one Node process — a real task that's genuinely slow (a large file
  write, a real database migration) adds that real time to every test that calls it.
- Forgetting that `cy.task()` arguments and return values must be serializable — functions, DOM nodes, or
  other non-JSON-safe values can't cross the real boundary between the browser context and the Node
  process the way this lesson's plain strings and objects did.

## Exercise

Register a new task, `countRealLinesInFile(path)`, that uses Node's real `fs.readFileSync` plus
`.split("\n").length` to count a file's real lines, and call it against this course's own
`cypress.config.js`. Confirm the real returned count matches what a manual `wc -l cypress.config.js` in
the terminal reports — directly verifying `cy.task()`'s real Node-side computation against an independent,
non-Cypress check.
