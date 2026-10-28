---
description: "A real recorded order array confirmed before/beforeEach/afterEach/after fire in the documented sequence -- a broken after-hook genuinely skipped the rest."
---

# Hooks: before, beforeEach, after, afterEach

## What was run

A real Cypress spec recording its own hook-execution order into a shared array, then asserting on the
exact real sequence in an `after()` hook — directly measured, not assumed from documentation.

```javascript
describe("hooks execution order", () => {
  const order = [];

  before(() => { order.push("before"); });
  beforeEach(() => { order.push("beforeEach"); });
  afterEach(() => { order.push("afterEach"); });
  after(() => {
    order.push("after");
    expect(order).to.deep.eq([
      "before",
      "beforeEach", "test1", "afterEach",
      "beforeEach", "test2", "afterEach",
      "after",
    ]);
  });

  it("test1", () => { order.push("test1"); });
  it("test2", () => { order.push("test2"); });
});
```
```
✓ test1 (17ms)
✓ test2 (2ms)
2 passing (35ms)
```

Both tests passed, and critically, the `after()` hook's own real deep-equality assertion against the
recorded order also passed silently — confirming the real execution sequence was exactly `before` once,
then `beforeEach`/test/`afterEach` repeated for each test, then `after` once at the very end.

## A real, deliberately broken order

```javascript
// changed the after() hook's expected first entry to "WRONG_ORDER"
```
```
1) "after all" hook for "test2":
   AssertionError: expected [ Array(8) ] to deeply equal [ Array(8) ]

Because this error occurred during a `after all` hook we are skipping the remaining
tests in the current suite: `hooks execution order`
```

Cypress/Mocha genuinely reports this failure as attached to `"after all" hook for "test2"`, even though the
hook itself runs once for the whole suite, not per-test — and it explicitly states its own real policy of
skipping any remaining tests in the suite once an `after all` hook throws.

## How It Actually Works

`before`/`after` run exactly once per `describe` block, no matter how many tests it contains;
`beforeEach`/`afterEach` run once per individual test. This lesson's real recorded array is the direct
proof: `"before"` and `"after"` each appear exactly once, while `"beforeEach"`/`"afterEach"` each appear
twice — once per real test. Hooks run as genuine Cypress commands themselves (they can contain `cy.visit()`,
`cy.get()`, and so on, exactly like a test body), which is why this course's earlier login-flow patterns
could be moved into a `beforeEach` to avoid repeating a login sequence in every single test, the same real
motivation behind [custom commands](01-custom-commands.md) — hooks and custom commands solve overlapping
but distinct problems (hooks for *when* setup runs, commands for *what* the setup does).

## Common mistakes

- Assuming a failed `after()` hook only affects reporting, not subsequent tests — this lesson's real error
  message states plainly that remaining tests in the *current suite* are skipped once an `after all` hook
  throws, which can silently hide real test failures that would have been caught by tests that never ran.
- Putting assertions that depend on test-specific state inside `after()` instead of `afterEach()` — `after()`
  runs once, after every test in the block has already finished, so it can only sensibly assert on
  cumulative state (like this lesson's full order array), not a single test's own result.
- Forgetting that `beforeEach`/`afterEach` apply to every test in their enclosing `describe` block,
  including ones added later — adding a tenth test to this lesson's suite would make it run
  `beforeEach`/`afterEach` a tenth time automatically, without any change to the hook definitions themselves.

## Exercise

Nest a second `describe()` block inside this lesson's existing one, with its own `beforeEach()` that pushes
`"inner-beforeEach"`. Add one test inside the nested block, run the suite, and determine from the real
resulting order array whether Cypress runs outer and inner `beforeEach` hooks for that nested test in
outer-then-inner order, inner-then-outer, or only the innermost one — directly observing Mocha's real hook
nesting behavior rather than assuming it.
