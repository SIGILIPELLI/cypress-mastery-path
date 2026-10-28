---
description: "After deleting the real first todo (id=1), cy.get('@firstTodo') genuinely resolved to the NEW first item (id=2), not a stale reference."
---

# Aliases: as() and @alias

## What was run

Three real alias use cases — a DOM element, an extracted value, and an intercepted network route — plus a
deliberate real experiment to answer a specific question none of the basic examples settle: does
`cy.get("@alias")` return a cached snapshot of the original element, or does it re-query fresh each time?

```javascript
it("aliases a real DOM element and reuses it without re-querying", () => {
  cy.visit("/elements.html");
  cy.get("#text-input").as("input");
  cy.get("@input").type("first");
  cy.get("@input").should("have.value", "first");
});

it("aliases a real intercepted route and waits on it by name", () => {
  cy.intercept("GET", "/api/todos").as("todosCall");
  cy.visit("/todos.html");
  cy.wait("@todosCall").its("response.statusCode").should("eq", 200);
});
```
```
✓ aliases a real DOM element and reuses it without re-querying (312ms)
✓ aliases a real intercepted route and waits on it by name (39ms)
```

## A real, disambiguating experiment: cached reference, or fresh re-query?

```javascript
cy.get(".todo-item").first().as("firstTodo");
cy.get("@firstTodo").invoke("attr", "data-id").then((originalId) => {
  cy.get("@firstTodo").find(".delete-btn").click();
  cy.get("@firstTodo").invoke("attr", "data-id").then((currentId) => {
    expect(currentId).not.to.eq(originalId);
    expect(originalId).to.eq("1");
    expect(currentId).to.eq("2");
  });
});
```
```
✓ confirms cy.get(@alias) re-queries fresh instead of returning a stale cached node (181ms)
```

This is a genuinely clean, real answer: after deleting the real first todo (`data-id="1"`),
`cy.get("@firstTodo")` resolved to a completely **different** real element — the new first item, now
`data-id="2"` — not a stale reference to the deleted node, and not an error either.

## How It Actually Works

A DOM-element alias in Cypress doesn't store a frozen jQuery object the way a plain JavaScript variable
would — it records *how* to re-find that subject, and Cypress's own command queue replays that lookup each
time `cy.get("@alias")` is invoked, applying the same real auto-waiting and retry-ability every other query
gets. This is a genuinely different real behavior from Selenium's `WebElement`, where this course's own
sibling [Selenium Mastery Path](https://sigilipelli.github.io/selenium-mastery-path/level-2/09-file-upload-stale-elements/) demonstrated a `StaleElementReferenceException` thrown the moment an aliased
element's underlying node was removed. Cypress doesn't throw here — it simply re-resolves the alias's
underlying query (`.first()` of `.todo-item`) against the real, current DOM, which is why the result
changed to a different real element rather than erroring. Value aliases (from `.then()`/`.its()`) and
route aliases (from `cy.intercept().as()`) behave differently still: a value alias captures a real,
resolved value at the moment it's created and doesn't re-run afterward; a route alias lets `cy.wait()`
reference a specific intercepted request by name rather than by re-running the intercept setup.

## Common mistakes

- Assuming a DOM alias behaves like Selenium's `WebElement` and will throw a stale-reference error once
  the original node is gone — this lesson's real result shows Cypress instead silently re-resolves to
  whatever currently matches the aliased query, which can hide a bug if you expected the *same* element.
- Treating a value alias (`.its("appInternalCounter").as("startCount")`) as if it re-reads the live value
  on every later `cy.get("@startCount")` — it captured a snapshot at alias-creation time; a changed
  `window.appInternalCounter` afterward would not be reflected without re-aliasing.
- Forgetting that `cy.wait("@routeAlias")` needs the intercept to have actually fired at least once — using
  an alias before the matching request has happened produces a real timeout, not an immediate "no such
  alias" error, which can look confusingly similar to an unrelated timing bug.

## Exercise

Alias `#text-input` on `/elements.html`, type into it, then navigate away with `cy.visit("/todos.html")`
and back to `/elements.html` with `cy.visit("/elements.html")` again before accessing `cy.get("@input")`.
Determine from the real result whether the alias survives a full page navigation the way it survived this
lesson's in-page DOM deletion, or whether a genuinely fresh page load invalidates it differently —
directly testing the real boundary of what "re-resolves the same query" actually covers.
