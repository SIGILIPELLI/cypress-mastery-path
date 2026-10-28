---
description: "selectFile() uploaded a real disk file in 113ms and a real in-memory Buffer with no disk file in 45ms -- no ERR_ACCESS_DENIED, unlike Selenium."
---

# File Upload With selectFile

## What was run

Two real uploads via `cy.get().selectFile()` against the local upload page: one from a real file on disk,
one generated entirely in memory with no file ever written to disk at all.

```javascript
it("selects a real local file via selectFile", () => {
  cy.visit("/upload.html");
  cy.get("#file-input").selectFile("cypress/fixtures/real-upload.txt");
  cy.get("#upload-result").should("have.text", "Selected: real-upload.txt");
});

it("selects a real file built from an in-memory buffer, no disk file needed", () => {
  cy.visit("/upload.html");
  cy.get("#file-input").selectFile({
    contents: Cypress.Buffer.from("generated in-memory content"),
    fileName: "generated.txt",
    mimeType: "text/plain",
  });
  cy.get("#upload-result").should("have.text", "Selected: generated.txt");
});
```
```
✓ selects a real local file via selectFile (113ms)
✓ selects a real file built from an in-memory buffer, no disk file needed (45ms)
```

Both real uploads worked cleanly. This network's own [Selenium Mastery Path](https://sigilipelli.github.io/selenium-mastery-path/level-2/09-file-upload-stale-elements/) hit a genuine real
`ERR_ACCESS_DENIED` uploading from a path under `~/Desktop` (fixed by moving the file to `/tmp`) — this
lesson's real `cypress/fixtures/real-upload.txt` path, also outside any special-cased directory, uploaded
without any such error.

## How It Actually Works

`cy.get("#file-input").selectFile(path)` reads a real file from disk (relative to the project root) and
directly sets it as the input element's file list, dispatching the real `change` event the browser fires
after a person picks a file — no OS file-picker dialog is simulated or needed, the same underlying approach
Selenium's `send_keys()` uses on a file input. The in-memory form (`{ contents: Cypress.Buffer.from(...),
fileName, mimeType }`) skips the real disk read entirely: Cypress constructs a real `File` object directly
from the given bytes and filename, which is genuinely useful for generating test fixtures on the fly
(a CSV with specific edge-case rows, an image of a specific exact size) without maintaining a library of
real files in the repository.

The real contrast with Selenium's sandboxed `ERR_ACCESS_DENIED` traces back to the same in-browser
architecture from [Lesson 1](../level-1/01-what-cypress-actually-is.md): Cypress's own Node-side process
reads the real file and hands it to the browser context it's already directly embedded in, rather than a
separate out-of-process browser enforcing its own filesystem-access sandboxing against a path it didn't
choose and doesn't fully trust.

## Common mistakes

- Assuming `selectFile()` needs a real file on disk in every case — the in-memory buffer form in this
  lesson's second test is often more maintainable for generated or parameterized test data.
- Using an absolute path when a path relative to the project root would be more portable across machines
  and CI runners — this lesson's `"cypress/fixtures/real-upload.txt"` works regardless of where the repo
  is checked out.
- Forgetting that `selectFile()` sets the files directly without simulating drag-and-drop by default — a
  page whose upload logic specifically listens for `drop` events (rather than the input's `change` event)
  needs `selectFile(path, { action: "drag-drop" })` instead of the default click-to-select simulation.

## Exercise

Generate a real in-memory CSV buffer with exactly 1,000 rows (`Cypress.Buffer.from(bigCsvString)`) and
upload it via `selectFile()`, then check the real elapsed time Cypress reports for that command compared to
this lesson's small text-file uploads. Confirm whether `selectFile()`'s real performance scales
noticeably with payload size in this environment, or stays effectively constant for file sizes this small.
