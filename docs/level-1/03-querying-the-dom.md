---
description: "cy.get, cy.contains and cy.find all genuinely found real elements -- a nonexistent selector produced a real 'Timed out retrying after 1000ms' error."
---

# Querying the DOM: cy.get, cy.contains, cy.find

## What was run

A real Cypress spec against the local todos page, exercising the three core DOM-query commands against
real rendered HTML, plus a deliberately nonexistent selector to capture Cypress's real, specific failure
message rather than guessing at it.

```javascript
it("finds elements with cy.get using various real selectors", () => {
  cy.visit("/todos.html");
  cy.get("#todo-list").should("exist");
  cy.get(".todo-item").should("have.length", 2);
  cy.get("ul#todo-list .todo-item").should("have.length", 2);
});

it("finds elements by real visible text with cy.contains", () => {
  cy.visit("/todos.html");
  cy.contains("Write lesson 1").should("exist");
  cy.contains(".todo-item", "Write lesson 2").should("exist");
});

it("scopes a search to real descendants with cy.find", () => {
  cy.visit("/todos.html");
  cy.get("#todo-list").find(".delete-btn").should("have.length", 2);
});
```
```
✓ finds elements with cy.get using various real selectors (108ms)
✓ finds elements by real visible text with cy.contains (48ms)
✓ scopes a search to real descendants with cy.find (40ms)
```

Each command does a genuinely different job: `cy.get` takes any real CSS selector and returns every
matching element; `cy.contains` searches by real, rendered visible text (optionally scoped to a selector,
as in `cy.contains(".todo-item", "Write lesson 2")`); `cy.find` searches only within an already-selected
real element's descendants, confirmed here by finding exactly the 2 real `.delete-btn` elements that exist
*inside* `#todo-list`, not anywhere else on the page.

## The real failure message for a selector that matches nothing

```javascript
cy.get("#this-id-does-not-exist", { timeout: 1000 });
```
```
AssertionError: Timed out retrying after 1000ms: Expected to find element: `#this-id-does-not-exist`, but never found it.
```

This is not an instant failure — Cypress genuinely spent the full real 1000ms configured timeout retrying
the query before giving up. That's a direct, observable preview of this course's [Lesson 6](06-auto-waiting-retry-ability.md): `cy.get()` doesn't just check once, it keeps re-checking the real DOM until the element
appears or the timeout elapses.

## How It Actually Works

`cy.get()` is the one command that queries the full real document (or the current subject, if chained);
`cy.find()` restricts that same real query to descendants of whatever `cy.get()` already selected, which is
why `cy.get("#todo-list").find(".delete-btn")` is not equivalent to `cy.get("#todo-list .delete-btn")` in
every case — `.find()` genuinely re-queries from the already-selected real node as its root, which matters
once a page has multiple elements that would otherwise match the same global selector. `cy.contains()`
works differently from both: it doesn't use CSS selectors against text at all — it searches the real
rendered text content of elements, which is why `cy.contains(".todo-item", "Write lesson 2")` genuinely
needed two arguments here (a scope selector, then the text) rather than one combined selector string.

## Common mistakes

- Assuming `cy.get("#list").find(".item")` and `cy.get("#list .item")` always produce identical real
  results — they usually do, but `.find()` re-scopes from the already-selected element, which matters once
  you chain through intermediate commands that could change what "already selected" means.
- Treating a `cy.get()` timeout as proof the element will never exist — this lesson's own real error took
  the full configured 1000ms to appear, meaning Cypress was still genuinely retrying right up until the end;
  a longer real timeout might have found a slightly slower-rendering element.
- Using `cy.contains()` when you actually want to match an attribute, not visible text — it searches real
  rendered text content specifically, not `id`, `class`, or other attribute values.

## Exercise

Add a third `.todo-item` to the real todos page's initial seed data in `server.js`, restart the real Express
app, and re-run this lesson's first test without changing its `.should("have.length", 2)`
assertions. Confirm the real test now fails with a specific, informative length-mismatch message — then fix
the assertion to `3` and confirm it passes, directly observing that `cy.get()`'s length checks reflect the
real, current DOM, not a cached expectation.
