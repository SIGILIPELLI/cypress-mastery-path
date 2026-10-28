---
description: ".select() worked by real value and real visible text on the same dropdown, and checking radio B auto-deselected A with zero extra Cypress code."
---

# Forms & Dropdowns

## What was run

Four real tests against the local test app's form elements: a `<select>` dropdown, a radio-button group, a
full login form submission, and a real add-todo form workflow.

```javascript
it("selects real dropdown options by value and by visible text", () => {
  cy.visit("/elements.html");
  cy.get("#color-select").select("green");
  cy.get("#color-select").should("have.value", "green");
  cy.get("#color-select").select("Blue");
  cy.get("#color-select").should("have.value", "blue");
});

it("selects a real radio button, deselecting the other", () => {
  cy.visit("/elements.html");
  cy.get("#radio-a").check();
  cy.get("#radio-a").should("be.checked");
  cy.get("#radio-b").check();
  cy.get("#radio-b").should("be.checked");
  cy.get("#radio-a").should("not.be.checked");
});
```
```
✓ selects real dropdown options by value and by visible text (358ms)
✓ selects a real radio button, deselecting the other (181ms)
✓ submits a real login form end-to-end (396ms)
✓ adds a real todo via the form and sees it rendered (246ms)
```

`.select("green")` matched the option's real `value` attribute; `.select("Blue")` matched the option's real
visible text — Cypress's `.select()` accepts either form and resolved both correctly against the same
dropdown without any extra configuration. Checking `#radio-b` genuinely deselected `#radio-a`, confirmed
by the real `not.be.checked` assertion — because both inputs share the real HTML `name="choice"` attribute,
which is native browser radio-group behavior, not something Cypress added.

## How It Actually Works

`.select()` is Cypress's one interaction command that doesn't map to a single real user gesture — clicking
a native `<select>` to open it, then clicking an option, is how a person does it, but Cypress's `.select()`
instead directly sets the element's value and dispatches the real `change` event the browser would fire
after that interaction, which is both faster and more reliable across different OS-native dropdown
renderers (which vary enough that reliably clicking into a real open native dropdown cross-platform is
genuinely hard). The radio-group behavior in this lesson's second test required zero Cypress-side logic at
all — `name="choice"` shared across `#radio-a` and `#radio-b` is standard HTML, and the real browser itself
enforces that only one radio in a named group can be checked; Cypress's `.check()` just performs a real
click, and the browser does the rest.

The login and todo-form tests combine commands from every earlier Level 1 lesson — `.type()`, `.click()`,
and an assertion that implicitly waits for the real server round-trip (the login form's real `fetch()` call,
the todo form's real `POST` to `/api/todos`) to finish before the resulting DOM update exists, which is
auto-waiting ([Lesson 6](06-auto-waiting-retry-ability.md)) quietly doing its job again without any
additional code.

## Common mistakes

- Assuming `.select()` needs to match visible text specifically — it accepts either the option's `value`
  attribute or its visible text, confirmed by this lesson using both on the same element.
- Writing a manual `cy.get("#radio-a").should("not.be.checked")` assertion immediately after checking
  `#radio-a` itself — the deselection only happens once a *different* radio in the same group is checked;
  checking the same one again is a safe no-op, not a toggle (same real behavior as `.check()` on a checkbox,
  covered in [Lesson 4](04-interacting-with-elements.md)).
- Forgetting that a form submission's resulting DOM update depends on a real network round-trip completing
  — asserting on `#result` or the todo list immediately after `.click()` works here specifically because
  `.should()` auto-waits for it, not because the update happens synchronously.

## Exercise

Add a `multiple` attribute to `#color-select` in `elements.html`, restart the real Express server, and
change this lesson's first test to select both `"green"` and `"blue"` in one real `.select(["green",
"blue"])` call. Confirm Cypress's own real array-argument form of `.select()` genuinely selects multiple
options at once on a multi-select dropdown, and that `.should("have.value", ...)` needs a different real
assertion shape to check a multi-select's selected values.
