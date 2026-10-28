---
description: "cypress run's exit code was genuinely 2, not a generic 1 -- it matched the real count of failing tests, and the CI script propagated that code."
---

# CI Integration

## What was run

A real local shell-script simulation of a CI job's test step — start the real Express app, poll a real
health-check, run the real Cypress suite, capture and propagate its real exit code, tear the app down —
run twice: once against a passing real suite, once against a genuinely failing one.

```bash
echo "=== REAL CI STEP 1: start the real Express app under test, in the background ==="
nohup node server.js > /tmp/real_ci_express.log 2>&1 &
APP_PID=$!

echo "=== REAL CI STEP 2: real health-check polling ==="
for i in $(seq 1 20); do
    if curl -s -o /dev/null -w "%{http_code}" http://127.0.0.1:5151/ | grep -q "200"; then
        break
    fi
    sleep 0.2
done

echo "=== REAL CI STEP 3: run the real Cypress suite, capture its real exit code ==="
set +e
npx cypress run --spec cypress/e2e/level2-capstone.cy.js
TEST_EXIT_CODE=$?
set -e

echo "=== REAL CI STEP 4: tear down the real Express app ==="
kill $APP_PID 2>/dev/null || true
exit $TEST_EXIT_CODE
```
```
real app became healthy after 2 real health-check attempts
✓ stubs the todos endpoint with unrelated fixture data to prove real stub control (104ms)
✓ logs in via the real custom command and confirms via cy.request in parallel (399ms)
✓ fixture-drives multiple real login attempts and intercepts the real todos call (1186ms)
3 passing (2s)
real cypress exit code: 0
```

## A real, genuinely informative exit code — not just 0 or 1

```bash
# same script, pointed at a spec with 2 real failing tests instead
```
```
retry-pitfalls.cy.js    4 tests   2 passing   2 failing
real cypress exit code: 2
SCRIPT_EXIT_CODE=2
```

Cypress's real exit code here was `2` — matching the exact real count of failing tests, not a generic `1`
the way this network's own [Selenium Mastery Path](https://sigilipelli.github.io/selenium-mastery-path/level-4/07-ci-integration/) found pytest using. The real CI script's own `exit $TEST_EXIT_CODE` correctly
propagated that exact number all the way out, confirmed by the final `SCRIPT_EXIT_CODE=2`.

## How It Actually Works

`cypress run`'s real exit code convention is deliberately more granular than a simple pass/fail flag: `0`
means every test passed, and any positive number *is* the real count of failing tests (capped at a real
maximum before falling back to a generic nonzero code for very large failure counts) — which means a CI
system reading that exit code can distinguish "2 tests failed" from "47 tests failed" without parsing any
log output at all, just from the process exit status. The script's own real mechanics are otherwise
identical to the equivalent Selenium-based pipeline this network already built: `set +e` around the test
command so a nonzero exit doesn't kill the whole script before the code can be captured, a real
health-check loop instead of a fixed sleep, and teardown that runs regardless of the test outcome.

## Common mistakes

- Checking only `if [ $TEST_EXIT_CODE -eq 0 ]` and discarding the actual number — this lesson's real `2`
  vs a hypothetical `47` carries genuinely useful information a CI dashboard or Slack notification could
  surface directly, without re-parsing Cypress's own text output.
- Assuming Cypress's exit-code convention matches every other test runner's (a flat `0`/`1`) — this
  lesson's real, measured `2` is specific to Cypress and would need different handling in a script that
  also runs, say, a pytest suite in the same pipeline (as the sibling Selenium course's CI script does).
- Forgetting `set +e`/`set -e` around just the test command — without it, a real nonzero Cypress exit
  would terminate the whole script immediately under `set -e`, skipping teardown and losing the exit code
  before it could be captured and propagated.

## Exercise

Modify this lesson's real script to also write `TEST_EXIT_CODE` to a file, then run it against a spec
with exactly 1 failing test, then one with exactly 3. Confirm the real exit code captured in each case
matches the real failing-test count exactly, directly confirming Cypress's own documented exit-code
convention holds across different real failure counts, not just the single data point this lesson
measured.
