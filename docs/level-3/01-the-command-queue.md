---
description: "A real recorded log array proved every sync statement after cy.visit()/cy.get() ran BEFORE their real effects -- the .then() callback executed last."
---

# The Command Queue: Why Cypress Code Isn't Actually Async

## What was run

A real test that records the actual order of execution into a plain array, deliberately interleaving
synchronous JavaScript statements with Cypress commands, to directly prove — not just explain — why
Cypress code that *looks* synchronous genuinely isn't.

```javascript
it("demonstrates that commands are enqueued, not executed immediately", () => {
  const log = [];
  log.push("sync-1: before cy.visit() call");
  cy.visit("/elements.html");
  log.push("sync-2: right after cy.visit() call returns");
  cy.get("#text-input").then(() => {
    log.push("async-1: inside a .then() on cy.get()");
  });
  log.push("sync-3: right after cy.get().then() call returns");
  cy.wrap(null).then(() => {
    expect(log).to.deep.eq([
      "sync-1: before cy.visit() call",
      "sync-2: right after cy.visit() call returns",
      "sync-3: right after cy.get().then() call returns",
      "async-1: inside a .then() on cy.get()",
    ]);
  });
});
```
```
✓ demonstrates that commands are enqueued, not executed immediately (80ms)
```

The real recorded order proves it directly: all three plain `log.push()` calls ran **before** the
`.then()` callback's `"async-1"` entry — even though `cy.get("#text-input").then(...)` appears in the code
*before* `"sync-3"`'s `log.push()` call. `cy.visit()` and `cy.get()` didn't actually navigate or query
anything the instant they were called; they only queued that work.

## How It Actually Works

Every `cy` command — `cy.visit()`, `cy.get()`, `.click()`, `.should()` — returns immediately without doing
its real work yet; it just appends an instruction to Cypress's internal command queue and returns a
special, chainable object. The actual navigating, querying, and clicking all happen afterward, once your
synchronous test function has finished running top-to-bottom, when Cypress begins draining that queue one
command at a time. This is why `cy.visit(...); log.push("sync-2")` genuinely runs `log.push` before the
real navigation happens — the navigation hasn't started yet when that line executes; it's merely been
scheduled. The only way to run code *after* a command's real work actually finishes is `.then()` (or
`.should()` with a callback, as seen in [Level 1, Lesson 5](../level-1/05-assertions.md)), which itself
just queues *its own* callback to run once the queue reaches that point — which is exactly why this
lesson's `.then()` callback executed last, after every synchronous statement in the test function had
already run.

This single fact explains several real behaviors covered elsewhere in this course: chaining
([Level 1, Lesson 7](../level-1/07-forms-dropdowns.md)) works because each command in a chain is just the
next queued item, not a literal nested function call; and it's also the root cause of the real,
specific mistake covered next in [Lesson 3](03-retry-ability-pitfalls.md), where code that *looks* like it
reads a value synchronously actually reads a stale one because the real value isn't available until the
queue reaches that point.

## Common mistakes

- Writing `const text = cy.get("#el").text()` and expecting `text` to hold a real string immediately — `cy`
  commands never return real values synchronously; this lesson's own recorded order proves nothing from a
  queued command is available until its actual turn comes, inside a `.then()` or similar callback.
- Assuming `async`/`await` makes Cypress commands behave like normal promises — Cypress deliberately
  doesn't support `await` on `cy` commands (the command queue predates and intentionally differs from
  plain promise chaining); `.then()` remains the real mechanism for sequencing.
- Debugging a confusing test by adding `console.log()` calls interleaved with `cy` commands and trusting
  their printed order at face value — this lesson's own experiment shows that order reflects when the
  *synchronous* code ran, which is almost never the same moment the corresponding real command's effect
  actually happened.

## Exercise

Add a fourth `log.push()` call directly inside the body of a `.should(($el) => { ... })` callback (the
retrying form from [Level 1, Lesson 5](../level-1/05-assertions.md)) targeting an element that doesn't
exist for the first `1000ms`. Re-run this lesson's recorded-order test and determine, from the real
resulting array, whether that callback's log entry appears once or multiple times — directly observing
whether a retried `.should()` callback re-queues itself fresh on every retry attempt.
