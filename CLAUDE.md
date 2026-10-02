# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A VS Code extension (`visual-json`) that registers a **custom editor** for `*.json` files, rendering them in a settings-page-like GUI instead of as text. Publisher `slaugaus`, entry point `./dist/extension.js`.

## Commands

The package manager is **Bun** (`bun.lock`; all scripts and build tooling run through `bun`, including the `.ts` scripts in `scripts/`).

- `bun install` — install deps
- `bun compile` — type-check (`tsc --noEmit`), lint, then bundle via `scripts/build.ts`
- `bun compile:prod` — same, but minified + no sourcemaps (used by `vscode:prepublish`)
- `bun check-types` — `tsc --noEmit` only
- `bun lint` — `eslint src`
- `bun run ci` — stamp a dev version (`scripts/ci-version.ts`) then `vsce package` into a `.vsix`
- **Tests:** `bun run compile:tests` (emits test JS to `out/` via `tsc -p .`) then `bun run test` (invokes `vscode-test`, which launches a headless VS Code against `out/test/**/*.test.js`). Use `bun run test`, not `bun test` — the latter runs Bun's own test runner instead of the npm script.
- **Run/debug the extension:** press F5 in VS Code (launches the Extension Development Host). There is no CLI run target.

## Architecture

The extension has **two separately-bundled targets**, both produced by `scripts/build.ts` into `./dist`:

1. **Extension host** (`src/extension/`, Node/CommonJS) — runs in VS Code's Node process.
2. **Editor webview** (`src/editor/`, browser/IIFE) — runs in the webview iframe. `build.ts` also copies codicon CSS/font into `dist`.

`src/common.ts` holds the types shared by both sides (message shapes, `JsonEdit`, type tables). Each target has its own `tsconfig.json` with different `lib`/`module`/`target` (Node16+ES2022 for the extension, ES6+DOM for the editor); the root `tsconfig.json` only references them as project refs (used for test compilation).

### The key design decision: document state lives in the DOM, not a model

There is **no in-memory object model** of the edited document. The webview's `#jsonContainer` DOM *is* the document while editing. Consequences that ripple through the codebase:

- **Saving** (`JsonDocument.saveAs`) asks the webview for its `#jsonContainer.innerHTML` via a request/response message (`getData` → `responseReady`), then reconstructs a JS object from that HTML in `JsonDocument._readHtml`/`_addFromNode` using `node-html-parser`. Element type is read from CSS classes (`_getTypeOfElement`).
- `webviewOptions.retainContextWhenHidden: true` is required — losing the webview would lose the document.
- `supportsMultipleEditorsPerDocument: false` — the same file can't be open in two editor instances (no state sync). `getData` throws if it finds ≠1 webview for a doc.
- Save **fails safe**: if the parsed result is an empty object/array when the HTML was non-empty, the save is cancelled rather than writing data loss.

### Extension ↔ webview messaging

`JsonEditorProvider` is the `CustomEditorProvider`. It communicates with the webview exclusively through `postMessage` using `Message<T>` from `common.ts`. Notable flows:

- Webview sends `ready` on load → extension replies with `doc` (the parsed object) to populate the editor.
- Webview sends `edit` (a `JsonEdit`) on every user change → `JsonDocument.makeEdit` records it and fires `onDidChange` so VS Code shows the dirty indicator.
- **Undo/redo/revert is edit replay, not state diffing:** `JsonDocument` keeps a `_freshEdits` list. Undo/redo pop/push that list and fire `onDidChangeContent` with the edit list; the webview (`editor.ts` `change` handler) re-parses the *starting* object and replays edits via `Helpers.playbackEdits` (with `Helpers.ignoreEdits` set so replay doesn't echo new edits back).
- `_sendMessageWithResponse` implements request/response over the one-way channel via a `_requestId` + `_callbacks` map.

### Webview internals (`src/editor/`)

- `editor.ts` — message handler + entry point (sets up `#rootPlus` "New Item" button).
- `EditorItem.ts` — one JSON key/value pair, rendered as a `<details>` element. Owns name, type dropdown, and value.
- `EditorValue.ts` — abstract factory + subclasses per type (`EditorString`, `EditorNumber`, `EditorBool`, `EditorCollection`, `EditorColor`, `EditorDateTime`). Collections recurse by holding child `EditorItem`s.
- `Helpers.ts` — static utilities: the `jsonContainer` ref, codicon icon map, the `validConversions` table governing which type→type conversions are allowed, edit sending/playback, object parsing.
- `vscode-webview.ts` — typed wrapper around the webview's `acquireVsCodeApi()`.

### Numbers

`lossless-json` (not `JSON.parse`/`stringify`) is used on both sides so large/precise numbers aren't clamped to JS float limits. Numbers round-trip as `LosslessNumber`.

## Types & conventions

- `common.ts` is the source of truth for the type system: `editorTypes`, `editorSubTypes` (maps special UI types like `color`/`datetime` to their base `string`), `JsonEdit`/`JsonEditType`, `EditAddition`, `OutputHTML`, `Message`.
- When adding a new value type: extend `editorTypes`/`editorSubTypes` in `common.ts`, add a subclass in `EditorValue.ts`, handle it in `JsonDocument._addFromNode` (HTML→object), add an entry to `Helpers.validConversions` and `Helpers.codiconMap`.

## Settings

`visual-json.outputPrettiness` (indent spaces for saved JSON, 0 = single line; passed as `JSON.stringify` `space`) and `visual-json.maxFileSize` (KB cap before the editor refuses to open and offers the text editor fallback).

## Scratch / planning

`DA-PLAN.md` is the author's informal roadmap/bug list — useful for intent, not a spec. Known limitations (no multi-editor sync, no keyboard nav) are listed there and in the README.
