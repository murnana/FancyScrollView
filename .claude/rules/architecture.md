---
paths: **/*.cs, docs/architecture.md
---

# Architecture

## Core Components

The library is organized into four main modules under `Packages/jp.setchi.fancyscrollview/Runtime/`:

### 1. Core Module (`Core/`)
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

### 2. Scroller Module (`Scroller/`)
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

### 3. ScrollRect Module (`ScrollRect/`)
Traditional ScrollRect-style scroll views (vertical/horizontal lists).

- **`FancyScrollRect<TItemData, TContext>`** - Base for ScrollRect-style views
  - Does NOT support infinite scrolling or snapping
  - Supports cell reuse with configurable margin (`reuseCellMarginCount`)
  - Supports padding (head/tail) and spacing
  - Must implement `CellSize` property
  - Cells implement `FancyScrollRectCell<TItemData, TContext>`

### 4. GridView Module (`GridView/`)
Grid layout scroll views.

- **`FancyGridView<TItemData, TContext>`** - Base for grid-style views
  - Extends `FancyScrollRect` (inherits same limitations: no infinite scroll/snap)
  - Uses cell groups for rows/columns
  - Configuration:
    - `startAxisCellCount`: number of cells in perpendicular axis
    - `cellSize`: Vector2 for cell dimensions
    - `startAxisSpacing`: spacing in perpendicular axis
  - Cell groups implement `FancyCellGroup<TItemData, TContext>`

## The Context Pattern

The `Context` object enables communication between cells and scroll view:

- **Shared Instance**: Same Context instance is shared between ScrollView and all Cells
- **Use Cases**:
  - Cell → ScrollView: Pass events (e.g., cell clicked)
  - ScrollView → Cell: Pass shared state (e.g., selected index)
  - See `Assets/FancyScrollView/Examples/02_FocusOn/Context.cs` for example

```csharp
// Example Context
class Context
{
    public int SelectedIndex = -1;
    public Action<int> OnCellClicked;
}
```

## Key Design Patterns

1. **Cell Pooling**: Only visible cells are created; cells are reused as you scroll
2. **Normalized Positioning**: Cells receive position as 0.0-1.0 value for custom animations
3. **Template Method**: Override `UpdateContent` (data) and `UpdatePosition` (animation) in cells
4. **Generic Types**: `TItemData` for your data model, `TContext` for cell-view communication
