# カスタムアニメーション

FancyScrollView の最大の特徴は、`UpdatePosition(float position)` メソッドを使った柔軟なアニメーション実装です。このガイドでは、様々なアニメーション技法を紹介します。

**注意**: このガイドは `FancyScrollView` を対象としています。`FancyScrollRect` と `FancyGridView` ではカスタムアニメーションは使用できません。

## 基本: position パラメータ

`UpdatePosition(float position)` の `position` は、セルのビューポート内での正規化された位置です:

```
0.0 ← 画面外(上/左)
     ↓
0.5 ← 画面中央
     ↓
1.0 ← 画面外(下/右)
```

この値を使ってセルの見た目を制御します。

## アニメーションパターン集

### 1. スケールアニメーション

中央に近いセルを大きく表示:

```csharp
public override void UpdatePosition(float position)
{
    // 中央(0.5)からの距離を計算
    var distance = Mathf.Abs(0.5f - position);

    // 距離に応じてスケールを変更(0.5 ～ 1.0)
    var scale = Mathf.Lerp(1f, 0.5f, distance * 2f);
    transform.localScale = Vector3.one * scale;
}
```

### 2. フェードアニメーション

中央に近いほど不透明に:

```csharp
[SerializeField] CanvasGroup canvasGroup;

public override void UpdatePosition(float position)
{
    var distance = Mathf.Abs(0.5f - position);

    // 距離に応じて透明度を変更(0.0 ～ 1.0)
    canvasGroup.alpha = Mathf.Lerp(1f, 0f, distance * 2f);
}
```

### 3. 回転アニメーション

位置に応じて回転:

```csharp
public override void UpdatePosition(float position)
{
    // -1.0 ～ 1.0 に変換
    var normalizedPosition = (position - 0.5f) * 2f;

    // -30度 ～ 30度 回転
    var angle = normalizedPosition * 30f;
    transform.localRotation = Quaternion.Euler(0f, 0f, angle);
}
```

### 4. カーブを使った奥行き表現

Z軸の移動で奥行きを表現:

```csharp
public override void UpdatePosition(float position)
{
    var distance = Mathf.Abs(0.5f - position);

    // 中央から離れるほど奥に配置
    var z = -distance * 100f;
    transform.localPosition = new Vector3(
        transform.localPosition.x,
        transform.localPosition.y,
        z
    );

    // 奥のセルは小さく
    var scale = Mathf.Lerp(1f, 0.7f, distance * 2f);
    transform.localScale = Vector3.one * scale;
}
```

### 5. カルーセル(円形配置)

セルを円形に配置:

```csharp
public override void UpdatePosition(float position)
{
    var angle = (position - 0.5f) * Mathf.PI; // -π/2 ～ π/2
    var radius = 500f;

    var x = Mathf.Sin(angle) * radius;
    var y = (Mathf.Cos(angle) - 1f) * radius;

    transform.localPosition = new Vector3(x, y, 0f);

    // 回転も追加
    transform.localRotation = Quaternion.Euler(0f, 0f, -angle * Mathf.Rad2Deg);
}
```

### 6. イージング曲線を使った動き

```csharp
public override void UpdatePosition(float position)
{
    var distance = Mathf.Abs(0.5f - position);

    // イージング関数を適用
    var easedDistance = EaseOutCubic(distance * 2f);

    var scale = Mathf.Lerp(1f, 0.5f, easedDistance);
    transform.localScale = Vector3.one * scale;
}

float EaseOutCubic(float t)
{
    return 1f - Mathf.Pow(1f - t, 3f);
}
```

## Animator を使った高度なアニメーション

### セットアップ

1. Animator Controller を作成
2. Float パラメータ "scroll" を追加
3. Blend Tree を作成して、0.0 ～ 1.0 の範囲でアニメーションをブレンド

### Cell の実装

```csharp
[SerializeField] Animator animator;

static class AnimatorHash
{
    public static readonly int Scroll = Animator.StringToHash("scroll");
}

public override void UpdatePosition(float position)
{
    currentPosition = position;

    if (animator.isActiveAndEnabled)
    {
        // position (0.0-1.0) を Animator のパラメータとして渡す
        animator.Play(AnimatorHash.Scroll, -1, position);
    }

    // 手動制御のため速度を0に
    animator.speed = 0;
}

// GameObject が非アクティブになると Animator がリセットされるため保持
float currentPosition = 0;

void OnEnable()
{
    UpdatePosition(currentPosition);
}
```

### Blend Tree の構成例

```
Scroll (Float Parameter: 0.0 - 1.0)
├─ 0.0: OutOfView (画面外・上/左)
├─ 0.3: Entering (入ってくる)
├─ 0.5: Center (中央・フォーカス)
├─ 0.7: Leaving (出ていく)
└─ 1.0: OutOfView (画面外・下/右)
```

各アニメーションで以下を制御:
- Position
- Scale
- Rotation
- Color (Animator でマテリアルカラーを制御)

## 複合アニメーション

複数のエフェクトを組み合わせる:

```csharp
[SerializeField] CanvasGroup canvasGroup;
[SerializeField] RectTransform iconTransform;

public override void UpdatePosition(float position)
{
    var distance = Mathf.Abs(0.5f - position);
    var normalizedPos = (position - 0.5f) * 2f; // -1 ～ 1

    // 1. スケール
    var scale = Mathf.Lerp(1f, 0.7f, distance * 2f);
    transform.localScale = Vector3.one * scale;

    // 2. 透明度
    canvasGroup.alpha = Mathf.Lerp(1f, 0.3f, distance * 2f);

    // 3. 回転
    var angle = normalizedPos * 15f;
    transform.localRotation = Quaternion.Euler(0f, 0f, angle);

    // 4. アイコンの回転(逆回転でバランスを取る)
    iconTransform.localRotation = Quaternion.Euler(0f, 0f, -angle * 2f);

    // 5. Z軸で奥行き
    var z = -distance * 50f;
    transform.localPosition = new Vector3(
        transform.localPosition.x,
        transform.localPosition.y,
        z
    );
}
```

## シェーダーを使ったアニメーション

マテリアルプロパティを制御:

```csharp
[SerializeField] Material material;

public override void UpdatePosition(float position)
{
    var distance = Mathf.Abs(0.5f - position);

    // シェーダープロパティを更新
    material.SetFloat("_Blur", distance * 5f);
    material.SetFloat("_Brightness", Mathf.Lerp(1f, 0.5f, distance * 2f));

    // カラーグラデーション
    var color = Color.Lerp(Color.white, Color.gray, distance * 2f);
    material.SetColor("_Color", color);
}
```

### カスタムシェーダー例

```glsl
Shader "Custom/ScrollCell"
{
    Properties
    {
        _MainTex ("Texture", 2D) = "white" {}
        _Blur ("Blur", Float) = 0
        _Brightness ("Brightness", Float) = 1
        _Color ("Color", Color) = (1,1,1,1)
    }
    // ... シェーダーの実装
}
```

## パフォーマンス最適化

### 1. キャッシュを活用

```csharp
// Bad: 毎フレームGetComponentを呼ぶ
public override void UpdatePosition(float position)
{
    GetComponent<CanvasGroup>().alpha = ...;
}

// Good: SerializeField または Awake/Start でキャッシュ
[SerializeField] CanvasGroup canvasGroup;

public override void UpdatePosition(float position)
{
    canvasGroup.alpha = ...;
}
```

### 2. 不要な計算を避ける

```csharp
public override void UpdatePosition(float position)
{
    // 表示範囲外なら処理をスキップ
    if (position < -0.5f || position > 1.5f)
    {
        return;
    }

    // アニメーション処理
    var distance = Mathf.Abs(0.5f - position);
    // ...
}
```

### 3. Animator.speed の設定

```csharp
public override void UpdatePosition(float position)
{
    if (animator.isActiveAndEnabled)
    {
        animator.Play(AnimatorHash.Scroll, -1, position);
        animator.speed = 0; // 重要: 自動再生を停止
    }
}
```

### 4. アロケーションを避ける

```csharp
// Bad: 毎回 Vector3.one を作成
transform.localScale = Vector3.one * scale;

// Good: 再利用可能な変数
static readonly Vector3 BaseScale = Vector3.one;
transform.localScale = BaseScale * scale;
```

## 実践例: カードスタック風

```csharp
public class CardCell : FancyCell<CardData, Context>
{
    [SerializeField] CanvasGroup canvasGroup;
    [SerializeField] RectTransform cardTransform;

    const float StackSpacing = 20f;
    const float RotationAngle = 10f;
    const float ScaleMin = 0.8f;

    public override void UpdateContent(CardData itemData)
    {
        // カードのデータを表示
    }

    public override void UpdatePosition(float position)
    {
        var distance = Mathf.Abs(0.5f - position);
        var direction = position < 0.5f ? -1f : 1f;

        // スタック効果: 中央から離れるほど下に配置
        var yOffset = -distance * StackSpacing * 5f;

        // 軽い回転
        var rotation = direction * distance * RotationAngle;

        // スケール
        var scale = Mathf.Lerp(1f, ScaleMin, distance * 2f);

        // 透明度
        var alpha = Mathf.Lerp(1f, 0.3f, Mathf.Clamp01(distance * 3f));

        // 適用
        cardTransform.anchoredPosition = new Vector2(0f, yOffset);
        cardTransform.localRotation = Quaternion.Euler(0f, 0f, rotation);
        cardTransform.localScale = Vector3.one * scale;
        canvasGroup.alpha = alpha;

        // Z オーダー(重なり順)
        cardTransform.SetSiblingIndex(distance < 0.5f ? 100 : 0);
    }
}
```

## トラブルシューティング

### 問題: アニメーションが止まっている

**原因**: `Animator.speed = 0` の設定忘れ

**解決策**:
```csharp
animator.Play(AnimatorHash.Scroll, -1, position);
animator.speed = 0; // この行を追加
```

### 問題: GameObject が非アクティブ化されるとアニメーションがリセット

**原因**: Animator は非アクティブ時にリセットされる

**解決策**: 現在位置を保持して `OnEnable` で復元
```csharp
float currentPosition = 0;

public override void UpdatePosition(float position)
{
    currentPosition = position; // 保存
    // ...
}

void OnEnable()
{
    UpdatePosition(currentPosition); // 復元
}
```

### 問題: アニメーションがカクつく

**原因**: 重い処理や不要な GetComponent

**解決策**: コンポーネントをキャッシュし、計算を最適化

## まとめ

FancyScrollView のカスタムアニメーションは:

- **`UpdatePosition(float position)`** で実装
- **position**: 0.0(画面外上/左) ～ 1.0(画面外下/右)
- **組み合わせ可能**: Scale, Rotation, Position, Alpha, Color など
- **Animator 対応**: Blend Tree で高度なアニメーション
- **パフォーマンス**: キャッシュと最適化を意識

この柔軟性により、UI/UXに合わせた独自のスクロールビューを実装できます。

次のステップ:
- [Examples](../../../Assets/FancyScrollView/Examples/) でサンプルを確認
- デモサイト: https://setchi.jp/FancyScrollView/demo
