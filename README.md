# Visual JSON Editor

[![CI](https://github.com/slaugaus/visual-json-editor-vscode/actions/workflows/ci.yml/badge.svg)](https://github.com/slaugaus/visual-json-editor-vscode/actions/workflows/ci.yml)

Open JSON files in a GUI that looks (kind of) like the VS Code settings page! Includes type changing, item rearrangement, undo/redo, and assistance with colors, dates, and times.

## Features

### Edit Names & Values

Names are validated:

![](.readme/rename.gif)

Strings have multi-line support:

![](.readme/string.gif)

Numbers are validated and aren't constrained by JS limits:

![](.readme/number.gif)

Booleans become checkboxes:

![](.readme/bool.gif)

Full support for object and array nesting:

![](.readme/obj.gif)

6-digit hex codes give you the Chromium color picker:

![](.readme/color.png)

Finally, ISO 8601 zoneless date-time strings give you the Chromium `datetime-local` picker:

![](.readme/datetime.png)

### Type Conversion

Convert certain types of item to other types using the dropdown box.

![](.readme/type.gif)

If something's "stuck" as a certain type, use the clear button to nullify it and try again.

![](.readme/clear.gif)

### Rearrange Items

Use the up and down arrows to move an item around the "layer" it's in.

![](.readme/move.gif)

### Theme Compatible

All colors are pulled from your current color theme.

![](.readme/theme-collage.png)

## Extension Settings

This extension contributes the following settings:

* `visual-json.outputPrettiness`: Number of space characters to indent saved JSON files by, or 0 to save in one line. Defaults to 2 spaces. (This gets passed as the [`space` parameter of `JSON.stringify()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify#space).)
* `visual-json.maxFileSize`: Maximum file size (in kilobytes) the editor will open before displaying an error message. Defaults to 256 KB.

## Building & Running from Source

### Prerequisites

This project builds with [Bun](https://bun.sh). The toolchain is pinned in `mise.toml`, so if you use [mise](https://mise.jdx.dev) you can just run `mise install` to get the right Bun and Node versions. Otherwise, install Bun 1.3.6 (or newer) manually.

Install dependencies once:

```sh
bun install
```

### Run in a development window (F5)

1. Open this folder in VS Code.
2. Press <kbd>F5</kbd> (or Run → "Run Extension").

This builds the extension (`bun run compile`) and opens a second VS Code window — the Extension Development Host — with the extension loaded. Open any `.json` file there to use the editor.

While developing:
* Edited a `.ts` file? Press <kbd>F5</kbd> again to rebuild and relaunch.
* Edited only CSS (in `media/`)? It's served as-is — just reload the dev window with <kbd>Ctrl</kbd>+<kbd>R</kbd> (<kbd>Cmd</kbd>+<kbd>R</kbd> on macOS).

### Install from source for ongoing use

To use your local build as a regular installed extension (not just in the dev window), package it into a `.vsix` and install that:

```sh
bunx vsce package        # produces visual-json-<version>.vsix (runs a production build first)
code --install-extension visual-json-*.vsix
```

Alternatively, install the `.vsix` from the UI: Extensions view → `...` menu → "Install from VSIX…".

To update later, re-run the two commands above; to uninstall, remove it from the Extensions view like any other extension.

## Known Issues

* The same file cannot be open in multiple instances of the editor. This was done intentionally to avoid the complexity of syncing state between them.
  * The editor doesn't sync its state with text editors that have the same file open, either. Not sure if that's even possible.
* Tab-key navigation has not yet been implemented - sorry, keyboard warriors, you'll need your mouse for this.

## Version History

### 1.0.0

* Initial release
