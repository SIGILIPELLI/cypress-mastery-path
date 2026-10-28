---
description: "cy.click() on a real disabled button refused after retrying 4050ms with a specific error -- Selenium's own .click() raises nothing on the same element."
---

# Interacting With Elements: Type, Click, Check

## What was run

A real Cypress spec against the local `/elements.html` page, exercising `.type()`, `.check()`/`.uncheck()`,
and `.click()` against real form elements — including a deliberate attempt to click a real `disabled`
button, to see exactly what Cypress actually does rather than assuming.

```javascript
it("types into a real text input", () => {
  cy.visit("/elements.html");
  cy.get("#text-input").type("real cypress text");
  cy.get("#text-input").should("have.value", "real cypress text");
});

it("checks and unchecks a real checkbox", () => {
  cy.visit("/elements.html");
  cy.get("#checkbox-1").should("not.be.checked");
  cy.get("#checkbox-1").check();
  cy.get("#checkbox-1").should("be.checked");
  cy.get("#checkbox-1").uncheck();
  cy.get("#checkbox-1").should("not.be.checked");
});

it("clicks a real enabled button and updates real state", () => {
  cy.visit("/elements.html");
  cy.get("#action-btn").click();
  cy.get("#status-text").should("have.text", "action button clicked");
});
```
```
✓ types into a real text input (214ms)
✓ checks and unchecks a real checkbox (175ms)
✓ clicks a real enabled button and updates real state (94ms)
```

All three commands worked exactly as expected against real, interactable elements.

## A real, deliberate click on a disabled button

```javascript
cy.get("#disabled-btn").click({ force: false });
```
```
CypressError: Timed out retrying after 4050ms: `cy.click()` failed because this element is `disabled`:

`<button id="disabled-btn" disabled="" ...>Disabled</button>`

Fix this problem, or use `{force: true}` to disable error checking.
```

This is a genuinely different real result from this network's own
[Selenium Mastery Path](https://sigilipelli.github.io/selenium-mastery-path/level-1/06-interacting-with-elements/),
which found that Selenium's `.click()` on an identical disabled button raises **no exception at all** and
simply does nothing silently. Cypress's real `.click()` actively checks an element's actionability
(visible, not disabled, not covered) before clicking, retried for a real 4050ms, and failed loudly with a
specific, actionable error — rather than succeeding at nothing and leaving you to notice the lack of effect
yourself.

## How It Actually Works

Cypress's interaction commands (`.type()`, `.click()`, `.check()`) all perform a real actionability check
before acting: is the element visible, not disabled, not detached from the DOM, and not covered by another
element? Selenium's `.click()`, demonstrated as *not* performing this check in the sibling Selenium course,
sends the click command regardless and lets the browser's own native click handling decide what happens
(which, for a genuinely disabled button, is nothing). Cypress's own in-browser architecture
([Lesson 1](01-what-cypress-actually-is.md)) is exactly what makes this proactive check possible — Cypress's
test code is running right there in the browser and can inspect the real element's actual state before
deciding whether to proceed, rather than firing a command blind over a wire protocol.

`.check()` and `.uncheck()` are themselves thin wrappers that assert the element is a checkbox or radio
input, then click it if its checked state doesn't already match the real target state — which is why calling
`.check()` on an already-checked box is a safe no-op rather than a toggle.

## Common mistakes

- Assuming `cy.click()`'s silence on success means it behaves like Selenium's silence on a disabled-button
  no-op — the two are opposite outcomes that happen to both be "no visible error" in casual reading; always
  check what state actually changed, not just whether an exception was thrown.
- Reaching for `{ force: true }` immediately after a real actionability failure, without first checking
  whether the element is disabled *on purpose* (as this lesson's real button genuinely is) — forcing a click
  past a real, intentional disabled state usually indicates a test bug, not a Cypress limitation.
- Calling `.check()` expecting toggle behavior — it only ever moves toward checked; use `.uncheck()`
  explicitly for the other direction, as this lesson's test does.

## Exercise

Change `#disabled-btn` in `elements.html` to remove the `disabled` attribute, restart the real Express
server, and re-run this lesson's click test with `{ force: false }` still set. Confirm the real test now
passes and that `#status-text` genuinely updates — directly observing that Cypress's actionability check is
tied to the element's real, current state, not a cached assumption from an earlier page load.
