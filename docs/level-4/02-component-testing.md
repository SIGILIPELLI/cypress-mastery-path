---
description: "4 real setup errors, each specific and fixable, stood between an empty config and a passing React component test, with no server or page needed."
---

# Component Testing

## What was run

A real, from-scratch Cypress component-testing setup — a real React component (`Counter.jsx`, local
state, no server calls), Vite as the dev server, and `cy.mount()` — built by actually working through each
real configuration error Cypress produced, rather than copying a finished config and assuming it.

```jsx
// src/Counter.jsx
export default function Counter() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <p id="count-value">{count}</p>
      <button id="increment-btn" onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

```jsx
// cypress/component/Counter.cy.jsx
it("increments on a real click, entirely in isolation", () => {
  cy.mount(<Counter />);
  cy.get("#increment-btn").click();
  cy.get("#increment-btn").click();
  cy.get("#count-value").should("have.text", "2");
});
```
```
✓ renders with a real initial count of 0, with no server or page involved (40ms)
✓ increments on a real click, entirely in isolation (178ms)
```

## Four real, specific errors on the way to a working setup

Each was fixed in order, and each error message was specific enough to say exactly what was missing:

1. `"vite for your dev server, but a configuration file was not found"` → added a real `vite.config.js`.
2. `"ENOENT: ... component-index.html"` → added a real `cypress/support/component-index.html`.
3. `"cy.mount() must be implemented by the user"` → registered `Cypress.Commands.add("mount", mount)` from
   `@cypress/react` in a real component support file.
4. `"No element found that matches selector [data-cy-root]"` → added `<div data-cy-root></div>` to the
   real index HTML — the specific attribute Cypress's own mount helper looks for to attach the component.

None of these were Cypress bugs — each was a real, missing piece of a from-scratch setup, and each error
message pointed directly at the fix.

## How It Actually Works

Component testing mounts a real component directly into a real browser page Cypress controls — no
`cy.visit()`, no server, no full application route — using a real dev server (Vite here) to compile just
that one component and its dependencies on demand. `cy.mount()` is not a built-in Cypress command; it's a
thin wrapper this lesson registered itself around `@cypress/react`'s real `mount()` function, which is why
step 3's error explicitly says it "must be implemented by the user" — Cypress deliberately doesn't assume
which UI framework (React, Vue, Angular, Svelte) a project uses, so mounting logic is framework-specific
and supplied by a small adapter package. The `data-cy-root` element from step 4 is simply the real DOM node
that adapter looks for to attach the mounted component — ordinary React rendering, pointed at a specific
real element Cypress's tooling already knows to query.

This is architecturally different from every earlier lesson in this course: there is no real Express
server involved in `Counter.cy.jsx` at all — confirmed by the real speed difference between this lesson's
component test durations (`40ms`, `178ms`) and the full page-driven [Level 1 capstone](../level-1/10-project-login-flow-test.md)'s per-test durations (`458ms`, `373ms`, `378ms`), which each pay the real cost of
a full page navigation and a real server round-trip that component testing skips entirely.

## Common mistakes

- Assuming component testing requires a full backend or routing setup — this lesson's `Counter` component
  has zero server dependency, and the test mounted and verified it with none either.
- Copying a finished `cypress.config.js`'s `component` block from documentation without also creating the
  real supporting files (`component-index.html`, a mount-registering support file) it silently depends on
  — this lesson's own 4 sequential real errors are exactly what happens when those pieces are missing.
- Treating component tests as a replacement for the E2E tests this course spent 3 levels building — they
  answer a genuinely different question (does this one component behave correctly in isolation?) than an
  E2E test (does the real, full user flow work end to end?); this course's real speed difference shows why
  component tests are faster, not why they're a strict upgrade.

## Exercise

Add a second React component, `Greeting.jsx`, that accepts a `name` prop and renders `Hello, {name}!`, and
write a real component test mounting it with `cy.mount(<Greeting name="Cypress" />)`, asserting on the
real rendered text. Confirm whether any of this lesson's 4 real setup errors recur for a second component
in the same project, or whether the shared `vite.config.js`/`component-index.html`/mount registration from
this lesson's setup already covers it.
