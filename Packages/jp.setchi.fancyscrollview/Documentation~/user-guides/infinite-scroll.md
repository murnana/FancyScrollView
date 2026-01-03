# 無限スクロールの実装

無限スクロールは、セルを循環的に配置して、リストの最後まで到達しても最初に戻り、無限にスクロールできる機能です。

## 無限スクロールの仕組み

通常のスクロール:
```
[Cell 0] [Cell 1] [Cell 2] ... [Cell N-1] [終端]
```

無限スクロール:
```
... [Cell N-1] [Cell 0] [Cell 1] [Cell 2] ... [Cell N-1] [Cell 0] ...
```

最後のセルの次に最初のセルが、最初のセルの前に最後のセルが配置されます。

## 実装方法

無限スクロールを有効にするには、**2つの設定が必要**です:

### 1. FancyScrollView の `loop` を `true` に設定

```csharp
public class ScrollView : FancyScrollView<ItemData>
{
    [SerializeField] Scroller scroller;
    [SerializeField] GameObject cellPrefab;

    protected override GameObject CellPrefab => cellPrefab;

    void Awake()
    {
        // loop を有効化(Inspector でも設定可能)
        loop = true;
    }

    protected override void Initialize()
    {
        base.Initialize();
        scroller.OnValueChanged(UpdatePosition);
    }

    public void UpdateData(IList<ItemData> items)
    {
        UpdateContents(items);
        scroller.SetTotalCount(items.Count);
    }
}
```

### 2. Scroller の `MovementType` を `Unrestricted` に設定

```csharp
// Inspector で設定、または:
scroller.MovementType = MovementType.Unrestricted;
```

## Inspector での設定

### FancyScrollView コンポーネント

- **Loop**: チェックを入れる ✓

### Scroller コンポーネント

- **Movement Type**: `Unrestricted` を選択

### 設定の組み合わせ

| `loop` | `MovementType`    | 結果                                      |
|--------|-------------------|-------------------------------------------|
| true   | Unrestricted      | ✓ 無限スクロール(正しい設定)             |
| true   | Elastic / Clamped | セルは循環するが端で停止(不完全)          |
| false  | Unrestricted      | 端を超えてスクロールできるが循環しない    |
| false  | Elastic / Clamped | 通常のスクロール                          |

**重要**: 無限スクロールには **両方とも有効化が必須** です。

## 完全な実装例

```csharp
using UnityEngine;
using System.Collections.Generic;
using System.Linq;
using FancyScrollView;

public class InfiniteScrollView : FancyScrollView<ItemData, Context>
{
    [SerializeField] Scroller scroller;
    [SerializeField] GameObject cellPrefab;

    protected override GameObject CellPrefab => cellPrefab;

    void Awake()
    {
        // 無限スクロールを有効化
        loop = true;
    }

    protected override void Initialize()
    {
        base.Initialize();

        Context.OnCellClicked = OnCellClicked;
        scroller.OnValueChanged(UpdatePosition);

        // Movement Type を Unrestricted に設定
        scroller.MovementType = MovementType.Unrestricted;
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
}
```

## 注意点

### 最小アイテム数

無限スクロールを正しく動作させるには、**十分な数のアイテムが必要**です。

- アイテム数が少なすぎると、同じセルが複数回表示されてしまいます
- 推奨: ビューポートに収まる数の **2倍以上**のアイテム

```csharp
// 例: ビューポートに5個表示される場合、10個以上のアイテムが推奨
var items = Enumerable.Range(0, 20)
    .Select(i => new ItemData($"Cell {i}"))
    .ToList();
```

### スクロール位置の管理

無限スクロールでは、スクロール位置が負の値や、アイテム数を超える値になることがあります:

```csharp
// 通常のスクロール: 0 ～ items.Count-1
// 無限スクロール: -∞ ～ +∞
```

特定のインデックスにスクロールする場合:

```csharp
// OK: インデックスを直接指定
scroller.ScrollTo(5, 0.35f);

// OK: 現在位置から相対的に移動
scroller.ScrollTo(scroller.Position + 3, 0.35f);
```

### スナップとの組み合わせ

無限スクロールはスナップ機能と組み合わせて使用できます:

```csharp
// Scroller の Inspector で設定
// Snap:
//   - Enable: true
//   - Velocity Threshold: 0.5
//   - Duration: 0.3
```

または、コードで:

```csharp
scroller.Snap.Enable = true;
scroller.Snap.VelocityThreshold = 0.5f;
scroller.Snap.Duration = 0.3f;
```

## 実践例: タブバー

無限スクロールはタブバーの実装に適しています:

```csharp
public class TabBar : FancyScrollView<TabData, Context>
{
    [SerializeField] Scroller scroller;
    [SerializeField] GameObject tabPrefab;

    protected override GameObject CellPrefab => tabPrefab;

    void Awake()
    {
        loop = true;
    }

    protected override void Initialize()
    {
        base.Initialize();

        Context.OnTabClicked = OnTabClicked;
        scroller.OnValueChanged(UpdatePosition);
        scroller.MovementType = MovementType.Unrestricted;

        // スナップを有効化
        scroller.Snap.Enable = true;
        scroller.Snap.VelocityThreshold = 0.5f;
        scroller.Snap.Duration = 0.3f;
    }

    void OnTabClicked(int index)
    {
        // タブ選択時の処理
        SelectTab(index);
    }

    public void SelectTab(int index)
    {
        Context.SelectedIndex = index;
        Refresh();
        scroller.ScrollTo(index, 0.35f, Ease.OutCubic);
    }

    public void UpdateTabs(IList<TabData> tabs)
    {
        UpdateContents(tabs);
        scroller.SetTotalCount(tabs.Count);

        // 最初のタブを選択
        if (tabs.Count > 0)
        {
            SelectTab(0);
        }
    }
}
```

## MovementType の詳細

### Unrestricted (無制限)

- スクロール範囲に制限なし
- 無限スクロールに必須
- 慣性でどこまでもスクロール可能

### Elastic (弾性)

- 端まで到達すると跳ね返る
- 端を少し超えてスクロール可能(引っ張って離すと戻る)
- 通常のスクロールビューに適している

### Clamped (固定)

- 端で完全に停止
- 端を超えてスクロール不可
- 厳密な範囲制限が必要な場合に使用

## トラブルシューティング

### 問題: セルが循環しない

**原因**: `loop` が `false` になっている

**解決策**:
```csharp
loop = true; // コードで設定
// または Inspector で Loop にチェック
```

### 問題: 端で停止してしまう

**原因**: `MovementType` が `Unrestricted` になっていない

**解決策**:
```csharp
scroller.MovementType = MovementType.Unrestricted;
// または Inspector で Movement Type を Unrestricted に設定
```

### 問題: 同じセルが何度も表示される

**原因**: アイテム数が少なすぎる

**解決策**: アイテム数を増やす(ビューポートに表示される数の2倍以上推奨)

### 問題: スクロールが不安定

**原因**: セル間隔(`cellInterval`)が大きすぎる、またはアイテム数が少ない

**解決策**:
- `cellInterval` を小さくする(0.1 ～ 0.3 程度)
- アイテム数を増やす

## まとめ

無限スクロールの実装には:

1. **`loop = true`** を設定
2. **`MovementType.Unrestricted`** を設定
3. 十分な数のアイテムを用意(ビューポートの2倍以上推奨)
4. 必要に応じてスナップを設定

この2つの設定により、シームレスな無限スクロールが実現できます。

次のステップ:
- [カスタムアニメーション](custom-animations.md) - より高度なエフェクト
- [ScrollRect 形式](scrollrect-usage.md) - 従来型リストの実装
