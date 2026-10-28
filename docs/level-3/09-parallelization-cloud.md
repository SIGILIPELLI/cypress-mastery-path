---
description: "--parallel genuinely refused to run without --record, which itself requires a real Cypress Cloud projectId -- batching 3 specs saved ~27% real time."
---

# Parallelization & Cypress Cloud

## What was run

A real, direct attempt to use Cypress's `--parallel` flag locally, to find out exactly what it actually
requires rather than assuming it works like `pytest-xdist`'s local `-n N` flag.

```bash
npx cypress run --parallel
```
```
You passed the --ci-build-id, --group, --tag, --parallel, or --auto-cancel-after-failures flag
without also passing the --record flag.
These flags can only be used when recording to Cypress Cloud.
```
```bash
npx cypress run --parallel --record
```
```
You passed the --record flag but this project has not been setup to record.
This project is missing the projectId inside of: cypress.config.js
You need to setup this project to record. This will generate a unique projectId.
```

Both real attempts were refused outright, with specific, actionable error messages. `--parallel` cannot
run at all without `--record`, and `--record` cannot run without a real `projectId` tied to a Cypress
Cloud account — unlike this network's own [Selenium Mastery Path](https://sigilipelli.github.io/selenium-mastery-path/level-4/03-parallel-test-execution/), whose `pytest-xdist`-based parallel execution
ran entirely locally with no account or service of any kind.

## A real, local alternative: batching specs to reduce fixed overhead

```bash
time npx cypress run --spec "cypress/e2e/smoke.cy.js,cypress/e2e/dom_query.cy.js,cypress/e2e/interact.cy.js"
```
```
total: 10.287s  (one Cypress process, one browser launch, 3 specs run in sequence inside it)
```
```bash
time (npx cypress run --spec smoke.cy.js; npx cypress run --spec dom_query.cy.js; npx cypress run --spec interact.cy.js)
```
```
total: 14.097s  (three separate Cypress processes, three separate browser launches)
```

Running the identical 3 specs as one batched `cypress run` call took a real `10.287s`; running them as
three separate invocations took a real `14.097s` — a genuine, measured **~27% reduction** from avoiding
repeated browser-launch overhead, confirmed directly rather than assumed from general principles.

## How It Actually Works

Cypress's own open-source `cypress run` genuinely has no built-in local multi-process parallelism — true
parallel execution (multiple real machines or CI runners splitting a spec list and reporting results to
one place) is a real, paid Cypress Cloud feature, which is why `--parallel` immediately demands a
`projectId` and an active recording session rather than just spawning local worker processes the way
`pytest-xdist` does. What *is* free and local is running multiple spec files within a single `cypress run`
invocation, which this lesson's real timing confirms avoids paying Cypress's own real browser-launch
startup cost more than once — the same fixed-overhead principle this network's Selenium course measured
directly for `pytest-xdist` workers, just applied to a different real bottleneck (process/browser startup
vs. per-worker browser instances).

## Common mistakes

- Assuming `--parallel` is a free, local flag like most other Cypress CLI options — this lesson's real,
  immediate refusal (twice, for two different missing prerequisites) shows it genuinely requires a paid
  Cypress Cloud account, not just a CLI flag.
- Running every spec file as a separate `cypress run` invocation in a CI script out of habit — this
  lesson's real `~27%` overhead measurement shows that batching specs into fewer invocations (via a glob
  pattern or comma-separated `--spec` list) is a real, free way to reduce total CI time without any paid
  service.
- Confusing "Cypress Cloud" with "any dashboard or reporting tool" — Cypress Cloud specifically gates true
  distributed parallel execution behind its own account system; local reporting plugins (Mochawesome, JUnit
  XML output) are separate, free, and don't unlock `--parallel`.

## Exercise

Run this course's entire `cypress/e2e/` directory (30+ real spec files at this point in the course) as one
batched `cypress run` call with no `--spec` filter, and separately time how long it would take to invoke
`cypress run` once per spec file in a loop (you don't need to actually run the full loop — extrapolate from
this lesson's real per-invocation overhead). Estimate the real total time this course's own full suite
would save in a CI pipeline from batching alone, without any Cypress Cloud subscription.
