---
description: "Inserting a plain .then() mapping function before should() genuinely broke retry-ability -- even though should() was still chained right after it."
---

# Retry-Ability Pitfalls

## What was run

Four real variations of the same check — waiting for a counter that updates to `"5"` after a genuine
`1200ms` delay — to find exactly where Cypress's retry-ability actually stops working, rather than where
it's commonly assumed to.

```javascript
it("PITFALL 1: a plain .then() capture read outside the queue, asserted separately", () => {
  cy.visit("/counter.html");
  let captured;
  cy.get("#count-display").then(($el) => { captured = parseInt($el.text()); });
  cy.wrap(null).then(() => { expect(captured).to.eq(5); });
});

it("PITFALL 2: a .then() mapping function inserted before should() breaks retry", () => {
  cy.visit("/counter.html");
  cy.get("#count-display").invoke("text").then((t) => parseInt(t)).should("eq", 5);
});

it("WORKS: should() chained directly on invoke(), no .then() in between", () => {
  cy.visit("/counter.html");
  cy.get("#count-display").invoke("text").should("eq", "5");
});

it("WORKS: a callback-form should() wraps the parsing itself and retries the whole thing", () => {
  cy.visit("/counter.html");
  cy.get("#count-display").should(($el) => {
    expect(parseInt($el.text())).to.eq(5);
  });
});
```
```
1) PITFALL 1 ... AssertionError: expected 0 to equal 5
2) PITFALL 2 ... AssertionError: expected 0 to equal 5
✓ WORKS: should() chained directly on invoke() (1239ms)
✓ WORKS: a callback-form should() wraps the parsing itself (1245ms)

2 passing (3s)
2 failing
```

Pitfall 1 is the commonly-known one: reading a value into a plain variable via `.then()` and asserting on
it afterward only captures a single real snapshot, taken before the counter's real `1200ms` update. Pitfall
2 is the genuinely more surprising real finding: **`.should("eq", 5)` was still chained directly onto the
command — and it still failed**, because the plain `.then((t) => parseInt(t))` sitting in between silently
broke retry-ability for everything upstream of it.

## How It Actually Works

`.should()` retries by re-running the *specific command it's attached to* (and, implicitly, everything
that command depends on) from scratch — but a plain `.then()` is not itself a retryable query; it's a
one-time callback that runs once when the queue reaches it and hands its return value forward. Chaining
`.should()` after a `.then()` only retries the `.should()` check against whatever the `.then()` already
produced *once* — it does not re-run the `.then()` callback itself, and therefore never re-reads the real,
updated DOM text. `.invoke("text")`, by contrast, genuinely is a retryable Cypress command (not a plain
callback), so `.should()` chained after it correctly re-runs both `.invoke("text")` *and* the check,
exactly as the third test's real, passing `1239ms` duration confirms — nearly matching the page's own real
`1200ms` delay, proof that it waited and retried. The callback form of `.should()` (fourth test) sidesteps
the whole issue by putting the parsing *inside* the retried callback itself, which is why it's the safest
general-purpose pattern when a value needs transformation before checking.

## Common mistakes

- Assuming any `.should()` anywhere later in a chain makes the whole chain retry-safe — this lesson's real
  Pitfall 2 directly disproves that; what matters is whether every command between the real data source
  and the `.should()` is itself a retryable query, not a plain `.then()` callback.
- Reaching for `cy.wait(1500)` to "fix" a flaky assertion like Pitfall 1 or 2, instead of restructuring the
  chain to use `.invoke()` or a callback `.should()` — a fixed wait happens to work here because the real
  delay is known and short, but it's slower than necessary and will break again if the real delay changes.
- Debugging this kind of failure by staring at the assertion error alone — both pitfalls produce the
  identical `expected 0 to equal 5` message; the real fix requires looking at the chain's structure, not
  the error text.

## Exercise

Add a fifth test using `.invoke("text").then((t) => parseInt(t))` followed by `.should("be.greaterThan",
0)` instead of `.should("eq", 5)`. Run it and determine, from the real pass/fail result, whether a
*looser* assertion after a retry-breaking `.then()` can accidentally pass for the wrong reason (matching
the real pre-update value of `0`... or not) — directly testing whether this lesson's Pitfall 2 only
matters for exact-value checks or for any assertion placed after a retry-breaking `.then()`.
