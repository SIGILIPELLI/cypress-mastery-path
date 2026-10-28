---
description: "Cypress.env() has genuinely been removed in this installed 16.2.1 -- cy.env() also now requires an array of keys, discovered by reading the real error."
---

# Environment Variables & Configuration

## What was run

A real attempt to read a config-level environment variable, which immediately surfaced a genuine, current
API change in this installed Cypress 16.2.1 — investigated directly rather than assumed to still work the
way older Cypress documentation (or training knowledge) might suggest.

```javascript
// cypress.config.js
module.exports = defineConfig({
  e2e: { baseUrl: "http://localhost:5151", supportFile: false },
  env: { configEnvVar: "from-config-file" },
});
```

```javascript
// first attempt -- the "classic" API
expect(Cypress.env("configEnvVar")).to.eq("from-config-file");
```
```
CypressError: `Cypress.env()` was removed in Cypress version 16.0.0. Please update to use
`Cypress.expose()` for non-sensitive values, or `cy.env()` for sensitive values.
```

## Two real, genuinely surprising API changes, found by actually running the code

```javascript
// second attempt -- cy.env() with a bare string key, as older docs describe
cy.env("configEnvVar").should("eq", "from-config-file");
```
```
CypressError: `cy.env()` must be passed a non-empty list of strings. You passed: 'configEnvVar'.
```

```javascript
// third attempt -- the real, current signature: an array of keys
cy.env(["configEnvVar"]).then((vals) => {
  expect(vals.configEnvVar).to.eq("from-config-file");
});
```
```
✓ reads a real env value set in cypress.config.js via cy.env([list]) (22ms)
```

Two real breaking changes stacked on top of each other in this installed version: `Cypress.env()` is gone
entirely, and `cy.env()` itself now requires an array of key names (returning an object keyed by those
names), not a single bare string returning a single value the way older Cypress versions' documentation
describes.

## Real, confirmed CLI-over-config precedence

```bash
npx cypress run --spec cypress/e2e/env-config.cy.js --env configEnvVar=from-cli-flag
```
```
AssertionError: expected 'from-cli-flag' to equal 'from-config-file'
```

This test was still hardcoded to expect the config-file value — the real failure here is the *proof*: the
`--env` CLI flag's value (`"from-cli-flag"`) genuinely won over `cypress.config.js`'s `"from-config-file"`,
confirming the real precedence order directly rather than trusting documentation about it.

## How It Actually Works

Cypress historically exposed environment-style configuration through `Cypress.env()` (synchronous, browser
side) and `cy.env()` (as a queued command); this installed version has consolidated that into `cy.env()`
alone, now explicitly keyed-and-batched (an array of names in, an object of values out) rather than one
value per call — a real, deliberate API tightening, not an accidental breakage, going by the migration
pointer in the error message itself (`Cypress.expose()` for values the browser-side app needs directly,
`cy.env()` for values the test itself needs). Configuration values can come from multiple real sources —
`cypress.config.js`'s `env` block, a `cypress.env.json` file, or `--env key=value` on the command line — and
this lesson's real CLI-override test confirms the command line wins when multiple sources define the same
key, which matters for CI pipelines that need to override a value (a target URL, a feature flag) without
editing the checked-in config file.

## Common mistakes

- Copying `Cypress.env("key")` snippets from older tutorials or AI-generated code without checking against
  the actually-installed Cypress version — this lesson's own real first attempt did exactly that and hit a
  real, informative removal error instead of a confusing silent failure.
- Passing a bare string to `cy.env()` expecting a single value back, based on older API familiarity — this
  installed version's real signature needs an array and returns an object, confirmed by this lesson's
  second real attempt.
- Assuming env values are a `.should()`-style retrying assertion target the same way DOM queries are —
  `cy.env()` reads a fixed configuration value set before the run started; it has nothing to retry against
  and will never change mid-test the way a DOM element's state might.

## Exercise

Create a real `cypress.env.json` file at the project root with `{"configEnvVar": "from-json-file"}`, leave
`cypress.config.js`'s `env` block as-is, and re-run this lesson's test with no CLI `--env` flag. Determine,
from the real observed result, where `cypress.env.json` sits in Cypress's real precedence order relative to
`cypress.config.js`'s `env` block — directly extending this lesson's own confirmed CLI-wins-over-config
finding to a third real configuration source.
