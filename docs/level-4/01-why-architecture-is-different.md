---
description: "The real process tree for one cypress run showed only Cypress's own Electron-family processes -- zero separate driver binary, confirmed directly."
---

# Why Cypress's Architecture Is Different

## What was run

A direct real process-tree capture during a live `cypress run`, to confirm at the operating-system level
what every earlier lesson in this course has already demonstrated indirectly: Cypress runs as its own
process family, with no separate driver binary anywhere in the picture.

```bash
npx cypress run --spec cypress/e2e/smoke.cy.js &
ps aux | grep -i "cypress\|electron"
```
```
.../Cypress.app/.../Cypress --no-sandbox -- --run-project ... --spec cypress/e2e/smoke.cy.js
.../Cypress Helper (Renderer).app/.../Cypress Helper (Renderer) --type=renderer ...
.../Cypress Helper.app/.../Cypress Helper --type=utility --utility-sub-type=network.mojom.NetworkService ...
.../Cypress Helper.app/.../Cypress Helper --type=gpu-process ...
Cypress: Config Manager
```

Every process belongs to Cypress's own Electron-based application family — a main process, a renderer
process (the actual browser tab running the test page), a GPU process, a network service process, and a
small config-manager helper. There is no `chromedriver`, no `geckodriver`, no separate driver binary of
any kind in this real process list.

## This course's own full, accumulated real evidence for the architectural claim

Every earlier real finding in this course traces back to the same single fact — Cypress's test code runs
*inside* the browser, in the same process family as the page under test:

- [Level 1, Lesson 1](../level-1/01-what-cypress-actually-is.md): `cy.window()` returned a real, live,
  directly-mutable reference to the page's own `window` object — only possible because the test code and
  the page share the same JS execution context.
- [Level 1, Lesson 6](../level-1/06-auto-waiting-retry-ability.md): a real `10/10` pass rate with zero
  manual waits — auto-waiting works cheaply because Cypress's own retry loop runs inside the browser
  itself, not over a separate network protocol.
- [Level 2, Lesson 5](../level-2/05-iframes-real-limitation.md): a real, working `contentDocument`
  workaround reached a same-origin iframe's content directly, using the DOM's own native API, not a
  WebDriver-style frame-switching command.
- [Level 3, Lesson 1](../level-3/01-the-command-queue.md): the real recorded execution order proved
  Cypress commands queue rather than execute immediately — a direct consequence of needing to coordinate
  with the browser's own event loop rather than sending one HTTP command at a time.

## How It Actually Works

A Selenium or Playwright client is a genuinely separate process (or even a separate machine, via Grid)
sending commands over a wire protocol to a browser it controls from the outside — this course's own sibling
[Selenium Mastery Path](https://sigilipelli.github.io/selenium-mastery-path/level-1/01-what-webdriver-does/) captured that real HTTP traffic directly. Cypress collapses the "test driver" and "browser" into one
real process family, injecting its own test-running code directly into the browser alongside the
application under test. This single architectural choice is the real, common cause behind nearly every
distinctive Cypress behavior this course has measured: free, built-in auto-waiting (no network round-trip
per retry check); JavaScript-only test code (it must run in the same JS engine as the page); the
same-origin constraint covered next in [Lesson 4](04-same-origin-constraint.md) (in-browser code is itself
subject to the browser's own cross-origin security rules); and no local, free parallelism
([Level 3, Lesson 9](../level-3/09-parallelization-cloud.md)) — splitting work across multiple real
machines needs a coordinating service (Cypress Cloud), not just more local worker processes, because each
Cypress instance *is* a full browser-plus-test-runner unit, not a lightweight client.

## Common mistakes

- Treating Cypress's architecture as a minor implementation detail rather than the direct cause of its
  most distinctive real behaviors — every item in this lesson's evidence list is a measured consequence of
  this one architectural fact, not an unrelated feature list.
- Assuming "in-browser" means "less powerful" — this course's own real measurements show the opposite for
  several capabilities (auto-waiting's `10/10` reliability, `cy.window()`'s direct state access) precisely
  because of this architecture, even though it also genuinely costs Cypress some capabilities (native
  multi-tab support, true free local parallelism) that an out-of-process tool can offer more naturally.
- Expecting this architecture to change between Cypress versions without a major rewrite — it's a
  foundational design decision, not a tunable setting; the genuinely new real findings this course hit
  along the way (the `Cypress.env()` removal, the Electron deprecation warning) were API surface changes,
  not architectural ones.

## Exercise

Using this lesson's real process-capture technique (`ps aux | grep -i cypress` during a live run), compare
the real process count and total memory footprint of a `cypress run` against bundled Electron versus
`cypress run --browser chrome` (from [Level 1, Lesson 9](../level-1/09-cypress-app-headless-mode.md)).
Determine whether driving an external, already-installed browser instead of the bundled one changes the
real process tree's shape in a way that matters for this architectural picture, or whether it's simply a
different real browser engine filling the same structural role.
