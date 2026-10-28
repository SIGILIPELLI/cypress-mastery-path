---
description: "Level 3: the command queue, aliases, retry-ability pitfalls, cookies/cy.session, cy.intercept internals, screenshots, flaky-test diagnosis, CI, and parallelization."
---

# Level 3 · Architecture & Advanced Techniques

Cypress code looks synchronous but genuinely isn't — and that single fact explains nearly every confusing
Cypress bug you'll ever hit. This level builds real understanding of the command queue itself, aliases for
referencing values across the queue, the real limits of retry-ability, session caching with `cy.session`,
a deeper real look at `cy.intercept`, and the specific real techniques for diagnosing flaky tests and
running Cypress inside a CI pipeline.

By the end of this level you'll understand exactly why Cypress code executes the way it does, not just how
to write it, and you'll have built a real Page-Object-style framework with custom commands as this level's
project.
