---
description: "Level 2: custom commands, fixtures, network stubbing with cy.intercept, cy.request, iframes, file upload, env config, hooks, and debugging."
---

# Level 2 · Intermediate Patterns

A handful of passing tests isn't a maintainable suite. This level builds the real patterns that scale: custom
commands to stop repeating yourself, fixtures for real test data, and Cypress's own signature capability —
`cy.intercept` for genuinely controlling the network layer — plus calling APIs directly, handling file
uploads, environment configuration, test hooks, and the real tools for debugging a test that fails.

By the end of this level you'll have a real suite that stubs and asserts against real network requests,
and you'll know exactly which Cypress feature to reach for when a test needs real data, a custom action, or
genuine control over what the browser actually receives from the server.
