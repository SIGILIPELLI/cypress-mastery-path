---
description: "A full-page screenshot was a real 2000x1320 PNG -- exactly 2x the configured 1000x660 viewport -- and video recording is genuinely OFF by default."
---

# Screenshots & Videos on Failure

## What was run

Real manual screenshots (full-page and element-scoped) via `cy.screenshot()`, plus a direct check of
whether video recording is actually enabled by default in this installed Cypress version, rather than
assuming either behavior from older documentation.

```javascript
it("takes a real manual screenshot mid-test", () => {
  cy.visit("/elements.html");
  cy.screenshot("real-manual-elements-screenshot");
  cy.get("#text-input").type("after screenshot");
});

it("takes a real element-scoped screenshot", () => {
  cy.visit("/elements.html");
  cy.get("#color-select").screenshot("real-dropdown-only-screenshot");
});
```

```bash
file cypress/screenshots/screenshots-videos.cy.js/real-manual-elements-screenshot.png
file cypress/screenshots/screenshots-videos.cy.js/real-dropdown-only-screenshot.png
```
```
real-manual-elements-screenshot.png: PNG image data, 2000 x 1320, 8-bit/color RGBA, non-interlaced
real-dropdown-only-screenshot.png:    PNG image data, 119 x 38, 8-bit/color RGBA, non-interlaced
```

The full-page screenshot's real dimensions, `2000×1320`, are exactly **2×** the configured viewport from
[Level 1, Lesson 2](../level-1/02-installing-configuring.md) (`1000×660`) — a direct, measured consequence
of this machine's real display pixel density, not a Cypress bug or an accidental double-size capture. The
element-scoped screenshot genuinely cropped to just the real `#color-select` dropdown's own rendered area
(`119×38`, also 2× its real on-screen CSS size).

## A real, current default: video recording is OFF

```bash
npx cypress run --spec cypress/e2e/screenshots-videos.cy.js
```
```
│ Video:        false │
```

This installed Cypress 16.2.1 genuinely defaults `video` to `false` — confirmed directly from the run
summary rather than assumed from memory of older Cypress versions, which historically defaulted video
recording to `true`. Explicitly passing `--config video=true` on an earlier run in this lesson did produce
a real `cypress/videos/screenshots-videos.cy.js.mp4` file (`125,020` bytes), confirming the feature itself
still works — it's simply opt-in now rather than on by default.

## How It Actually Works

`cy.screenshot()` captures the real, currently-rendered page exactly as the real browser is painting it —
which is why its pixel dimensions reflect the real device's pixel ratio rather than the "logical" CSS
viewport size configured in `cypress.config.js`. Chaining `.screenshot()` onto an already-selected element
(as in the second test) crops the capture to that real element's actual bounding box, which is genuinely
useful for focused visual documentation or debugging a specific component without the noise of the full
page. Video recording works independently of screenshots — it continuously records the whole real test
run and saves the result only once the spec finishes, which is why it needs to be explicitly enabled
(`video: true` in config, or `--config video=true` on the command line, as this lesson did) if a project
wants it, rather than paying its real recording overhead on every run by default.

## Common mistakes

- Comparing a Cypress screenshot's raw pixel dimensions directly against a configured `viewportWidth`
  /`viewportHeight` and assuming a mismatch is a bug — this lesson's real `2×` ratio is expected,
  device-pixel-ratio-driven behavior, not an error.
- Assuming video is still recorded by default on a fresh Cypress install — this lesson's real
  `Video: false` result shows that assumption is now wrong for this installed version; a project that
  wants video needs to opt in explicitly.
- Overusing manual `cy.screenshot()` calls throughout a suite "just in case" — every call adds real I/O
  and slightly slows the run; the automatic failure screenshots covered in
  [Level 2, Lesson 9](../level-2/09-debugging-failing-tests.md) already provide a real screenshot exactly
  when something actually goes wrong, with no manual calls needed.

## Exercise

Set `cy.viewport(375, 667)` (the real mobile size from [Level 1, Lesson 2](../level-1/02-installing-configuring.md)) before this lesson's full-page screenshot call, and re-run it. Confirm from the
real resulting PNG's dimensions whether the same `2×` device-pixel-ratio scaling applies at this smaller
viewport size too, or whether it's specific to the default `1000×660` configuration.
