---
description: "The identical contentDocument workaround from Level 2 could not see a real cross-origin iframe's content -- confirmed against a real second server."
---

# The Same-Origin Constraint, Investigated

## What was run

The exact same `contentDocument` workaround this course verified against a same-origin iframe in
[Level 2, Lesson 5](../level-2/05-iframes-real-limitation.md), run again against a genuinely
cross-origin iframe — a real second Express server on `localhost:5152`, embedded in a page served from
`localhost:5151` — to directly confirm where that workaround's real boundary actually sits.

```javascript
it("fails to reach a real cross-origin iframe's contentDocument", () => {
  cy.visit("/cross-origin-iframe.html"); // iframe src="http://localhost:5152/cross-origin-frame-content"
  cy.get("#cross-frame")
    .its("0.contentDocument.body.innerHTML")
    .should("not.be.empty");
});

it("confirms the same-origin workaround from Level 2 works for a same-origin iframe", () => {
  cy.visit("/iframe.html"); // iframe src is same-origin, localhost:5151
  cy.get("#content-frame")
    .its("0.contentDocument.body.innerHTML")
    .should("not.be.empty");
});
```
```
1) fails to reach a real cross-origin iframe's contentDocument:
   CypressError: Timed out retrying after 4000ms: `cy.its()` errored because the property:
   `0.contentDocument.body.innerHTML` does not exist on your subject.

✓ confirms the same-origin workaround from Level 2 works for a same-origin iframe (41ms)
```

The identical code, the identical Cypress command, against two real iframes differing in exactly one way
(same-origin vs. a genuinely different real server on a different port) — and the real result is
completely different. The cross-origin iframe's `contentDocument` genuinely doesn't exist as an accessible
property at all from the host page's perspective.

## How It Actually Works

This isn't a Cypress limitation — it's the real browser's own, standard same-origin policy, which blocks
any script (Cypress's own in-browser test code included, per
[Lesson 1](01-why-architecture-is-different.md)) from reading a cross-origin iframe's `contentDocument`
property at all. The browser doesn't even return a restricted-but-readable object; it simply refuses to
expose the property, which is exactly why Cypress's own `cy.its()` reports it as not existing rather than
reporting a permission error. [Level 2, Lesson 5](../level-2/05-iframes-real-limitation.md)'s workaround
only ever worked because that iframe happened to be same-origin — this lesson closes the loop that lesson
left open, confirming with a genuinely different real origin exactly where that workaround's limit is.

`cy.origin()` ([Lesson 3](03-multiple-origins.md)) doesn't help here either, despite also dealing with
cross-origin boundaries — `cy.origin()` lets Cypress's *own test commands* run inside a different origin's
context after a full page navigation; it has no mechanism for reading *into* a live cross-origin iframe
embedded within the current page, because the browser's own security model draws that line at the DOM
property level, not at the level of which Cypress API you call.

## Common mistakes

- Assuming any `contentDocument`-based workaround generalizes to all iframes — this lesson's own real,
  side-by-side comparison shows it depends entirely on whether the iframe is same-origin, not on anything
  about how carefully the workaround code is written.
- Reaching for `cy.origin()` to solve a cross-origin-iframe problem — it solves a different real problem
  (commands after a full navigation to a different origin), not reading into an embedded iframe from the
  page that contains it.
- Treating this as something a plugin can cleanly fix — cross-origin iframe content inspection is blocked
  by the real browser's own security model; a plugin can at best offer narrow workarounds (postMessage
  bridging the frame itself implements, or testing the iframe's content as its own separate page via a
  direct `cy.visit()`), not a general bypass.

## Exercise

Modify `cross-origin-frame-content` on the real second server to `postMessage` its own button-click
events back to the parent window, and write a real Cypress test that listens for that message via
`cy.window().then(win => { win.addEventListener("message", ...) })` instead of trying to reach into the
iframe's DOM directly. Confirm whether this real, cooperative pattern — common for payment widgets and
other legitimate cross-origin embeds — can verify the cross-origin button's behavior without ever reading
its `contentDocument`.
