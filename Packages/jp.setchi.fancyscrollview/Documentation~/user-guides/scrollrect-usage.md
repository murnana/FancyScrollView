# ScrollRect 形式の使い方

ScrollRect 形式は、Unity の標準 `ScrollRect` に似た従来型のスクロールリストを実装するための機能です。垂直/水平方向の単純なリスト表示に適しています。

## ScrollRect 形式の特徴

### できること
- ✓ 縦/横スクロールリスト
- ✓ セルの再利用(パフォーマンス最適化)
- ✓ スクロールバー対応
- ✓ パディング(上下/左右の余白)
- ✓ セル間のスペース
- ✓ 可変セルサイズ

### できないこと
- ✗ 無限スクロール
- ✗ スナップ機能
- ✗ カスタムアニメーション(正規化位置ベース)

**無限スクロールやスナップが必要な場合は、標準の `FancyScrollView` を使用してください。**

## 基本的な実装

### ステップ1: Cell の実装

`FancyScrollRectCell<TItemData, TContext>` を継承します。

```csharp
using UnityEngine;
using UnityEngine.UI;
using FancyScrollView;

public class Cell : FancyScrollRectCell<ItemData, Context>
{
    [SerializeField] Text message;
    [SerializeField] Button button;

    public override void Initialize()
    {
        button.onClick.AddListener(() => Context.OnCellClicked?.Invoke(Index));
    }

    public override void UpdateContent(ItemData itemData)
    {
        message.text = itemData.Message;
    }
}
```

**注意**: `UpdatePosition()` メソッドはありません。ScrollRect 形式ではセル位置ベースのアニメーションは使用できません。

### ステップ2: ScrollView の実装

`FancyScrollRect<TItemData, TContext>` を継承します。

```csharp
using UnityEngine;
using System.Collections.Generic;
using FancyScrollView;

public class ScrollView : FancyScrollRect<ItemData, Context>
{
    [SerializeField] Scroller scroller;
    [SerializeField] GameObject cellPrefab;

    // セルのサイズを返す(必須)
    protected override GameObject CellPrefab => cellPrefab;

    // セルサイズの指定(必須)
    protected override float CellSize => 100f; // セルの高さ(縦)または幅(横)

    protected override void Initialize()
    {
        base.Initialize();

        Context.OnCellClicked = OnCellClicked;

        scroller.OnValueChanged(UpdatePosition);
        scroller.OnSelectionChanged(UpdateSelection);
    }

    void OnCellClicked(int index)
    {
        Context.SelectedIndex = index;
        Refresh();
        scroller.ScrollTo(index, 0.35f);
    }

    public void UpdateData(IList<ItemData> items)
    {
        UpdateContents(items);
        scroller.SetTotalCount(items.Count);
    }

    void UpdateSelection(int index)
    {
        Context.SelectedIndex = index;
        Refresh();
    }
}
```

## セルサイズの設定

### 固定サイズ

全セルが同じサイズの場合:

```csharp
protected override float CellSize => 100f;
```

### 可変サイズ

セルごとにサイズが異なる場合、`GetCellSize()` をオーバーライド:

```csharp
protected override float GetCellSize(int index)
{
    // アイテムのデータに基づいてサイズを決定
    return ItemsSource[index].IsLarge ? 150f : 100f;
}
```

例: テキストの長さに応じてセルサイズを変更

```csharp
protected override float GetCellSize(int index)
{
    var item = ItemsSource[index];
    var textLength = item.Message.Length;

    // 基本サイズ + テキスト長に応じた追加
    return 80f + Mathf.Min(textLength * 2f, 100f);
}
```

## スクロールバーの追加

### Hierarchy 構成

```
Canvas
└── ScrollView
    ├── Viewport
    │   └── Content
    └── Scrollbar (Scrollbar コンポーネント)
        └── Sliding Area
            └── Handle
```

### Scrollbar の設定

1. `GameObject > UI > Scrollbar` でスクロールバーを作成
2. Direction を設定:
   - 縦スクロール: `Bottom To Top`
   - 横スクロール: `Left To Right`

### Scroller にスクロールバーを接続

```csharp
[SerializeField] Scroller scroller;
[SerializeField] Scrollbar scrollbar;

protected override void Initialize()
{
    base.Initialize();

    scroller.OnValueChanged(UpdatePosition);

    // スクロールバーとの同期
    scrollbar.onValueChanged.AddListener(OnScrollbarValueChanged);
}

void OnScrollbarValueChanged(float value)
{
    // スクロールバーの値(0-1)をスクロール位置に変換
    scroller.ScrollTo(value * (ItemsSource.Count - 1), 0.1f);
}
```

または、Inspector で:
- Scroller の `Scrollbar` フィールドに Scrollbar をドラッグ&ドロップ

## パディングとスペーシング

### パディング

リストの最初と最後に余白を追加:

```csharp
public class ScrollView : FancyScrollRect<ItemData, Context>
{
    protected override float CellSize => 100f;

    // リスト先頭のパディング
    protected override float PaddingHead => 20f;

    // リスト末尾のパディング
    protected override float PaddingTail => 20f;
}
```

### スペーシング

セル間の間隔を設定:

```csharp
protected override float Spacing => 10f;
```

### 視覚的なイメージ

```
[PaddingHead: 20px]
[Cell 0]
[Spacing: 10px]
[Cell 1]
[Spacing: 10px]
[Cell 2]
[PaddingTail: 20px]
```

## セルの再利用

ScrollRect 形式では、表示領域外のセルは自動的に非表示になり、再利用されます。

### 再利用マージンの設定

画面外のセルをどこまで保持するか設定できます:

```csharp
public class ScrollView : FancyScrollRect<ItemData, Context>
{
    // 表示範囲外に保持するセルの数
    // デフォルト: 0(表示中のセルのみ生成)
    protected override int ReuseCellMarginCount => 1;
}
```

- `0`: 表示中のセルのみ生成(最小メモリ使用)
- `1`: 表示範囲の前後1セルずつ余分に生成(スクロールがスムーズ)
- `2+`: さらに余裕を持たせる

## レイアウトの調整

### 揃え位置(Alignment)

セルをビューポート内でどこに揃えるか設定:

```csharp
// 上揃え(縦スクロール)、左揃え(横スクロール)
public enum Alignment
{
    Upper,  // 上/左
    Middle, // 中央
    Lower   // 下/右
}

// ScrollView で設定
void Awake()
{
    // デフォルトは Upper
    alignment = Alignment.Middle; // 中央揃え
}
```

### スクロール方向

Inspector または コードで設定:

```csharp
scroller.ScrollDirection = ScrollDirection.Vertical;   // 縦
scroller.ScrollDirection = ScrollDirection.Horizontal; // 横
```

## 完全な実装例

```csharp
using UnityEngine;
using UnityEngine.UI;
using System.Collections.Generic;
using FancyScrollView;

// ItemData
public class ItemData
{
    public string Message;
    public ItemData(string message) => Message = message;
}

// Context
public class Context
{
    public int SelectedIndex = -1;
    public Action<int> OnCellClicked;
}

// Cell
public class Cell : FancyScrollRectCell<ItemData, Context>
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
        background.color = isSelected ? Color.cyan : Color.white;
    }
}

// ScrollView
public class ScrollView : FancyScrollRect<ItemData, Context>
{
    [SerializeField] Scroller scroller;
    [SerializeField] GameObject cellPrefab;
    [SerializeField] Scrollbar scrollbar;

    protected override GameObject CellPrefab => cellPrefab;
    protected override float CellSize => 100f;
    protected override float Spacing => 10f;
    protected override float PaddingHead => 20f;
    protected override float PaddingTail => 20f;

    protected override void Initialize()
    {
        base.Initialize();

        Context.OnCellClicked = OnCellClicked;

        scroller.OnValueChanged(UpdatePosition);
        scroller.OnSelectionChanged(UpdateSelection);

        if (scrollbar != null)
        {
            scrollbar.onValueChanged.AddListener(OnScrollbarValueChanged);
        }
    }

    void OnCellClicked(int index)
    {
        Context.SelectedIndex = index;
        Refresh();
        scroller.ScrollTo(index, 0.35f, Ease.OutCubic);
    }

    void UpdateSelection(int index)
    {
        Context.SelectedIndex = index;
        Refresh();
    }

    void OnScrollbarValueChanged(float value)
    {
        var targetIndex = Mathf.Clamp(
            Mathf.RoundToInt(value * (ItemsSource.Count - 1)),
            0,
            ItemsSource.Count - 1
        );
        scroller.ScrollTo(targetIndex, 0.1f);
    }

    public void UpdateData(IList<ItemData> items)
    {
        UpdateContents(items);
        scroller.SetTotalCount(items.Count);

        // スクロールバーのサイズを更新
        if (scrollbar != null)
        {
            var viewportSize = 5; // ビューポートに表示される数
            scrollbar.size = Mathf.Clamp01((float)viewportSize / items.Count);
        }
    }
}
```

## FancyScrollView との違い

| 機能                     | FancyScrollView | FancyScrollRect |
|--------------------------|-----------------|-----------------|
| 無限スクロール           | ✓               | ✗               |
| スナップ                 | ✓               | ✗               |
| UpdatePosition アニメ     | ✓               | ✗               |
| スクロールバー           | 手動実装        | ✓ 簡単          |
| 可変セルサイズ           | 困難            | ✓ 簡単          |
| パディング/スペーシング   | 手動実装        | ✓ 組込み        |
| 使用するCell基底クラス    | FancyCell       | FancyScrollRectCell |

## まとめ

ScrollRect 形式は以下の場合に適しています:

- ✓ 標準的な縦/横スクロールリスト
- ✓ スクロールバーが必要
- ✓ セルサイズが固定または可変
- ✓ シンプルな実装を求める場合

無限スクロールやカスタムアニメーションが必要な場合は、標準の `FancyScrollView` を使用してください。

次のステップ:
- [GridView](gridview-usage.md) - グリッドレイアウトの実装
- [カスタムアニメーション](custom-animations.md) - FancyScrollView でのアニメーション
