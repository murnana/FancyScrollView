# FancyScrollView アーキテクチャ

このドキュメントでは、FancyScrollView の内部アーキテクチャと設計思想について詳しく説明します。

## 設計思想

### 核となるコンセプト

1. **柔軟性**: カスタムアニメーションやレイアウトを実装可能
2. **パフォーマンス**: セルプーリングによる効率的なレンダリング
3. **拡張性**: 抽象基底クラスを継承して機能を追加
4. **再利用性**: ジェネリクスによる型安全な実装

### アーキテクチャの階層

```
┌─────────────────────────────────────────┐
│        Application Layer                │
│  (ScrollView, Cell の具体実装)          │
└─────────────────────────────────────────┘
              ↓ 継承
┌─────────────────────────────────────────┐
│        Framework Layer                  │
│  - FancyScrollView<TItemData, TContext> │
│  - FancyCell<TItemData, TContext>       │
│  - FancyScrollRect                      │
│  - FancyGridView                        │
└─────────────────────────────────────────┘
              ↓ 使用
┌─────────────────────────────────────────┐
│        Core Components Layer            │
│  - Scroller                             │
│  - Draggable                            │
│  - Easing                               │
└─────────────────────────────────────────┘
              ↓ 基盤
┌─────────────────────────────────────────┐
│        Unity Engine Layer               │
│  - MonoBehaviour                        │
│  - EventSystems                         │
└─────────────────────────────────────────┘
```

## コアアーキテクチャ

### FancyScrollView の内部構造

```
FancyScrollView<TItemData, TContext>
│
├─ ItemsSource: IList<TItemData>  // データソース
├─ Context: TContext              // 共有コンテキスト
├─ pool: List<FancyCell>          // セルプール
│
├─ セルライフサイクル管理
│  ├─ ResizePool()       // プール拡張
│  ├─ UpdateCells()      // セル更新
│  └─ CircularIndex()    // 循環インデックス計算
│
└─ 公開API
   ├─ UpdateContents()   // データ更新
   ├─ Refresh()          // 強制再描画
   ├─ Relayout()         // レイアウト再計算
   └─ UpdatePosition()   // スクロール位置更新
```

### セルプーリングのメカニズム

#### プールサイズの動的調整

```csharp
void ResizePool(float firstPosition)
{
    // 必要なセル数を計算
    var requiredCells = Mathf.CeilToInt((1f - firstPosition) / cellInterval);
    var addCount = requiredCells - pool.Count;

    // 不足分を生成
    for (var i = 0; i < addCount; i++)
    {
        var cell = Instantiate(CellPrefab, cellContainer);
        cell.SetContext(Context);
        cell.Initialize();
        pool.Add(cell);
    }
}
```

**ポイント**:
- ビューポートサイズに応じて動的にプールサイズを調整
- 一度生成したセルは破棄せず再利用
- 必要最小限のセルのみ生成

#### セルの更新ロジック

```csharp
void UpdateCells(float firstPosition, int firstIndex, bool forceRefresh)
{
    for (var i = 0; i < pool.Count; i++)
    {
        var index = firstIndex + i;
        var position = firstPosition + i * cellInterval;
        var cell = pool[CircularIndex(index, pool.Count)];

        // 無限スクロール対応
        if (loop)
        {
            index = CircularIndex(index, ItemsSource.Count);
        }

        // 範囲外チェック
        if (index < 0 || index >= ItemsSource.Count || position > 1f)
        {
            cell.SetVisible(false);
            continue;
        }

        // セル更新
        if (forceRefresh || cell.Index != index || !cell.IsVisible)
        {
            cell.Index = index;
            cell.SetVisible(true);
            cell.UpdateContent(ItemsSource[index]);
        }

        cell.UpdatePosition(position);
    }
}
```

**ポイント**:
- プール内のセルを循環的に使用
- インデックスが変わった時のみ `UpdateContent` を呼ぶ
- 毎フレーム `UpdatePosition` を呼んでアニメーション更新

### 正規化座標系

FancyScrollView は独自の正規化座標系を使用:

```
position = scrollPosition - scrollOffset / cellInterval

firstIndex = Ceil(position)
firstPosition = (Ceil(position) - position) * cellInterval
```

**座標変換の例**:
- `scrollPosition = 0`, `scrollOffset = 0.5`, `cellInterval = 0.2`
  - `position = 0 - 0.5 / 0.2 = -2.5`
  - `firstIndex = Ceil(-2.5) = -2`
  - `firstPosition = (-2 - (-2.5)) * 0.2 = 0.1`

**結果**: インデックス -2 のセルが 0.1 の位置に配置(0.5 の位置に インデックス 0 が配置される)

## Scroller の内部構造

### 状態管理

```
Scroller
│
├─ 状態
│  ├─ Position: float           // 現在位置
│  ├─ Velocity: float           // 速度
│  ├─ Dragging: bool            // ドラッグ中か
│  └─ AutoScrollState           // 自動スクロール状態
│
├─ 設定
│  ├─ ScrollDirection           // スクロール方向
│  ├─ MovementType              // 移動タイプ
│  ├─ Elasticity                // 弾性係数
│  ├─ Inertia                   // 慣性
│  ├─ DecelerationRate          // 減衰率
│  └─ Snap                      // スナップ設定
│
└─ 機能
   ├─ ドラッグ処理 (Draggable)
   ├─ 慣性スクロール
   ├─ 弾性スクロール
   ├─ スナップ
   └─ 自動スクロール (ScrollTo)
```

### スクロールの更新ループ

```csharp
void LateUpdate()
{
    // ドラッグ中は何もしない
    if (dragging) return;

    // 慣性スクロール
    if (Inertia && velocity != 0f)
    {
        velocity *= Mathf.Pow(decelerationRate, Time.deltaTime);
        position += velocity * Time.deltaTime;
    }

    // 弾性処理(範囲外の場合)
    if (MovementType == MovementType.Elastic)
    {
        if (position < 0 || position > totalCount - 1)
        {
            // 弾性による引き戻し
            position = Mathf.Lerp(position, clampedPosition, elasticity);
        }
    }

    // スナップ処理
    if (Snap.Enable && ShouldSnap())
    {
        SnapToNearest();
    }

    // 自動スクロール
    UpdateAutoScroll();

    // スクロール位置を通知
    onValueChanged?.Invoke(position);
}
```

### MovementType の実装

#### Elastic (弾性)

```csharp
if (position < 0f)
{
    // 範囲外に出た分を弾性で引き戻す
    var overshoot = -position;
    position = -RubberDelta(overshoot, scrollSize);
}

float RubberDelta(float overStretch, float viewSize)
{
    return (1 - (1 / ((Mathf.Abs(overStretch) * 0.55f / viewSize) + 1))) * viewSize;
}
```

**特徴**:
- 端を超えてドラッグ可能
- 離すと元の範囲に戻る
- 「引っ張る」感覚を実現

#### Clamped (固定)

```csharp
position = Mathf.Clamp(position, 0f, totalCount - 1);
```

**特徴**:
- 端で完全停止
- 範囲外に出られない

#### Unrestricted (無制限)

```csharp
// 制限なし
```

**特徴**:
- 無限にスクロール可能
- 無限スクロールに必須

### スナップ機構

```csharp
void SnapToNearest()
{
    var targetIndex = Mathf.RoundToInt(position);

    if (Mathf.Abs(velocity) < Snap.VelocityThreshold)
    {
        // 自動スクロールでスナップ
        ScrollTo(targetIndex, Snap.Duration, Snap.Easing);
    }
}
```

**条件**:
1. `Snap.Enable = true`
2. 速度が閾値以下
3. ドラッグ終了後

### 自動スクロール (ScrollTo)

```csharp
public void ScrollTo(float position, float duration, Ease easing = Ease.Linear, Action onComplete = null)
{
    autoScrollState.Enable = true;
    autoScrollState.Duration = duration;
    autoScrollState.StartTime = Time.unscaledTime;
    autoScrollState.StartPosition = this.position;
    autoScrollState.EndPosition = position;
    autoScrollState.Easing = easing;
    autoScrollState.OnComplete = onComplete;
}

void UpdateAutoScroll()
{
    if (!autoScrollState.Enable) return;

    var elapsed = Time.unscaledTime - autoScrollState.StartTime;
    var progress = Mathf.Clamp01(elapsed / autoScrollState.Duration);

    // イージング適用
    var easedProgress = Easing.Get(autoScrollState.Easing)(progress);

    // 位置を補間
    position = Mathf.Lerp(
        autoScrollState.StartPosition,
        autoScrollState.EndPosition,
        easedProgress
    );

    if (progress >= 1f)
    {
        autoScrollState.Enable = false;
        autoScrollState.OnComplete?.Invoke();
    }
}
```

## ScrollRect / GridView アーキテクチャ

### 階層構造

```
FancyScrollRect<TItemData, TContext>
│
├─ セルサイズ管理
│  ├─ CellSize (abstract)
│  ├─ GetCellSize(index)
│  └─ CalculateTotalSize()
│
├─ レイアウト計算
│  ├─ PaddingHead
│  ├─ PaddingTail
│  ├─ Spacing
│  └─ GetCellPosition(index)
│
└─ セル再利用
   ├─ ReuseCellMarginCount
   ├─ CalculateVisibleRange()
   └─ UpdateCells()

FancyGridView<TItemData, TContext>
│  (extends FancyScrollRect)
│
└─ グリッド管理
   ├─ cellSize: Vector2
   ├─ startAxisCellCount
   ├─ startAxisSpacing
   ├─ CellGroupPrefab
   └─ UpdateCellGroups()
```

### セル再利用の仕組み

```csharp
void UpdateCells()
{
    // 表示範囲を計算
    var (startIndex, endIndex) = CalculateVisibleRange();

    // マージンを適用
    startIndex = Mathf.Max(0, startIndex - ReuseCellMarginCount);
    endIndex = Mathf.Min(ItemsSource.Count - 1, endIndex + ReuseCellMarginCount);

    // 範囲外のセルを非表示
    foreach (var cell in pool)
    {
        if (cell.Index < startIndex || cell.Index > endIndex)
        {
            cell.SetVisible(false);
        }
    }

    // 範囲内のセルを更新
    for (var i = startIndex; i <= endIndex; i++)
    {
        var cell = GetOrCreateCell();
        cell.Index = i;
        cell.SetVisible(true);
        cell.UpdateContent(ItemsSource[i]);

        // 位置を設定
        var position = GetCellPosition(i);
        cell.transform.anchoredPosition = position;
    }
}
```

### GridView のセルグループ

```
FancyGridView
│
└─ CellGroups (行/列のグループ)
   ├─ Group 0
   │  ├─ Cell 0
   │  ├─ Cell 1
   │  └─ Cell 2
   ├─ Group 1
   │  ├─ Cell 3
   │  ├─ Cell 4
   │  └─ Cell 5
   └─ ...
```

セルグループの管理:
```csharp
class FancyCellGroup<TItemData, TContext>
{
    List<FancyCell<TItemData, TContext>> cells;

    public void UpdateContents(IList<TItemData> items)
    {
        for (var i = 0; i < items.Count; i++)
        {
            cells[i].UpdateContent(items[i]);
        }
    }

    public void SetActive(bool active)
    {
        gameObject.SetActive(active);
    }
}
```

## Context パターン

### 設計意図

Context は以下の目的で導入されています:

1. **Cell → ScrollView の通信**: イベント通知
2. **ScrollView → Cell の状態共有**: 選択状態など
3. **疎結合**: Cell が ScrollView を直接参照しない

### データフロー

```
┌──────────────┐
│  ScrollView  │
│              │
│  Context ◄───┼───┐ 同じインスタンス
│              │   │
└──────┬───────┘   │
       │           │
       │ UpdateContents
       ↓           │
┌──────────────┐   │
│    Cell 0    │   │
│              │   │
│  Context ────┼───┘
└──────────────┘

User Action → Cell.OnClick → Context.Callback → ScrollView
                                                     ↓
                                            Context.State 更新
                                                     ↓
                                                 Refresh()
                                                     ↓
                                        全 Cell.UpdateContent()
```

### メモリ管理

Context は FancyScrollView で1回だけ生成:

```csharp
protected TContext Context { get; } = new TContext();
```

全てのセルが同じインスタンスを共有するため:
- メモリ効率が良い
- 状態の一貫性が保たれる

## パフォーマンス最適化

### セルプーリング

**最適化のポイント**:
1. セル生成は初回とプールサイズ変更時のみ
2. セルの Destroy はしない(再利用)
3. 表示/非表示の切り替えのみ

**GC Alloc の削減**:
```csharp
// Bad: 毎フレーム List を生成
var visibleCells = new List<Cell>();

// Good: 再利用可能なフィールド
readonly List<Cell> reusableCellList = new List<Cell>();
```

### UpdateContent vs UpdatePosition

```csharp
// UpdateContent: データが変わった時のみ
if (forceRefresh || cell.Index != index || !cell.IsVisible)
{
    cell.UpdateContent(ItemsSource[index]);
}

// UpdatePosition: 毎フレーム(アニメーション用)
cell.UpdatePosition(position);
```

**理由**:
- `UpdateContent`: 重い処理(テキスト設定、画像ロードなど)
- `UpdatePosition`: 軽い処理(Transform 更新のみ)

### レイアウト計算のキャッシュ

```csharp
// 計算結果をキャッシュ
float cachedTotalSize;

float GetTotalSize()
{
    if (isDirty)
    {
        cachedTotalSize = CalculateTotalSize();
        isDirty = false;
    }
    return cachedTotalSize;
}
```

## 拡張性

### カスタムスクロールビューの実装

```csharp
public class CustomScrollView<TItemData> : FancyScrollView<TItemData>
{
    // カスタムロジックを追加
    protected override void UpdatePosition(float position)
    {
        // 独自の位置計算
        base.UpdatePosition(TransformPosition(position));
    }

    float TransformPosition(float position)
    {
        // 位置を変換
        return position;
    }
}
```

### カスタムセルの実装

```csharp
public class CustomCell<TItemData, TContext> : FancyCell<TItemData, TContext>
{
    // 共通の拡張機能
    protected virtual void OnSelected()
    {
        // 選択時の処理
    }

    protected virtual void OnDeselected()
    {
        // 非選択時の処理
    }
}
```

## まとめ

FancyScrollView のアーキテクチャは:

1. **セルプーリング**: 効率的なメモリ管理
2. **正規化座標系**: 柔軟なアニメーション
3. **Context パターン**: 疎結合な通信
4. **抽象基底クラス**: 高い拡張性
5. **モジュール設計**: 機能ごとの分離

これらの設計により、パフォーマンスと柔軟性を両立しています。
