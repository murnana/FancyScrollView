# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FancyScrollView is a Unity package providing a highly flexible, performant scroll view component with support for infinite scrolling, custom animations, and grid layouts. The library is primarily written in Japanese with Japanese comments throughout the codebase.

**Unity Version:** 2021.3.45f2 (minimum requirement: Unity 2019.4+)
**.NET Version:** 4.x Scripting Runtime

## Development Commands

This is a Unity project. Development is done through the Unity Editor, not command-line tools.

### Opening the Project
- Open Unity Hub and add this project
- Ensure you have Unity 2019.4+ installed
- The project will open with the Unity Editor

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

### Core Components

The library is organized into four main modules under `Assets/FancyScrollView/Sources/Runtime/`:

#### 1. Core Module (`Core/`)
The foundation of the scroll view system.

- **`FancyScrollView<TItemData, TContext>`** - Abstract base class for implementing scroll views
  - Supports infinite scrolling via `loop` property
  - Supports snapping to cells
  - Uses cell pooling for performance (only visible cells are instantiated)
  - Key properties:
    - `cellInterval`: spacing between cells (0.01-1.0 range)
    - `scrollOffset`: scroll position reference point (0.0-1.0)
    - `loop`: enables circular cell arrangement for infinite scroll
  - Simplified variant: `FancyScrollView<TItemData>` (without Context)

- **`FancyCell<TItemData, TContext>`** - Abstract base class for implementing cells
  - `UpdateContent(TItemData)`: Update cell display from data
  - `UpdatePosition(float)`: Update cell appearance based on normalized position (0.0-1.0)
  - Simplified variant: `FancyCell<TItemData>` (without Context)

#### 2. Scroller Module (`Scroller/`)
Handles scroll position control and user input.

- **`Scroller`** - Component controlling scroll position
  - Scroll directions: `Horizontal` or `Vertical`
  - Movement types:
    - `Elastic`: bounces at edges
    - `Clamped`: stops at edges
    - `Unrestricted`: no bounds (for infinite scroll)
  - Features: inertia, deceleration, snap-to-cell, scrollbar support
  - Key methods:
    - `ScrollTo(position, duration, easing, onComplete)`: Animate to position
    - `JumpTo(index)`: Instantly jump to index
    - `SetTotalCount(count)`: Set total item count

#### 3. ScrollRect Module (`ScrollRect/`)
Traditional ScrollRect-style scroll views (vertical/horizontal lists).

- **`FancyScrollRect<TItemData, TContext>`** - Base for ScrollRect-style views
  - Does NOT support infinite scrolling or snapping
  - Supports cell reuse with configurable margin (`reuseCellMarginCount`)
  - Supports padding (head/tail) and spacing
  - Must implement `CellSize` property
  - Cells implement `FancyScrollRectCell<TItemData, TContext>`

#### 4. GridView Module (`GridView/`)
Grid layout scroll views.

- **`FancyGridView<TItemData, TContext>`** - Base for grid-style views
  - Extends `FancyScrollRect` (inherits same limitations: no infinite scroll/snap)
  - Uses cell groups for rows/columns
  - Configuration:
    - `startAxisCellCount`: number of cells in perpendicular axis
    - `cellSize`: Vector2 for cell dimensions
    - `startAxisSpacing`: spacing in perpendicular axis
  - Cell groups implement `FancyCellGroup<TItemData, TContext>`

### The Context Pattern

The `Context` object enables communication between cells and scroll view:

- **Shared Instance**: Same Context instance is shared between ScrollView and all Cells
- **Use Cases**:
  - Cell → ScrollView: Pass events (e.g., cell clicked)
  - ScrollView → Cell: Pass shared state (e.g., selected index)
  - See `Assets/FancyScrollView/Examples/Sources/02_FocusOn/Context.cs` for example

```csharp
// Example Context
class Context
{
    public int SelectedIndex = -1;
    public Action<int> OnCellClicked;
}
```

### Key Design Patterns

1. **Cell Pooling**: Only visible cells are created; cells are reused as you scroll
2. **Normalized Positioning**: Cells receive position as 0.0-1.0 value for custom animations
3. **Template Method**: Override `UpdateContent` (data) and `UpdatePosition` (animation) in cells
4. **Generic Types**: `TItemData` for your data model, `TContext` for cell-view communication

## File Organization

```
Assets/FancyScrollView/
├── Sources/
│   ├── Runtime/
│   │   ├── Core/              # Base classes: FancyScrollView, FancyCell
│   │   ├── Scroller/          # Scroll control: Scroller, easing utilities
│   │   ├── ScrollRect/        # ScrollRect variant (no infinite/snap)
│   │   └── GridView/          # Grid layout variant
│   └── Editor/
│       └── ScrollerEditor.cs  # Custom inspector for Scroller
└── Examples/
    ├── 01_Basic/              # Minimal implementation
    ├── 02_FocusOn/            # Using Context for cell selection
    ├── 03_InfiniteScroll/     # Loop + Unrestricted movement
    ├── 04_Metaball/           # Shader-based animation
    ├── 05_Voronoi/            # Shader-based animation
    ├── 06_LoopTabBar/         # Tab navigation pattern
    ├── 07_ScrollRect/         # ScrollRect with scrollbar
    ├── 08_GridView/           # Grid layout
    └── 09_LoadTexture/        # Async texture loading
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

- **Language**: All comments and documentation in source code are in Japanese
- **Assembly Definitions**: Core library uses `FancyScrollView.asmdef`
- **Examples**: Self-contained in `Examples/` folder, safe to modify for learning
- **API Documentation**: Available at https://setchi.jp/FancyScrollView/api/FancyScrollView.html
- **WebGL Demo**: Available at https://setchi.jp/FancyScrollView/demo

## When Implementing Features

- Refer to examples for implementation patterns (especially `01_Basic` and `02_FocusOn`)
- For infinite scroll: Set `loop = true` on FancyScrollView AND `MovementType.Unrestricted` on Scroller
- For snap behavior: Configure `Scroller` component's snap settings
- ScrollRect and GridView variants do NOT support infinite scrolling or snapping
- Cell position is always normalized (0.0-1.0) - use this for animation calculations
