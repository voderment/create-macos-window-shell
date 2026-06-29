# create-macos-window-shell

A Codex skill for creating reusable native macOS window shells.

The skill packages repeatable patterns for initializing macOS `NSWindow`/`NSPanel`
containers across SwiftUI, AppKit, and Electron-style projects. It keeps window
chrome separate from feature UI: the skill owns the native titlebar, traffic
lights, full-size content behavior, glass backdrop, drag behavior, and validation
checks.

## Current Recipe

The first included recipe is:

**macOS 26 Liquid Glass standard window**

- Standard resizable `NSWindow`, not borderless.
- Native close/minimize/zoom traffic lights.
- Transparent full-size titlebar.
- Empty unified `NSToolbar` for native rounded unified chrome.
- Clear, non-opaque window background.
- macOS 26 `NSGlassEffectView` full-window backdrop.
- `NSVisualEffectView` fallback for older macOS versions.
- Bare content area by default: no demo drop zone, dashed card, or placeholder UI.

Detailed recipe:

```text
references/liquid-glass-standard-window.md
```

## Installation

Clone directly into your Codex skills directory:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
git clone https://github.com/voderment/create-macos-window-shell.git \
  "${CODEX_HOME:-$HOME/.codex}/skills/create-macos-window-shell"
```

Or keep the repository somewhere else and symlink it into Codex:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
ln -sfn /path/to/create-macos-window-shell \
  "${CODEX_HOME:-$HOME/.codex}/skills/create-macos-window-shell"
```

## Usage

Ask Codex to use the skill when creating or refactoring a native macOS window
container:

```text
Use $create-macos-window-shell to create a bare macOS 26 Liquid Glass window shell
for this SwiftUI app.
```

Other useful prompts:

```text
Use $create-macos-window-shell to extract this AppKit window setup into a reusable
WindowFactory and WindowChromeController.
```

```text
Use $create-macos-window-shell to adapt this Electron BrowserWindow to native
macOS Liquid Glass chrome.
```

## Repository Structure

```text
.
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    └── liquid-glass-standard-window.md
```

- `SKILL.md` is the trigger and workflow layer.
- `agents/openai.yaml` contains UI metadata for Codex.
- `references/` contains detailed window recipes. Add one reference file per
  new window type.

## Design Principles

- Prefer native `NSWindow` semantics before custom borderless windows.
- Centralize all window mutation in one owner.
- Keep feature views out of window chrome decisions.
- Use system material/backdrop APIs for glass.
- Avoid handmade gradients for the base shell unless a product specifically
  needs branded lighting.
- Make backdrop views noninteractive so glass never swallows clicks.
- Start with a bare shell. Add feature UI only when the caller asks for it.

## Roadmap

Planned future recipes:

- Native settings window.
- Floating utility panel.
- Inspector window.
- Launcher or command palette panel.
- Electron native glass shell.

## Reference

The first recipe was informed by the native window-backdrop approach in
[AlexandrosGounis/pdfx](https://github.com/AlexandrosGounis/pdfx), especially
the use of an empty unified toolbar plus a pass-through native glass backdrop.
