# 基本的な実装

このガイドでは、FancyScrollView を使った最小限のスクロールビューの実装方法を説明します。

## 実装の流れ

FancyScrollView を使うには、以下の4つのステップが必要です:

1. **ItemData の定義** - 表示するデータの構造
2. **Cell の実装** - データを表示するUI要素
3. **ScrollView の実装** - スクロールビュー本体
4. **Scene のセットアップ** - Unity エディタでの設定

## ステップ1: ItemData の定義

まず、スクロールビューで表示するデータの構造を定義します。

```csharp
public class ItemData
{
    public string Message;

    public ItemData(string message)
    {
        Message = message;
    }
}
```

## ステップ2: Cell の実装

`FancyCell<TItemData>` を継承して、セルを実装します。

```csharp
using UnityEngine;
using UnityEngine.UI;
using FancyScrollView;

public class Cell : FancyCell<ItemData>
{
    [SerializeField] Text message;
    [SerializeField] Animator animator;

    static class AnimatorHash
    {
        public static readonly int Scroll = Animator.StringToHash("scroll");
    }

    // データが更新された時に呼ばれる
    public override void UpdateContent(ItemData itemData)
    {
        message.text = itemData.Message;
    }

    // スクロール位置が変わった時に呼ばれる
    public override void UpdatePosition(float position)
    {
        currentPosition = position;

        if (animator.isActiveAndEnabled)
        {
            // position (0.0-1.0) を使ってアニメーション制御
            animator.Play(AnimatorHash.Scroll, -1, position);
        }

        animator.speed = 0;
    }

    // GameObject が非アクティブになると Animator がリセットされるため
    // 現在位置を保持して OnEnable で再設定
    float currentPosition = 0;

    void OnEnable() => UpdatePosition(currentPosition);
}
```

### Cell の重要なメソッド

#### `UpdateContent(TItemData itemData)`

- セルに新しいデータが割り当てられた時に呼ばれます
- UI要素(Text, Image など)にデータをバインドします
- スクロール中、セルが再利用される際にも呼ばれます

#### `UpdatePosition(float position)`

- スクロール位置が変化した時に呼ばれます
- `position` は 0.0 ～ 1.0 の正規化された値です
  - `0.0`: ビューポート開始位置(画面外上/左)
  - `0.5`: ビューポート中央
  - `1.0`: ビューポート終了位置(画面外下/右)
- この値を使ってアニメーション、スケール、透明度などを制御できます

## ステップ3: ScrollView の実装

`FancyScrollView<TItemData>` を継承して、スクロールビューを実装します。

```csharp
using UnityEngine;
using System.Collections.Generic;
using FancyScrollView;

public class ScrollView : FancyScrollView<ItemData>
{
    [SerializeField] Scroller scroller;
    [SerializeField] GameObject cellPrefab;

    protected override GameObject CellPrefab => cellPrefab;

    protected override void Initialize()
    {
        base.Initialize();

        // Scroller のスクロール位置が変わった時に UpdatePosition を呼ぶ
        scroller.OnValueChanged(UpdatePosition);
    }

    public void UpdateData(IList<ItemData> items)
    {
        // データを設定してセルを更新
        UpdateContents(items);

        // Scroller にアイテム数を設定
        scroller.SetTotalCount(items.Count);
    }
}
```

### ScrollView の重要な要素

#### `CellPrefab` プロパティ

- セルの Prefab を返す必要があります
- この Prefab には前述の `Cell` コンポーネントがアタッチされている必要があります

#### `Initialize()` メソッド

- 初回のセル生成前に1回だけ呼ばれます
- `Scroller` とスクロール位置の同期を設定します

#### `UpdateData()` メソッド

- アプリケーション側から呼び出すカスタムメソッド
- `UpdateContents()` でデータを設定
- `scroller.SetTotalCount()` でアイテム総数を設定

## ステップ4: Scene のセットアップ

### Hierarchy 構成

```
Canvas
└── ScrollView (GameObject)
    ├── ScrollView (Component: ScrollView.cs)
    ├── Scroller (Component: Scroller.cs)
    └── Viewport (GameObject)
        └── Content (Transform) ← cellContainer として設定
```

### ScrollView の Inspector 設定

1. **FancyScrollView 設定**
   - `Cell Interval`: セル間隔 (例: 0.2)
   - `Scroll Offset`: スクロール位置の基準 (例: 0.5 で中央)
   - `Loop`: 無限スクロールの有効化 (基本実装では false)
   - `Cell Container`: Content Transform を設定

2. **ScrollView コンポーネント設定**
   - `Scroller`: Scroller コンポーネントを設定
   - `Cell Prefab`: Cell Prefab を設定

### Scroller の Inspector 設定

- `Viewport`: Viewport Transform を設定
- `Direction`: `Vertical` または `Horizontal`
- `Movement Type`:
  - `Elastic`: 端で跳ね返る
  - `Clamped`: 端で停止
  - `Unrestricted`: 制限なし(無限スクロール用)

### Cell Prefab の作成

1. UI要素(Panel など)を作成
2. Cell コンポーネントをアタッチ
3. Text, Image などの参照を設定
4. Animator をアタッチ(アニメーションを使う場合)
5. Prefab 化

## ステップ5: データの設定

スクリプトからデータを設定します:

```csharp
using UnityEngine;
using System.Linq;

public class Example : MonoBehaviour
{
    [SerializeField] ScrollView scrollView;

    void Start()
    {
        // サンプルデータを生成
        var items = Enumerable.Range(0, 50)
            .Select(i => new ItemData($"Cell {i}"))
            .ToList();

        // スクロールビューにデータを設定
        scrollView.UpdateData(items);
    }
}
```

## セル間隔とスクロールオフセット

### Cell Interval (cellInterval)

セル同士の間隔を 0.01 ～ 1.0 の範囲で設定します。

- `0.2`: セルが密集して表示
- `0.5`: 適度な間隔
- `1.0`: セルが大きく離れて表示

### Scroll Offset (scrollOffset)

スクロール位置の基準点を 0.0 ～ 1.0 の範囲で設定します。

- `0.0`: 最初のセルが画面上端/左端に配置
- `0.5`: 最初のセルが画面中央に配置(デフォルト推奨)
- `1.0`: 最初のセルが画面下端/右端に配置

例: `scrollOffset = 0.5`, スクロール位置 `0` の場合、インデックス 0 のセルが中央に表示されます。

## アニメーションの設定

`UpdatePosition(float position)` で受け取る `position` 値を使って、様々なエフェクトを実装できます:

### スケールアニメーション

```csharp
public override void UpdatePosition(float position)
{
    // 中央(0.5)に近いほど大きく表示
    float scale = 1f - Mathf.Abs(0.5f - position);
    transform.localScale = Vector3.one * scale;
}
```

### フェードアニメーション

```csharp
public override void UpdatePosition(float position)
{
    var canvasGroup = GetComponent<CanvasGroup>();
    // 中央(0.5)に近いほど不透明に
    canvasGroup.alpha = 1f - Mathf.Abs(0.5f - position) * 2f;
}
```

### Animator を使った複雑なアニメーション

Animator Controller で "scroll" という Float パラメータを作成し、0.0 ～ 1.0 の範囲でアニメーションブレンドツリーを設定します。

```csharp
public override void UpdatePosition(float position)
{
    animator.Play(AnimatorHash.Scroll, -1, position);
    animator.speed = 0; // 手動制御のため停止
}
```

## まとめ

基本的な実装には以下が必要です:

1. `ItemData` クラス - データ構造
2. `FancyCell<ItemData>` を継承した `Cell` - UI表示
3. `FancyScrollView<ItemData>` を継承した `ScrollView` - スクロール制御
4. Scene での適切な設定

次のステップ:
- [Context の使い方](using-context.md) - セル選択などの高度な機能
- [無限スクロール](infinite-scroll.md) - ループするスクロールビュー
- [カスタムアニメーション](custom-animations.md) - より高度なエフェクト
