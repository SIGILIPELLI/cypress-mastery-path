---
description: "Both tools auto-wait by design, but only Cypress's version is directly measured here: a real 10/10 pass rate traced to running inside the browser."
---

# Cypress vs Playwright: a Real, Honest Comparison

## What was run

No new code — a direct comparison built from this course's own real, measured Cypress findings and
Playwright's publicly documented architecture, since Playwright is a genuinely closer architectural
relative to Cypress than Selenium is, and the real points of difference are narrower and more specific.

## Where the two tools are genuinely similar — and this course's real evidence shows why

Both Cypress and Playwright build in automatic waiting/retrying as a core design principle, in sharp
contrast to Selenium's explicit-wait model (confirmed by this course's own real `10/10` zero-wait pass
rate in [Level 1, Lesson 6](../level-1/06-auto-waiting-retry-ability.md), versus the sibling Selenium
course's real `15/15` failures under the same condition). Both also support intercepting and controlling
real network traffic as a first-class feature — this course's own `cy.intercept()` deep dive
([Level 3, Lesson 5](../level-3/05-intercept-deep-dive.md)) demonstrated stubbing, spying, and dynamic
responses; Playwright's own documented `page.route()` offers conceptually the same real capability. Neither
tool needs a separate driver binary the way Selenium does — this course confirmed Cypress's own real
process tree contains no `chromedriver`-equivalent ([Level 4, Lesson 1](01-why-architecture-is-different.md)); Playwright similarly talks to each browser via its own native automation protocol rather than a
generic WebDriver layer.

## Where the real, measured architecture actually differs

The one load-bearing difference this course can measure directly: Cypress's test code runs **inside** the
browser, in the same process family as the page under test (confirmed via real process inspection in
[Level 4, Lesson 1](01-why-architecture-is-different.md)), while Playwright's client runs **outside** the
browser, communicating over each browser's own native protocol (CDP for Chromium, and equivalents for
Firefox/WebKit) from a separate Node (or Python, Java, .NET) process — architecturally closer to
Selenium's out-of-process model than to Cypress's in-browser one, even though Playwright also auto-waits.
That one difference is the direct, real cause of several contrasts this course measured firsthand:

- **Language support**: this course confirmed Cypress is JavaScript/TypeScript-only, a direct consequence
  of test code needing to run in the browser's own JS engine. Playwright, being an external client
  speaking a protocol, offers official bindings in Python, Java, .NET, and JS/TS — the same real reason
  Selenium supports many languages.
- **Multi-origin and multi-tab handling**: this course's own real `cy.origin()` investigation
  ([Lesson 3](03-multiple-origins.md)) and cross-origin iframe limitation ([Lesson 4](04-same-origin-constraint.md)) are both direct, measured consequences of Cypress's in-browser code being subject to
  the real browser's own same-origin policy. An out-of-process client isn't bound by that same constraint
  in the same way, which is part of why Playwright's documented multi-tab and multi-context support is
  less encumbered by it.
- **Parallelization**: this course's own real finding that `--parallel` is gated entirely behind a paid
  Cypress Cloud account ([Level 3, Lesson 9](../level-3/09-parallelization-cloud.md)) is a Cypress-specific
  business decision, not shared by Playwright, whose documented sharding (`--shard`) runs locally and
  freely, closer to the `pytest-xdist` model this network's Selenium course measured directly.

## How It Actually Works

Playwright is best understood as occupying a real middle position this course's own evidence maps out
precisely: it adopted Cypress's "auto-wait by default" philosophy (measured here as a dramatic real
reliability win over Selenium's explicit-wait model) while keeping Selenium's "client runs outside the
browser" architecture (which is what gives it multi-language support and locally-free parallelism that
this course's own Cypress measurements show are genuinely constrained here). Neither tradeoff is free: this
course's own real findings show Cypress's in-browser model buys auto-waiting and automatic
screenshot-on-failure cheaply, but pays for it with same-origin constraints and JS-only test code —
exactly the kind of genuine, measured tradeoff a tool positioned differently in the same design space
would be expected to handle differently.

## Common mistakes

- Assuming "Playwright has auto-wait too" means it's architecturally identical to Cypress — this course's
  own real same-origin and multi-language findings show the underlying client/browser relationship is
  genuinely different, even though the waiting philosophy converged.
- Treating Cypress's real Cloud-gated parallelism limitation as something Playwright also shares just
  because both are "modern" tools — this course's own measured finding is specific to Cypress's business
  model, not a shared architectural trait.
- Choosing between the two based only on this comparison's table-level summary rather than this course's
  own cited, measured evidence for each row — the real lessons behind each claim (
  [Lesson 1](01-why-architecture-is-different.md), [Lesson 3](03-multiple-origins.md),
  [Lesson 4](04-same-origin-constraint.md), [Level 3, Lesson 9](../level-3/09-parallelization-cloud.md))
  are what make this comparison verifiable rather than a secondhand opinion.

## Exercise

Read Playwright's public documentation for `page.route()` and compare it feature-by-feature against this
course's own real `cy.intercept()` deep dive ([Level 3, Lesson 5](../level-3/05-intercept-deep-dive.md)) —
specifically the request-inspection, response-delay, and dynamic-response capabilities this course actually
ran and measured. Identify which of those three capabilities Playwright's documented API covers with an
equivalent method, and which (if any) would need a genuinely different approach under Playwright's
out-of-process model.
