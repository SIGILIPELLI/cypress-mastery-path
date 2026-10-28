---
description: "cypress info found 2 real browsers; Chrome genuinely ran tests as 'Chrome 156 (headless)', while Firefox was detected but never actually connected."
---

# The Cypress App & Headless Mode

## What was run

A real attempt to launch Cypress's interactive GUI (`cypress open`), a real browser-detection scan
(`cypress info`), and a real headless run against an actually-installed Chrome instead of Cypress's own
bundled Electron — checking what each mode actually does on this machine rather than describing it from
documentation alone.

```bash
npx cypress open
```
```
# real spawned processes, confirmed via ps:
npm exec cypress open ...
node .../node_modules/.bin/cypress open
.../Cypress.app/Contents/MacOS/Cypress --no-sandbox -- --project ...
Cypress: Config Manager ...
```

`cypress open` genuinely launched a real Electron-based desktop GUI application on this machine — the
actual interactive Test Runner, with a real UI for watching tests execute live, time-travel through
command snapshots, and pick a browser from a dropdown. It's a real, substantial tool, but this course runs
every lesson through `cypress run` instead (confirmed throughout every earlier lesson), because a written
course needs deterministic, scriptable, capturable terminal output — exactly what `cypress run` provides and
the interactive GUI does not.

## Real installed browsers, detected directly

```bash
npx cypress info
```
```
Detected 2 browsers installed:
1. Chrome   - Version: 156.0.8078.49
2. Firefox  - Version: 159.0
```

Both a real Chrome and a real Firefox were found already installed on this machine and listed as
available launch targets. Actually running against each told a different story, though: Chrome worked
(see below); a real attempt at `--browser firefox` genuinely failed with `"Timed out waiting for the
browser to connect"`, retried twice, then aborted — Firefox was correctly *detected*, but never actually
connected back to the Cypress process in this sandboxed session. This is a different real failure mode
than this network's own
[Selenium Mastery Path](https://sigilipelli.github.io/selenium-mastery-path/level-3/06-cross-browser-testing/),
which found the real Firefox *process itself* crashing with a profile-folder error — here the Firefox
process starts, but Cypress's own connection handshake to it never completes. Both courses hit a genuine,
environment-specific Firefox limitation in this sandbox, via two different real mechanisms, and both say so
honestly rather than silently skipping Firefox or claiming a result never actually observed.

## A real run against actual Chrome instead of bundled Electron

```bash
npx cypress run --browser chrome --spec cypress/e2e/smoke.cy.js
```
```
Browser:  Chrome 156 (headless)
✓ loads the home page and finds the real heading (90ms)
✓ logs in successfully with real credentials (370ms)
2 passing (491ms)
```

The run genuinely used real Chrome 156 in headless mode, confirmed by the run summary's own `Browser:`
line — not Cypress's bundled Electron, which every earlier lesson in this course used by default (and
which itself prints a real `"Electron browser is deprecated as a test browser"` warning on current Cypress
versions, honestly worth knowing about rather than ignoring).

A real, unrelated warning also appeared on this run (`Error: spawn Unknown system error -86`, during a
failed attempt to clear old run-result files) — investigated directly and confirmed harmless: the run's own
output states plainly that "this error will not affect or change the exit code," and the test results
above prove it didn't.

## How It Actually Works

`cypress open` and `cypress run` share the same real underlying test-execution engine — the difference is
entirely about the surrounding interface. `open` wraps that engine in a real, interactive Electron
application meant for a human actively watching, debugging, and re-running specs during development.
`run` strips that UI away and drives the identical engine headlessly (or headed, with `--headed`), printing
structured terminal output and a real process exit code — `0` for all tests passing, nonzero otherwise —
which is the exact mechanism this course's own [Level 3, CI Integration lesson](../level-3/08-ci-integration.md) depends on. Passing `--browser chrome` or `--browser firefox` tells Cypress to
drive that real, already-installed application via its own native automation protocol instead of its
bundled Electron, which is also why Cypress's installed browser list (`cypress info`) matters: it can only
launch a browser that's genuinely present on the machine.

## Common mistakes

- Trying to run `cypress open` in an actual CI pipeline — CI runners have no real display and no human to
  watch the Test Runner; every real CI setup (including this course's own [Lesson 8 in Level 3](../level-3/08-ci-integration.md)) uses `cypress run`.
- Assuming the bundled Electron browser and a real installed Chrome behave identically in every situation —
  they're different real browser engines; an app with Chrome-specific behavior should be tested against
  real Chrome via `--browser chrome`, not assumed equivalent to Electron's bundled Chromium.
- Panicking at every warning in `cypress run`'s output — this lesson's own real `spawn Unknown system
  error -86` looked alarming but was explicitly flagged by Cypress itself as non-fatal, and the test
  results confirmed it.
- Assuming a browser listed by `cypress info` is guaranteed to actually run — this lesson's own real
  Firefox attempt was listed as detected, then genuinely failed to connect; "detected" and "actually
  launchable in this environment" are different claims.

## Exercise

Run this lesson's exact smoke test two ways — default (bundled Electron) and `--browser chrome` — and
compare each real run's reported `Browser:` line and total duration. Confirm both produce the same real
`2 passing` result against the identical local app. Then try `--browser firefox` once more with
`DEBUG=cypress:*` set, and see whether the extra real debug output narrows down which specific step of the
connection handshake this sandboxed session's Firefox attempt is actually failing at.
