# API リファレンス

このドキュメントは、FancyScrollView の主要なクラスとメソッドのリファレンスです。

詳細な API ドキュメントは [公式サイト](https://setchi.jp/FancyScrollView/api/FancyScrollView.html) を参照してください。

## Core API

### FancyScrollView<TItemData, TContext>

スクロールビューの基底クラス。

#### プロパティ

| プロパティ | 型 | 説明 |
|-----------|-----|------|
| `ItemsSource` | `IList<TItemData>` | 表示するデータのリスト |
| `Context` | `TContext` | セルと共有するコンテキスト |
| `CellPrefab` | `GameObject` | セルの Prefab (抽象・要実装) |

#### 保護されたフィールド

| フィールド | 型 | デフォルト | 説明 |
|-----------|-----|-----------|------|
| `cellInterval` | `float` | 0.2 | セル間隔 (0.01-1.0) |
| `scrollOffset` | `float` | 0.5 | スクロール位置の基準 (0.0-1.0) |
| `loop` | `bool` | false | セルを循環配置するか |
| `cellContainer` | `Transform` | null | セルの親となる Transform |

#### メソッド

##### `Initialize()`
```csharp
protected virtual void Initialize()
```
初期化処理。セル生成前に1回だけ呼ばれます。

**使用例**:
```csharp
protected override void Initialize()
{
    base.Initialize();
    scroller.OnValueChanged(UpdatePosition);
}
```

##### `UpdateContents(IList<TItemData>)`
```csharp
protected virtual void UpdateContents(IList<TItemData> itemsSource)
```
データを設定してセルを更新します。

**パラメータ**:
- `itemsSource`: 表示するデータのリスト

**使用例**:
```csharp
public void UpdateData(IList<ItemData> items)
{
    UpdateContents(items);
    scroller.SetTotalCount(items.Count);
}
```

##### `UpdatePosition(float)`
```csharp
protected virtual void UpdatePosition(float position)
```
スクロール位置を更新します。

**パラメータ**:
- `position`: スクロール位置

**使用例**:
```csharp
scroller.OnValueChanged(UpdatePosition);
```

##### `Refresh()`
```csharp
protected virtual void Refresh()
```
セルのレイアウトと表示内容を強制的に更新します。

**使用例**:
```csharp
void OnSelectionChanged()
{
    Context.SelectedIndex = newIndex;
    Refresh(); // 全セルを更新
}
```

##### `Relayout()`
```csharp
protected virtual void Relayout()
```
セルのレイアウトのみを強制的に更新します(内容は更新しない)。

---

### FancyCell<TItemData, TContext>

セルの基底クラス。

#### プロパティ

| プロパティ | 型 | 説明 |
|-----------|-----|------|
| `Index` | `int` | セルのインデックス |
| `IsVisible` | `bool` | セルが表示中か |
| `Context` | `TContext` | ScrollView と共有するコンテキスト |

#### メソッド

##### `Initialize()`
```csharp
public virtual void Initialize()
```
セルの初期化処理。セル生成時に1回だけ呼ばれます。

**使用例**:
```csharp
public override void Initialize()
{
    button.onClick.AddListener(() => Context.OnCellClicked?.Invoke(Index));
}
```

##### `UpdateContent(TItemData)`
```csharp
public abstract void UpdateContent(TItemData itemData)
```
データが更新された時に呼ばれます。UI にデータをバインドします。

**パラメータ**:
- `itemData`: 表示するデータ

**使用例**:
```csharp
public override void UpdateContent(ItemData itemData)
{
    messageText.text = itemData.Message;
    iconImage.sprite = itemData.Icon;
}
```

##### `UpdatePosition(float)`
```csharp
public virtual void UpdatePosition(float position)
```
スクロール位置が変わった時に呼ばれます。アニメーション制御に使用します。

**パラメータ**:
- `position`: 正規化された位置 (0.0-1.0)

**使用例**:
```csharp
public override void UpdatePosition(float position)
{
    var distance = Mathf.Abs(0.5f - position);
    var scale = Mathf.Lerp(1f, 0.7f, distance * 2f);
    transform.localScale = Vector3.one * scale;
}
```

---

## Scroller API

### Scroller

スクロール位置の制御とユーザー入力の処理を担当します。

#### プロパティ

| プロパティ | 型 | 説明 |
|-----------|-----|------|
| `Position` | `float` | 現在のスクロール位置 |
| `Velocity` | `float` | 現在の速度 |
| `ScrollDirection` | `ScrollDirection` | スクロール方向 (Vertical/Horizontal) |
| `MovementType` | `MovementType` | 移動タイプ (Elastic/Clamped/Unrestricted) |
| `Elasticity` | `float` | 弾性係数 (0.0-1.0) |
| `ScrollSensitivity` | `float` | スクロール感度 |
| `Inertia` | `bool` | 慣性の有効/無効 |
| `DecelerationRate` | `float` | 減衰率 (0.0-1.0) |
| `Snap` | `SnapSettings` | スナップ設定 |
| `Draggable` | `bool` | ドラッグ可能か |

#### メソッド

##### `OnValueChanged(Action<float>)`
```csharp
public void OnValueChanged(Action<float> callback)
```
スクロール位置が変わった時のコールバックを設定します。

**パラメータ**:
- `callback`: スクロール位置を受け取るコールバック

**使用例**:
```csharp
scroller.OnValueChanged(UpdatePosition);
```

##### `OnSelectionChanged(Action<int>)`
```csharp
public void OnSelectionChanged(Action<int> callback)
```
選択インデックスが変わった時のコールバックを設定します。

**パラメータ**:
- `callback`: インデックスを受け取るコールバック

**使用例**:
```csharp
scroller.OnSelectionChanged(index => Debug.Log($"Selected: {index}"));
```

##### `SetTotalCount(int)`
```csharp
public void SetTotalCount(int totalCount)
```
アイテムの総数を設定します。

**パラメータ**:
- `totalCount`: アイテムの総数

**使用例**:
```csharp
scroller.SetTotalCount(items.Count);
```

##### `ScrollTo(float, float, Ease, Action)`
```csharp
public void ScrollTo(float position, float duration, Ease easing = Ease.Linear, Action onComplete = null)
```
指定位置にアニメーション付きでスクロールします。

**パラメータ**:
- `position`: 目標位置
- `duration`: アニメーション時間(秒)
- `easing`: イージング関数 (オプション)
- `onComplete`: 完了時のコールバック (オプション)

**使用例**:
```csharp
scroller.ScrollTo(5, 0.35f, Ease.OutCubic, () => {
    Debug.Log("スクロール完了");
});
```

##### `JumpTo(float)`
```csharp
public void JumpTo(float position)
```
即座に指定位置にジャンプします(アニメーションなし)。

**パラメータ**:
- `position`: 目標位置

**使用例**:
```csharp
scroller.JumpTo(0); // 先頭にジャンプ
```

---

### MovementType (列挙型)

```csharp
public enum MovementType
{
    Elastic,       // 弾性: 端で跳ね返る
    Clamped,       // 固定: 端で停止
    Unrestricted   // 無制限: 制限なし(無限スクロール用)
}
```

### ScrollDirection (列挙型)

```csharp
public enum ScrollDirection
{
    Vertical,      // 縦スクロール
    Horizontal     // 横スクロール
}
```

### SnapSettings (構造体)

スナップ設定。

```csharp
public class SnapSettings
{
    public bool Enable;              // スナップの有効/無効
    public float VelocityThreshold;  // スナップする速度の閾値
    public float Duration;           // スナップアニメーションの時間
    public Ease Easing;              // イージング関数
}
```

**使用例**:
```csharp
scroller.Snap.Enable = true;
scroller.Snap.VelocityThreshold = 0.5f;
scroller.Snap.Duration = 0.3f;
scroller.Snap.Easing = Ease.OutCubic;
```

---

## ScrollRect API

### FancyScrollRect<TItemData, TContext>

ScrollRect 形式のスクロールビュー。

#### 抽象プロパティ

| プロパティ | 型 | 説明 |
|-----------|-----|------|
| `CellPrefab` | `GameObject` | セルの Prefab |
| `CellSize` | `float` | セルのサイズ(高さまたは幅) |

#### プロパティ

| プロパティ | 型 | デフォルト | 説明 |
|-----------|-----|-----------|------|
| `Spacing` | `float` | 0 | セル間のスペース |
| `PaddingHead` | `float` | 0 | リスト先頭のパディング |
| `PaddingTail` | `float` | 0 | リスト末尾のパディング |
| `ReuseCellMarginCount` | `int` | 0 | 表示範囲外に保持するセル数 |

#### メソッド

##### `GetCellSize(int)`
```csharp
protected virtual float GetCellSize(int index)
```
指定インデックスのセルサイズを返します。可変サイズセルに使用します。

**パラメータ**:
- `index`: セルのインデックス

**デフォルト**: `CellSize` プロパティの値を返す

**使用例**:
```csharp
protected override float GetCellSize(int index)
{
    return ItemsSource[index].IsLarge ? 150f : 100f;
}
```

---

### FancyScrollRectCell<TItemData, TContext>

ScrollRect 用のセル基底クラス。

`FancyCell` とほぼ同じですが、`UpdatePosition(float)` メソッドがありません。

---

## GridView API

### FancyGridView<TItemData, TContext>

グリッドレイアウトのスクロールビュー。`FancyScrollRect` を拡張。

#### プロパティ

| プロパティ | 型 | 説明 |
|-----------|-----|------|
| `cellSize` | `Vector2` | セルのサイズ (幅, 高さ) |
| `startAxisCellCount` | `int` | 交差軸のセル数(列数/行数) |
| `startAxisSpacing` | `float` | 交差軸のスペーシング |

**使用例**:
```csharp
cellSize = new Vector2(150f, 150f);
startAxisCellCount = 3;  // 3列グリッド
startAxisSpacing = 10f;
```

---

## Easing API

### Ease (列挙型)

イージング関数の種類。

```csharp
public enum Ease
{
    Linear,
    InQuad, OutQuad, InOutQuad,
    InCubic, OutCubic, InOutCubic,
    InQuart, OutQuart, InOutQuart,
    InQuint, OutQuint, InOutQuint,
    InSine, OutSine, InOutSine,
    InExpo, OutExpo, InOutExpo,
    InCirc, OutCirc, InOutCirc,
    InElastic, OutElastic, InOutElastic,
    InBack, OutBack, InOutBack,
    InBounce, OutBounce, InOutBounce
}
```

### Easing クラス

```csharp
public static class Easing
{
    public static Func<float, float> Get(Ease ease)
    {
        // イージング関数を取得
    }
}
```

**使用例**:
```csharp
var easingFunc = Easing.Get(Ease.OutCubic);
var easedValue = easingFunc(0.5f); // 0.0-1.0 → イージング適用後の値
```

---

## よく使うパターン

### 基本的なスクロールビューの実装

```csharp
public class MyScrollView : FancyScrollView<ItemData>
{
    [SerializeField] Scroller scroller;
    [SerializeField] GameObject cellPrefab;

    protected override GameObject CellPrefab => cellPrefab;

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

### Context を使った実装

```csharp
// Context 定義
public class MyContext
{
    public int SelectedIndex = -1;
    public Action<int> OnCellClicked;
}

// Cell 実装
public class MyCell : FancyCell<ItemData, MyContext>
{
    public override void Initialize()
    {
        button.onClick.AddListener(() => Context.OnCellClicked?.Invoke(Index));
    }

    public override void UpdateContent(ItemData itemData)
    {
        var isSelected = Context.SelectedIndex == Index;
        // 選択状態に応じて表示を変更
    }
}

// ScrollView 実装
public class MyScrollView : FancyScrollView<ItemData, MyContext>
{
    protected override void Initialize()
    {
        base.Initialize();
        Context.OnCellClicked = OnCellClicked;
    }

    void OnCellClicked(int index)
    {
        Context.SelectedIndex = index;
        Refresh();
        scroller.ScrollTo(index, 0.35f);
    }
}
```

### 無限スクロールの実装

```csharp
public class InfiniteScrollView : FancyScrollView<ItemData>
{
    void Awake()
    {
        loop = true; // 循環配置を有効化
    }

    protected override void Initialize()
    {
        base.Initialize();
        scroller.MovementType = MovementType.Unrestricted; // 無制限
        scroller.OnValueChanged(UpdatePosition);
    }
}
```

### ScrollRect 形式の実装

```csharp
public class MyScrollRect : FancyScrollRect<ItemData, MyContext>
{
    [SerializeField] Scroller scroller;
    [SerializeField] GameObject cellPrefab;

    protected override GameObject CellPrefab => cellPrefab;
    protected override float CellSize => 100f;

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

### GridView の実装

```csharp
public class MyGridView : FancyGridView<ItemData, MyContext>
{
    [SerializeField] Scroller scroller;
    [SerializeField] GameObject cellPrefab;

    protected override GameObject CellPrefab => cellPrefab;

    void Awake()
    {
        cellSize = new Vector2(150f, 150f);
        startAxisCellCount = 3; // 3列
        spacing = 10f;
        startAxisSpacing = 10f;
    }

    protected override void Initialize()
    {
        base.Initialize();
        scroller.OnValueChanged(UpdatePosition);
    }
}
```

---

## まとめ

主要なクラスとメソッド:

- **FancyScrollView**: スクロールビューの基底クラス
  - `UpdateContents()`: データ設定
  - `Refresh()`: 表示更新
  - `UpdatePosition()`: スクロール位置更新

- **FancyCell**: セルの基底クラス
  - `Initialize()`: 初期化
  - `UpdateContent()`: データバインディング
  - `UpdatePosition()`: アニメーション制御

- **Scroller**: スクロール制御
  - `ScrollTo()`: アニメーション付きスクロール
  - `JumpTo()`: 即座に移動
  - `SetTotalCount()`: アイテム数設定

詳細な API 仕様は[公式ドキュメント](https://setchi.jp/FancyScrollView/api/FancyScrollView.html)を参照してください。
