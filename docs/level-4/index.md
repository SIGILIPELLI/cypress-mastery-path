---
description: "Level 4: Cypress's in-browser architecture, component testing, cy.origin, the same-origin constraint, plugins, performance, and Cypress vs Selenium/Playwright."
---

# Level 4 · Component Testing, Scaling & Comparisons

A test framework that works for one person, running one spec file, locally, still has real production
concerns left to solve: testing components in isolation rather than full pages, handling multiple origins,
extending Cypress with plugins and `cy.task`, and running the whole suite at scale. This level also
investigates the real architectural reason Cypress historically struggled with things Selenium and
Playwright handle natively — and gives Cypress a fair, evidence-based comparison against both.

By the end of this level you'll understand exactly where Cypress's in-browser architecture helps and where
it genuinely constrains you, and you'll have built a complete, production-shaped Cypress framework as this
course's capstone.
