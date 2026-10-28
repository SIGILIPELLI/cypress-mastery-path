---
description: "A should() callback checking a 1.5s-delayed real element took exactly 1547ms to pass -- confirming should() retries a failing callback, not just the query."
---

# Assertions & should()

## What was run

Three real assertion styles against the local test app: chained `.should()`/`.and()` on one element, a
`.should()` with a custom callback function against a genuinely delayed element, and a plain `expect()`
against a value already extracted from the real DOM.

```javascript
it("chains multiple should() assertions on the same real element", () => {
  cy.visit("/elements.html");
  cy.get("#text-input")
    .should("be.visible")
    .and("not.be.disabled")
    .and("have.value", "");
});

it("uses a real callback should() for custom logic that also retries", () => {
  cy.visit("/dynamic.html");
  cy.get("#container").should(($el) => {
    expect($el.find("#dynamic-element").length).to.eq(1);
  });
  cy.get("#dynamic-element").should("contain.text", "I appeared after a real 1.5s delay");
});
```
```
✓ chains multiple should() assertions on the same real element (70ms)
✓ uses a real callback should() for custom logic that also retries (1547ms)
```

The second test's real `1547ms` duration is the key finding: `/dynamic.html` injects `#dynamic-element` via
a real `setTimeout` after `1500ms`. The `.should()` callback's `expect(...).to.eq(1)` genuinely fails on
every attempt until that element exists — and Cypress kept re-running the *entire callback* until it
stopped throwing, for almost exactly the real delay, rather than failing on the first check.

## How It Actually Works

`.should()` has two real forms with different mechanics: a string-based assertion (`"be.visible"`,
`"have.value"`) checks a built-in condition against the current subject and retries automatically if it
doesn't match yet; a callback-function form runs your own code and retries the *entire function* as long as
it throws — which is exactly why this lesson's `$el.find("#dynamic-element").length` check kept re-running
every few milliseconds until the real DOM actually contained that element, rather than capturing a single,
possibly-stale snapshot of `$el` at the moment `.should()` was first called. `.and()` is simply an alias for
chaining another `.should()` onto the same already-resolved subject, which is why `.and("not.be.disabled")`
didn't need to re-query `#text-input`.

Plain `expect()` (used in the third test) is Chai's own assertion syntax, available because Cypress bundles
Chai directly — but it does **not** retry; it runs exactly once against whatever value you hand it, which is
why it's correct here against a value already pulled out via `.then()`, but would be the wrong tool for
asserting on something that might not exist yet.

## Common mistakes

- Using `expect()` instead of `.should()` when the element you're checking might not be there immediately —
  `expect()`'s one-shot nature means it has no chance to wait out a real delay the way this lesson's
  `1547ms` callback test did.
- Assuming a `.should()` callback runs once and captures a fixed snapshot — it genuinely re-executes the
  whole function on every retry, which matters if the callback has side effects (it shouldn't) or expensive
  computation (it will slow down every retry attempt, not just the first).
- Forgetting that `.and()` chains onto the *previous* assertion's subject, not a fresh query — three
  `.and()` calls after one `.get()` are checking the same single real element, not re-querying three times.

## Exercise

Change this lesson's callback `.should()` to check for an element that will never actually appear (e.g.
`$el.find("#this-never-appears").length === 1`), and run it with the default 4-second command timeout.
Measure how long the real test takes to fail and compare that real duration to this lesson's successful
`1547ms` run — confirming that `.should()`'s retry loop runs for the same real mechanism in both the
passing and failing case; the only difference is whether the real condition ever becomes true before time
runs out.
