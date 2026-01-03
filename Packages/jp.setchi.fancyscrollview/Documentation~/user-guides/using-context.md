# Context の使い方

Context は、ScrollView と Cell 間でデータや状態を共有するための仕組みです。セルのクリックイベントの処理や、選択状態の管理などに使用します。

## Context が必要なケース

Context は以下のような場合に使用します:

- **Cell → ScrollView への通知**: セルがクリックされた時にScrollViewに通知
- **ScrollView → Cell への状態共有**: 現在選択されているインデックスなどの状態をセルに伝える
- **Cell 間の情報共有**: 全セルで共有する情報(設定、状態など)

## 基本的な実装

### ステップ1: Context クラスの定義

```csharp
public class Context
{
    public int SelectedIndex = -1;
    public Action<int> OnCellClicked;
}
```

- `SelectedIndex`: 現在選択されているセルのインデックス
- `OnCellClicked`: セルがクリックされた時のコールバック

### ステップ2: Cell の実装

`FancyCell<TItemData, TContext>` を使用します(Context の型を指定)。

```csharp
using UnityEngine;
using UnityEngine.UI;
using FancyScrollView;

public class Cell : FancyCell<ItemData, Context>
{
    [SerializeField] Animator animator;
    [SerializeField] Text message;
    [SerializeField] Image image;
    [SerializeField] Button button;

    static class AnimatorHash
    {
        public static readonly int Scroll = Animator.StringToHash("scroll");
    }

    public override void Initialize()
    {
        // ボタンのクリックイベントを設定
        button.onClick.AddListener(() => Context.OnCellClicked?.Invoke(Index));
    }

    public override void UpdateContent(ItemData itemData)
    {
        message.text = itemData.Message;

        // 選択状態に応じて表示を変更
        var isSelected = Context.SelectedIndex == Index;
        image.color = isSelected ? Color.cyan : Color.white;
    }

    public override void UpdatePosition(float position)
    {
        currentPosition = position;

        if (animator.isActiveAndEnabled)
        {
            animator.Play(AnimatorHash.Scroll, -1, position);
        }

        animator.speed = 0;
    }

    float currentPosition = 0;

    void OnEnable() => UpdatePosition(currentPosition);
}
```

### ステップ3: ScrollView の実装

`FancyScrollView<TItemData, TContext>` を使用します。

```csharp
using UnityEngine;
using System.Collections.Generic;
using FancyScrollView;

public class ScrollView : FancyScrollView<ItemData, Context>
{
    [SerializeField] Scroller scroller;
    [SerializeField] GameObject cellPrefab;

    protected override GameObject CellPrefab => cellPrefab;

    protected override void Initialize()
    {
        base.Initialize();

        // Context のコールバックを設定
        Context.OnCellClicked = OnCellClicked;

        scroller.OnValueChanged(UpdatePosition);
    }

    void OnCellClicked(int index)
    {
        // セルがクリックされた時の処理
        Context.SelectedIndex = index;

        // セルの表示を更新(全セルの UpdateContent が呼ばれる)
        Refresh();

        // クリックされたセルにフォーカス
        scroller.ScrollTo(index, 0.35f);
    }

    public void UpdateData(IList<ItemData> items)
    {
        UpdateContents(items);
        scroller.SetTotalCount(items.Count);
    }
}
```

## Context の仕組み

### 同一インスタンスの共有

Context のインスタンスは `FancyScrollView` で生成され、全ての Cell に同じインスタンスが渡されます:

```csharp
// FancyScrollView 内部
protected TContext Context { get; } = new TContext();

// Cell に Context を設定
cell.SetContext(Context);
```

これにより、ScrollView と全ての Cell が同じ Context オブジェクトを参照します。

### データフロー

```
User Action (Cell クリック)
    ↓
Cell.button.onClick
    ↓
Context.OnCellClicked?.Invoke(Index)
    ↓
ScrollView.OnCellClicked(int index)
    ↓
Context.SelectedIndex = index
    ↓
ScrollView.Refresh()
    ↓
全 Cell.UpdateContent() が呼ばれる
    ↓
各 Cell が Context.SelectedIndex を参照して表示を更新
```

## 実践的な例

### 例1: セル選択とフォーカス

セルをクリックすると選択状態になり、そのセルにスクロール:

```csharp
public class Context
{
    public int SelectedIndex = -1;
    public Action<int> OnCellClicked;
}

// ScrollView
void OnCellClicked(int index)
{
    Context.SelectedIndex = index;
    Refresh();
    scroller.ScrollTo(index, 0.35f, Ease.OutCubic, () =>
    {
        Debug.Log($"Scrolled to {index}");
    });
}

// Cell
public override void UpdateContent(ItemData itemData)
{
    var isSelected = Context.SelectedIndex == Index;
    background.color = isSelected
        ? new Color(0, 1, 1, 0.4f)  // 選択: シアン
        : new Color(1, 1, 1, 0.4f); // 非選択: 白
}
```

### 例2: 編集モードの管理

全セルで編集モードのON/OFFを共有:

```csharp
public class Context
{
    public bool IsEditMode = false;
    public Action<int> OnCellClicked;
    public Action<int> OnDeleteClicked;
}

// ScrollView
public void SetEditMode(bool enabled)
{
    Context.IsEditMode = enabled;
    Refresh(); // 全セルの表示を更新
}

void OnDeleteClicked(int index)
{
    // アイテムを削除
    items.RemoveAt(index);
    UpdateData(items);
}

// Cell
public override void Initialize()
{
    button.onClick.AddListener(() => Context.OnCellClicked?.Invoke(Index));
    deleteButton.onClick.AddListener(() => Context.OnDeleteClicked?.Invoke(Index));
}

public override void UpdateContent(ItemData itemData)
{
    message.text = itemData.Message;

    // 編集モードでのみ削除ボタンを表示
    deleteButton.gameObject.SetActive(Context.IsEditMode);
}
```

### 例3: 複数選択

複数のセルを選択可能にする:

```csharp
public class Context
{
    public HashSet<int> SelectedIndices = new HashSet<int>();
    public Action<int> OnCellClicked;
}

// ScrollView
void OnCellClicked(int index)
{
    if (Context.SelectedIndices.Contains(index))
    {
        Context.SelectedIndices.Remove(index);
    }
    else
    {
        Context.SelectedIndices.Add(index);
    }

    Refresh();
}

// Cell
public override void UpdateContent(ItemData itemData)
{
    message.text = itemData.Message;

    var isSelected = Context.SelectedIndices.Contains(Index);
    checkmark.gameObject.SetActive(isSelected);
}
```

## Context を使わない場合

Context が不要な場合は、シンプルな `FancyScrollView<TItemData>` を使用します:

```csharp
// Context なし
public class Cell : FancyCell<ItemData>
{
    // Context にアクセスできない
}

public class ScrollView : FancyScrollView<ItemData>
{
    // Context にアクセスできない
}
```

この場合、以下の制限があります:
- Cell から ScrollView へのコールバックができない
- Cell 間で状態を共有できない

静的な表示のみで、ユーザーインタラクションが不要な場合は Context なしで十分です。

## ベストプラクティス

### 1. Context は薄く保つ

Context には必要最小限の情報のみを含めます:

```csharp
// Good: 必要最小限
public class Context
{
    public int SelectedIndex;
    public Action<int> OnCellClicked;
}

// Bad: ScrollView の責務を Context に詰め込みすぎ
public class Context
{
    public int SelectedIndex;
    public List<ItemData> Items; // ScrollView が管理すべき
    public Scroller Scroller;    // ScrollView が管理すべき
    public Action<int> OnCellClicked;
}
```

### 2. イベントには Action デリゲートを使う

```csharp
// Good: シンプルで型安全
public Action<int> OnCellClicked;

// 使用
Context.OnCellClicked?.Invoke(Index);
```

### 3. Initialize で一度だけ設定

```csharp
public override void Initialize()
{
    // イベントハンドラは Initialize で一度だけ設定
    button.onClick.AddListener(() => Context.OnCellClicked?.Invoke(Index));
}

// Bad: UpdateContent で毎回設定すると重複登録される
public override void UpdateContent(ItemData itemData)
{
    button.onClick.AddListener(() => Context.OnCellClicked?.Invoke(Index)); // NG!
}
```

### 4. Refresh() を適切に呼ぶ

Context の状態が変わったら `Refresh()` を呼んで全セルの表示を更新します:

```csharp
void OnSomeStateChanged()
{
    Context.SomeState = newValue;
    Refresh(); // 全セルの UpdateContent が呼ばれる
}
```

## まとめ

- **Context** は ScrollView と Cell 間のデータ共有に使用
- **`FancyCell<TItemData, TContext>`** と **`FancyScrollView<TItemData, TContext>`** で Context を利用
- Cell → ScrollView の通知には **Action デリゲート** を使用
- ScrollView → Cell の状態共有には **Context のプロパティ** を使用
- 状態変更後は **`Refresh()`** で全セルを更新

次のステップ:
- [無限スクロール](infinite-scroll.md) - ループするスクロールビュー
- [ScrollRect 形式](scrollrect-usage.md) - 従来型リストの実装
