---
name: create-macos-window-shell
description: Create or refactor reusable native macOS window shells for SwiftUI/AppKit apps, including NSWindow or NSPanel initialization, titlebar/full-size-content configuration, traffic-light positioning, unified toolbar chrome, transparent backgrounds, and macOS 26 Liquid Glass or older NSVisualEffectView backdrops. Use when the user asks to initialize/build a macOS window container, create a bare native window, package reusable window chrome, port a visual window style across projects, or add future window-type recipes such as standard app windows, settings windows, floating utility panels, launchers, or inspectors.
---

# Create macOS Window Shell

## Overview

Use this skill to create project-reusable macOS window containers. Keep feature UI separate from window shell code: this skill owns the native window, titlebar/chrome, background material, drag behavior, and traffic lights.

## Workflow

1. Inspect the target project before editing: SwiftUI lifecycle or AppKit delegate, deployment target, Swift language mode, existing window managers, and current titlebar/window modifiers.
2. Choose the closest recipe from `references/`.
3. Centralize all `NSWindow`/`NSPanel` mutation in one owner such as `WindowFactory`, `WindowConfigurator`, `WindowChromeController`, or a window manager.
4. Keep the first implementation bare unless the user explicitly asks for content. Do not include demo drop zones, dashed cards, sample toolbar buttons, or placeholder onboarding UI in the reusable shell.
5. Validate with the project build command and inspect the resulting window for blur, corner radius, shadow, traffic-light position, drag behavior, and hit testing.

## Recipes

- **macOS 26 Liquid Glass standard window**: read `references/liquid-glass-standard-window.md` when the user wants a normal resizable app/document window with native traffic lights, unified rounded chrome, a transparent titlebar, and an empty glass-backed content container.

Add future window types as one reference file per type. Keep this file as the decision and workflow layer.

## Rules

- Prefer native `NSWindow` semantics before custom borderless windows.
- Use `@MainActor` for AppKit window construction and traffic-light positioning in Swift 6+ projects.
- Keep window mutation out of feature views.
- Use system material/backdrop APIs for glass. Avoid handmade gradients unless the user explicitly asks for a branded illustration layer.
- Make backdrop views noninteractive so they never swallow clicks.
- If full-window background dragging conflicts with content, disable `isMovableByWindowBackground` and provide a dedicated drag region.
- For macOS 26, prefer `NSGlassEffectView` for a full-window backdrop when reproducing native window glass. Use SwiftUI `.glassEffect` for smaller component surfaces.

## Validation

- Build the target app.
- Confirm the real system traffic lights remain interactive.
- Confirm the backdrop does not intercept mouse events.
- Confirm the shell has no accidental demo content.
- Search changed Swift files for `LinearGradient`, `RadialGradient`, or hardcoded sample copy if the requested window should be bare.
