# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FancyScrollView is a Unity package providing a highly flexible, performant scroll view component with support for infinite scrolling, custom animations, and grid layouts. The library is primarily written in Japanese with Japanese comments throughout the codebase.

**Unity Version:** 6.0.64f1 (minimum requirement: Unity 6.0+)
**.NET Version:** 4.x Scripting Runtime

## Development Commands

This is a Unity project with a local UPM package. Development is done through the Unity Editor, not command-line tools.

### Opening the Project
- Open Unity Hub and add this project
- Ensure you have Unity 6.0+ installed
- The project will open with the Unity Editor
- The FancyScrollView package is located in `Packages/jp.setchi.fancyscrollview/`

### Running Examples
- Open any scene from `Assets/FancyScrollView/Examples/` (e.g., `01_Basic.unity`, `02_FocusOn.unity`)
- Press the Play button in Unity Editor to run the example
- Examples are numbered 01-09, each demonstrating different features

### Building
- Use Unity's Build Settings (`File > Build Settings`)
- Select target platform and click Build

### Testing
- Unity Test Framework is available (`com.unity.test-framework` v1.1.14)
- Open Test Runner via `Window > General > Test Runner` in Unity Editor

## Architecture

For detailed architecture documentation, see [.claude/rules/architecture.md](.claude/rules/architecture.md).

## File Organization

```
Packages/jp.setchi.fancyscrollview/
├── package.json               # UPM package manifest
├── LICENSE                    # MIT License
├── README.md                  # Package README
├── Documentation~/            # Documentation (UPM standard)
│   ├── development/           # Development guides (branch strategy, modifications)
│   └── upm-structure-reference.md  # UPM structure reference
├── Runtime/
│   ├── Core/                  # Base classes: FancyScrollView, FancyCell
│   ├── Scroller/              # Scroll control: Scroller, easing utilities
│   ├── ScrollRect/            # ScrollRect variant (no infinite/snap)
│   ├── GridView/              # Grid layout variant
│   └── FancyScrollView.asmdef # Runtime assembly definition
└── Editor/
    ├── ScrollerEditor.cs      # Custom inspector for Scroller
    └── FancyScrollView.Editor.asmdef  # Editor assembly definition

Assets/FancyScrollView/
└── Examples/                  # Sample scenes and code
    ├── 01_Basic.unity         # Minimal implementation
    ├── 02_FocusOn.unity       # Using Context for cell selection
    ├── 03_InfiniteScroll.unity # Loop + Unrestricted movement
    ├── 04_Metaball.unity      # Shader-based animation
    ├── 05_Voronoi.unity       # Shader-based animation
    ├── 06_LoopTabBar.unity    # Tab navigation pattern
    ├── 07_ScrollRect.unity    # ScrollRect with scrollbar
    ├── 08_GridView.unity      # Grid layout
    ├── 09_LoadTexture.unity   # Async texture loading
    ├── 01_Basic/              # Example source code
    ├── 02_FocusOn/
    ├── 03_InfiniteScroll/
    └── ... (all example sources)
```

## Common Implementation Pattern

When creating a new scroll view:

1. **Define Data Model** - Plain C# class representing item data
2. **Implement Cell** - Extend `FancyCell<TItemData>` or `FancyCell<TItemData, TContext>`
   - Override `UpdateContent()` to bind data to UI
   - Override `UpdatePosition()` to control animation based on scroll position
3. **Implement ScrollView** - Extend appropriate base class:
   - `FancyScrollView<TItemData>` for custom scroll views
   - `FancyScrollRect<TItemData>` for traditional lists
   - `FancyGridView<TItemData>` for grids
4. **Create Context (optional)** - If cells need to communicate with scroll view
5. **Wire up Scroller** - Connect Scroller component to ScrollView in `Initialize()`

## Important Notes

- **Package Structure**: The package is set up as a local UPM package in `Packages/jp.setchi.fancyscrollview/`
- **Language**: All comments and documentation in source code are in Japanese
- **Assembly Definitions**:
  - Runtime: `FancyScrollView.asmdef`
  - Editor: `FancyScrollView.Editor.asmdef`
- **Examples**: Located in `Assets/FancyScrollView/Examples/` folder
- **API Documentation**: Available at https://setchi.jp/FancyScrollView/api/FancyScrollView.html
- **WebGL Demo**: Available at https://setchi.jp/FancyScrollView/demo

## When Implementing Features

- Refer to examples for implementation patterns (especially `01_Basic` and `02_FocusOn`)
- For infinite scroll: Set `loop = true` on FancyScrollView AND `MovementType.Unrestricted` on Scroller
- For snap behavior: Configure `Scroller` component's snap settings
- ScrollRect and GridView variants do NOT support infinite scrolling or snapping
- Cell position is always normalized (0.0-1.0) - use this for animation calculations

## Development Branch Strategy

This is a fork of [setchi/FancyScrollView](https://github.com/setchi/FancyScrollView). We maintain two branches:

- **`master`**: Syncs with upstream (setchi/FancyScrollView). Keep clean for pulling updates.
- **`develop`**: Main development branch for custom modifications.

**Workflow:**
- Daily work: Use `develop` branch
- Upstream updates: Merge into `master` first, then merge `master` → `develop`
- License: MIT (Copyright (c) 2020 setchi). Original LICENSE file must be preserved.

**Detailed guide:** See [docs/branch-strategy.md](docs/branch-strategy.md) (Japanese)

## Versioning Strategy

This fork uses the `-fork.W` suffix (format: `X.Y.Z-fork.W`) to distinguish from upstream releases.

**Quick reference:**
- Update `package.json` version → commit → create tag → push tag
- GitHub Actions automatically builds and publishes releases

**Detailed rules:** See [.claude/rules/versioning.md](.claude/rules/versioning.md) and [docs/versioning.md](docs/versioning.md)

## CI/CD with GitHub Actions

Automated build and release workflows are configured in `.github/workflows/`.

**Quick reference:**
- PR builds: `.github/workflows/build-samples.yml`
- Releases: `.github/workflows/release.yml` (triggered by `v*.*.*-fork.*` tags)
- Requires: `UNITY_LICENSE`, `UNITY_EMAIL`, `UNITY_PASSWORD` secrets

**Detailed guide:** See [docs/github/workflows/ci-cd.md](docs/github/workflows/ci-cd.md)
