# GridView の使い方

GridView は、グリッド(格子状)レイアウトのスクロールビューを実装するための機能です。複数列/行のセルを並べたレイアウトに対応します。

## GridView の特徴

### できること
- ✓ グリッドレイアウト(2×N, 3×N など)
- ✓ セルの再利用
- ✓ 縦/横スクロール
- ✓ セルサイズとスペーシングの設定
- ✓ パディング対応

### できないこと
- ✗ 無限スクロール
- ✗ スナップ機能
- ✗ カスタムアニメーション(正規化位置ベース)

**注意**: GridView は内部的に `FancyScrollRect` を拡張しているため、ScrollRect と同じ制限があります。

## 基本的な実装

### ステップ1: Cell の実装

`FancyCell<TItemData, TContext>` を継承します(通常の FancyCell と同じ)。

```csharp
using UnityEngine;
using UnityEngine.UI;
using FancyScrollView;

public class Cell : FancyCell<ItemData, Context>
{
    [SerializeField] Text message;
    [SerializeField] Image image;
    [SerializeField] Button button;

    public override void Initialize()
    {
        button.onClick.AddListener(() => Context.OnCellClicked?.Invoke(Index));
    }

    public override void UpdateContent(ItemData itemData)
    {
        message.text = itemData.Message;

        var isSelected = Context.SelectedIndex == Index;
        image.color = isSelected ? Color.cyan : Color.white;
    }

    // GridView では UpdatePosition は使用されません
}
```

### ステップ2: GridView の実装

`FancyGridView<TItemData, TContext>` を継承します。

```csharp
using UnityEngine;
using System.Collections.Generic;
using FancyScrollView;

public class GridView : FancyGridView<ItemData, Context>
{
    [SerializeField] Scroller scroller;
    [SerializeField] GameObject cellPrefab;

    protected override GameObject CellPrefab => cellPrefab;

    protected override void Initialize()
    {
        base.Initialize();

        Context.OnCellClicked = OnCellClicked;

        scroller.OnValueChanged(UpdatePosition);
    }

    void OnCellClicked(int index)
    {
        Context.SelectedIndex = index;
        Refresh();
    }

    public void UpdateData(IList<ItemData> items)
    {
        UpdateContents(items);
        scroller.SetTotalCount(items.Count);
    }
}
```

## グリッドの設定

### Inspector での設定

GridView コンポーネントには以下の設定があります:

#### 基本設定
- **Cell Size**: セルのサイズ (Vector2)
  - 例: `(150, 150)` で 150×150 の正方形セル
- **Start Axis Cell Count**: 交差軸方向のセル数
  - 縦スクロールの場合: 列数(横に何個並べるか)
  - 横スクロールの場合: 行数(縦に何個並べるか)

#### スペーシング
- **Spacing**: スクロール方向のセル間隔
- **Start Axis Spacing**: 交差軸方向のセル間隔

#### パディング
- **Padding Head**: リスト先頭の余白
- **Padding Tail**: リスト末尾の余白

### コードでの設定

```csharp
public class GridView : FancyGridView<ItemData, Context>
{
    void Awake()
    {
        // セルサイズ
        cellSize = new Vector2(150f, 150f);

        // 交差軸のセル数(列数)
        startAxisCellCount = 3; // 3列グリッド

        // スペーシング
        spacing = 10f;           // 縦方向の間隔
        startAxisSpacing = 10f;  // 横方向の間隔

        // パディング
        paddingHead = 20f;
        paddingTail = 20f;
    }
}
```

## グリッドレイアウトの仕組み

### 縦スクロール × 3列グリッド の例

```
[Header Padding: 20px]

[Cell 0] [Cell 1] [Cell 2]
        (Spacing: 10px)
[Cell 3] [Cell 4] [Cell 5]
        (Spacing: 10px)
[Cell 6] [Cell 7] [Cell 8]

[Tail Padding: 20px]
```

- **スクロール方向**: 縦(上下)
- **Start Axis**: 横方向
- **Start Axis Cell Count**: 3(横に3つ並ぶ)

### 横スクロール × 2行グリッド の例

```
         [Header][Cell 0][Spacing][Cell 2][Spacing]...
[Row 0]  Padding [Cell 1]  10px   [Cell 3]  10px
         [  20px]
```

- **スクロール方向**: 横(左右)
- **Start Axis**: 縦方向
- **Start Axis Cell Count**: 2(縦に2つ並ぶ)

## 完全な実装例

```csharp
using UnityEngine;
using UnityEngine.UI;
using System.Collections.Generic;
using System.Linq;
using FancyScrollView;

// ItemData
public class ItemData
{
    public int Index;
    public string Message;

    public ItemData(int index)
    {
        Index = index;
        Message = $"Cell {index}";
    }
}

// Context
public class Context
{
    public int SelectedIndex = -1;
    public Action<int> OnCellClicked;
}

// Cell
public class Cell : FancyCell<ItemData, Context>
{
    [SerializeField] Text message;
    [SerializeField] Image background;
    [SerializeField] Button button;

    public override void Initialize()
    {
        button.onClick.AddListener(() => Context.OnCellClicked?.Invoke(Index));
    }

    public override void UpdateContent(ItemData itemData)
    {
        message.text = itemData.Message;

        var isSelected = Context.SelectedIndex == Index;
        background.color = isSelected
            ? new Color(0f, 1f, 1f, 0.5f)  // 選択: シアン
            : new Color(1f, 1f, 1f, 0.5f); // 非選択: 白
    }
}

// GridView
public class GridView : FancyGridView<ItemData, Context>
{
    [SerializeField] Scroller scroller;
    [SerializeField] GameObject cellPrefab;

    protected override GameObject CellPrefab => cellPrefab;

    void Awake()
    {
        // 3列グリッド
        cellSize = new Vector2(200f, 200f);
        startAxisCellCount = 3;
        spacing = 10f;
        startAxisSpacing = 10f;
        paddingHead = 20f;
        paddingTail = 20f;
    }

    protected override void Initialize()
    {
        base.Initialize();

        Context.OnCellClicked = OnCellClicked;
        scroller.OnValueChanged(UpdatePosition);
    }

    void OnCellClicked(int index)
    {
        Context.SelectedIndex = index;
        Refresh();
    }

    public void UpdateData(IList<ItemData> items)
    {
        UpdateContents(items);
        scroller.SetTotalCount(items.Count);
    }
}

// 使用例
public class Example : MonoBehaviour
{
    [SerializeField] GridView gridView;

    void Start()
    {
        // 30個のアイテムを生成
        var items = Enumerable.Range(0, 30)
            .Select(i => new ItemData(i))
            .ToList();

        gridView.UpdateData(items);
    }
}
```

## レイアウト調整

### 列数/行数の変更

```csharp
// 2列グリッド
startAxisCellCount = 2;

// 4列グリッド
startAxisCellCount = 4;
```

### セルのアスペクト比

```csharp
// 正方形
cellSize = new Vector2(150f, 150f);

// 横長
cellSize = new Vector2(200f, 100f);

// 縦長
cellSize = new Vector2(100f, 200f);
```

### 画面幅に合わせた動的列数

```csharp
void Start()
{
    var rectTransform = GetComponent<RectTransform>();
    var viewportWidth = rectTransform.rect.width;

    var cellWidth = 150f;
    var spacing = 10f;

    // ビューポート幅から列数を計算
    var columnCount = Mathf.FloorToInt((viewportWidth + spacing) / (cellWidth + spacing));
    startAxisCellCount = Mathf.Max(1, columnCount);
}
```

## セルグループ(CellGroup)

GridView は内部的にセルを「グループ」として管理します。各グループは交差軸方向のセルをまとめたものです。

### グループの概念

縦スクロール × 3列の場合:

```
[Group 0] → [Cell 0] [Cell 1] [Cell 2]
[Group 1] → [Cell 3] [Cell 4] [Cell 5]
[Group 2] → [Cell 6] [Cell 7] [Cell 8]
```

各グループは `FancyCellGroup` として管理され、スクロール時にグループ単位で再利用されます。

### カスタムセルグループ

デフォルトの `FancyCellGroup` を拡張できます:

```csharp
public class CustomCellGroup : FancyCellGroup<ItemData, Context>
{
    protected override void UpdateContents(IList<ItemData> items)
    {
        // グループ全体に対する処理
        base.UpdateContents(items);
    }
}
```

通常は標準の `FancyCellGroup` で十分です。

## スクロール方向

GridView は縦スクロールと横スクロールの両方に対応:

```csharp
// 縦スクロール(デフォルト)
scroller.ScrollDirection = ScrollDirection.Vertical;

// 横スクロール
scroller.ScrollDirection = ScrollDirection.Horizontal;
```

### 横スクロールの注意点

横スクロールの場合、`startAxisCellCount` は「行数」を表します:

```csharp
// 横スクロール × 2行
scroller.ScrollDirection = ScrollDirection.Horizontal;
startAxisCellCount = 2; // 2行
```

## パフォーマンス最適化

### 再利用マージン

画面外に保持するグループ数を設定:

```csharp
protected override int ReuseCellMarginCount => 1;
```

値が大きいほどスムーズですが、メモリ使用量が増えます。

### セル数の考慮

グリッドでは `startAxisCellCount × 表示行数` 分のセルが生成されるため:

```csharp
// 3列 × 5行表示 = 15セル生成
startAxisCellCount = 3;
// ビューポート高さで約5行表示される場合
```

メモリに注意して列数を設定してください。

## トラブルシューティング

### 問題: セルが正しく配置されない

**原因**: `cellSize` または `startAxisCellCount` が正しく設定されていない

**解決策**:
```csharp
// Inspector または Awake() で明示的に設定
cellSize = new Vector2(150f, 150f);
startAxisCellCount = 3;
```

### 問題: セルが重なる

**原因**: `spacing` または `startAxisSpacing` が不足

**解決策**:
```csharp
spacing = 10f;          // スクロール方向
startAxisSpacing = 10f; // 交差軸方向
```

### 問題: セルが表示されない

**原因**: `cellSize` が大きすぎてビューポート外にある

**解決策**: セルサイズを適切な値に調整

## まとめ

GridView の実装には:

1. **`FancyGridView<TItemData, TContext>`** を継承
2. **`cellSize`** でセルサイズを設定
3. **`startAxisCellCount`** で列数(または行数)を設定
4. **`spacing`** と **`startAxisSpacing`** でセル間隔を設定

GridView は ScrollRect を基盤としているため:
- ✗ 無限スクロール不可
- ✗ スナップ不可
- ✓ スクロールバー対応
- ✓ セル再利用によるパフォーマンス最適化

次のステップ:
- [カスタムアニメーション](custom-animations.md) - FancyScrollView でのアニメーション
- [ScrollRect 形式](scrollrect-usage.md) - ScrollRect の詳細
