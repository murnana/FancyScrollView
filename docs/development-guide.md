# FancyScrollView 開発ガイド

このドキュメントは、FancyScrollView のコードベースの開発・メンテナンスを行うための開発者向けガイドです。

## プロジェクト概要

### 基本情報

- **プロジェクト名**: FancyScrollView
- **種別**: Unity UPM パッケージ
- **Unity バージョン**: 6.0.64f1 以降
- **ライセンス**: MIT (Copyright (c) 2020 setchi)
- **言語**: C# (.NET 4.x)

### リポジトリ構造

```
FancyScrollView/
├── Packages/
│   └── jp.setchi.fancyscrollview/        # UPM パッケージ本体
│       ├── package.json                  # パッケージマニフェスト
│       ├── Runtime/                      # ランタイムコード
│       │   ├── Core/                     # コア機能
│       │   ├── Scroller/                 # スクロール制御
│       │   ├── ScrollRect/               # ScrollRect形式
│       │   ├── GridView/                 # GridView形式
│       │   └── FancyScrollView.asmdef    # ランタイムアセンブリ定義
│       ├── Editor/                       # エディタ拡張
│       │   └── FancyScrollView.Editor.asmdef
│       └── Documentation~/               # ドキュメント
│           └── modifications/            # 変更履歴・UPM関連
├── Assets/
│   └── FancyScrollView/
│       └── Examples/                     # サンプルシーン・コード
│           ├── Scenes/                   # シーン (01-09)
│           └── Sources/                  # サンプルコード
└── docs/                                 # 開発者向けドキュメント
    ├── branch-strategy.md                # ブランチ戦略
    └── development-guide.md              # このファイル
```

## 開発環境セットアップ

### 必要なツール

1. **Unity Hub** (最新版)
2. **Unity 6.0.64f1** 以降
3. **Git**
4. **適切なコードエディタ**
   - Visual Studio 2022 (推奨)
   - Visual Studio Code
   - JetBrains Rider

### プロジェクトのセットアップ

```bash
# リポジトリのクローン
git clone <repository-url>
cd FancyScrollView

# Unity Hub でプロジェクトを開く
# - Unity Hub を開く
# - 「Add」→ プロジェクトディレクトリを選択
# - Unity 6.0.64f1 を選択してプロジェクトを開く
```

### 開発ブランチの運用

このプロジェクトは2つの主要ブランチで運用されます:

- **`master`**: アップストリーム(setchi/FancyScrollView)との同期用
  - クリーンに保つ
  - アップストリームの更新を受け入れる
  - 直接開発は行わない

- **`develop`**: メイン開発ブランチ
  - 日常的な開発作業はこちらで行う
  - カスタム変更を含む
  - `master` からのマージを受ける

詳細は [branch-strategy.md](branch-strategy.md) を参照してください。

## コードベース構造

### Core モジュール (`Runtime/Core/`)

スクロールビューのコア機能を提供。

#### 主要クラス

**`FancyScrollView<TItemData, TContext>`**
- スクロールビューの基底クラス
- セルプーリング、無限スクロール、スナップ対応
- 抽象クラス: 継承して使用

重要なメソッド:
- `Initialize()`: 初期化処理
- `UpdateContents(IList<TItemData>)`: データ更新
- `UpdatePosition(float position)`: スクロール位置更新
- `Refresh()`: 強制再描画
- `Relayout()`: レイアウト再計算

**`FancyCell<TItemData, TContext>`**
- セルの基底クラス
- データバインディングとアニメーション制御
- 抽象クラス: 継承して使用

重要なメソッド:
- `Initialize()`: セル初期化(1回のみ)
- `UpdateContent(TItemData)`: データ表示の更新
- `UpdatePosition(float)`: 位置ベースのアニメーション
- `SetVisible(bool)`: 表示/非表示制御

### Scroller モジュール (`Runtime/Scroller/`)

スクロール位置の制御とユーザー入力の処理。

#### 主要クラス

**`Scroller`**
- スクロール入力の処理
- 慣性、減衰、スナップの実装
- `Draggable` を使ったドラッグ処理

主要プロパティ:
- `Position`: 現在のスクロール位置
- `ScrollDirection`: スクロール方向(Vertical/Horizontal)
- `MovementType`: 移動タイプ(Elastic/Clamped/Unrestricted)
- `Snap`: スナップ設定

主要メソッド:
- `ScrollTo(position, duration, easing, onComplete)`: アニメーション付きスクロール
- `JumpTo(position)`: 即座に移動
- `SetTotalCount(count)`: アイテム総数の設定

**`Draggable`**
- UI のドラッグ処理を実装
- `IBeginDragHandler`, `IDragHandler`, `IEndDragHandler` を実装

### ScrollRect モジュール (`Runtime/ScrollRect/`)

従来型のスクロールリスト実装。

**`FancyScrollRect<TItemData, TContext>`**
- ScrollRect 風のスクロールビュー
- 無限スクロール・スナップは非対応
- セルの再利用に対応

重要なプロパティ:
- `CellSize`: セルのサイズ(必須オーバーライド)
- `Spacing`: セル間隔
- `PaddingHead/Tail`: リスト両端のパディング

**`FancyScrollRectCell<TItemData, TContext>`**
- ScrollRect 用のセル基底クラス
- `UpdatePosition()` なし

### GridView モジュール (`Runtime/GridView/`)

グリッドレイアウトの実装。

**`FancyGridView<TItemData, TContext>`**
- `FancyScrollRect` を拡張
- グリッドレイアウトに対応

重要なプロパティ:
- `cellSize`: セルサイズ (Vector2)
- `startAxisCellCount`: 交差軸のセル数(列数/行数)
- `startAxisSpacing`: 交差軸のスペーシング

**`FancyCellGroup<TItemData, TContext>`**
- セルのグループ管理
- 行/列単位での管理

## コーディング規約

### 命名規則

```csharp
// クラス: PascalCase
public class FancyScrollView { }

// メソッド: PascalCase
public void UpdatePosition() { }

// プロパティ: PascalCase
public float CellSize { get; }

// フィールド(private): camelCase
float currentPosition;

// フィールド(SerializeField): camelCase
[SerializeField] GameObject cellPrefab;

// 定数: PascalCase
const float DefaultCellSize = 100f;

// 静的読み取り専用: PascalCase
static readonly int ScrollHash = Animator.StringToHash("scroll");
```

### コメント

- **日本語コメント**: コードベースのコメントは日本語で記述
- **XMLドキュメントコメント**: パブリックAPI には必須

```csharp
/// <summary>
/// スクロール位置を更新します.
/// </summary>
/// <param name="position">スクロール位置.</param>
public void UpdatePosition(float position)
{
    // 内部処理のコメントも日本語
}
```

### ファイル構造

```csharp
/*
 * FancyScrollView (https://github.com/setchi/FancyScrollView)
 * Copyright (c) 2020 setchi
 * Licensed under MIT (https://github.com/setchi/FancyScrollView/blob/master/LICENSE)
 */

using UnityEngine;
using System.Collections.Generic;

namespace FancyScrollView
{
    /// <summary>
    /// クラスの説明
    /// </summary>
    public class ClassName
    {
        // コード
    }
}
```

## テストとデバッグ

### サンプルシーンの活用

開発時は `Assets/FancyScrollView/Examples/` のサンプルシーンを使用:

1. 変更を加える
2. 該当するサンプルシーンで動作確認
3. 複数のサンプルシーンで回帰テストを実施

### Unity Test Framework

```bash
# Test Runner を開く
Window > General > Test Runner
```

テストの追加:
```csharp
using NUnit.Framework;
using FancyScrollView;

public class FancyScrollViewTests
{
    [Test]
    public void TestCellPooling()
    {
        // テストコード
    }
}
```

### デバッグのヒント

#### Editor での動作確認

`LateUpdate()` で Inspector の値変更を監視:

```csharp
#if UNITY_EDITOR
void LateUpdate()
{
    if (cachedLoop != loop)
    {
        cachedLoop = loop;
        UpdatePosition(currentPosition);
    }
}
#endif
```

#### ログ出力

```csharp
Debug.Log($"Position: {position}, Index: {index}");
Debug.Assert(condition, "エラーメッセージ");
```

## ビルドとパッケージング

### UPM パッケージの構造

`package.json` の重要フィールド:

```json
{
  "name": "jp.setchi.fancyscrollview",
  "version": "2.0.0",
  "displayName": "FancyScrollView",
  "description": "...",
  "unity": "6.0",
  "dependencies": {},
  "author": {
    "name": "setchi",
    "url": "https://github.com/setchi/FancyScrollView"
  }
}
```

### バージョニング

セマンティックバージョニング (SemVer) に従う:

- **Major (X.0.0)**: 破壊的変更
- **Minor (0.X.0)**: 新機能(後方互換性あり)
- **Patch (0.0.X)**: バグフィックス

バージョン更新時:
1. `package.json` の `version` を更新
2. CHANGELOG を更新(存在する場合)
3. Git タグを作成

```bash
git tag v2.1.0
git push origin v2.1.0
```

## よくある開発タスク

### 新しいセルタイプの追加

1. `FancyCell<TItemData, TContext>` を継承
2. `UpdateContent()` と `UpdatePosition()` を実装
3. サンプルシーンで動作確認

### 新しいスクロールビュータイプの追加

1. `FancyScrollView<TItemData, TContext>` を継承
2. `CellPrefab` プロパティを実装
3. `Initialize()` で Scroller と連携
4. サンプルシーンとドキュメントを追加

### Scroller の機能拡張

1. `Scroller.cs` を編集
2. 新しいプロパティ/メソッドを追加
3. `ScrollerEditor.cs` で Inspector 対応
4. 既存のサンプルで回帰テストを実施

## パフォーマンス考慮事項

### セルプーリングの最適化

- 可視領域のセルのみ生成
- セルの再利用で GC Alloc を削減
- `ResizePool()` は必要な時のみ呼ぶ

### アロケーションの削減

```csharp
// Bad: 毎フレームアロケーション
var list = new List<int>();

// Good: 再利用可能なフィールド
readonly List<int> reusableList = new List<int>();
```

### レイアウト計算の最適化

- 不要な `Refresh()` 呼び出しを避ける
- `Relayout()` は必要な時のみ
- エディタでの変更検知は `#if UNITY_EDITOR` で囲む

## アップストリームとの同期

### アップストリームからの更新取得

```bash
# アップストリームを追加(初回のみ)
git remote add upstream https://github.com/setchi/FancyScrollView.git

# アップストリームから fetch
git fetch upstream

# master に切り替えて merge
git checkout master
git merge upstream/master

# master を develop にマージ
git checkout develop
git merge master
```

詳細は [branch-strategy.md](branch-strategy.md) を参照。

### コンフリクト解決

コンフリクトが発生した場合:

1. 自動マージを試みる
2. コンフリクトしたファイルを手動で編集
3. テストを実施
4. コミット

## ドキュメントの更新

### ドキュメント構造

- **`Documentation~/`**: UPM パッケージのドキュメント(ユーザー向け使い方ガイド)
- **`docs/`**: 開発者向けドキュメント(開発ガイド、ブランチ戦略など)
- **`README.md`**: パッケージの概要

### ドキュメント更新のタイミング

- 新機能追加時: 使い方ガイドを追加
- API 変更時: ドキュメントを更新
- バグフィックス時: 必要に応じて注意事項を追加

## ライセンスとクレジット

### ライセンス

このプロジェクトは MIT ライセンス下にあります:

```
Copyright (c) 2020 setchi
```

**重要**:
- オリジナルの `LICENSE` ファイルは保持する必要があります
- カスタム変更を加えても、ライセンスは変更できません
- フォークであることを明記してください

### 貢献

- アップストリームへの貢献は setchi/FancyScrollView へ PR を送る
- フォーク固有の機能は `develop` ブランチで開発

## トラブルシューティング

### よくある問題

#### アセンブリ定義ファイルのエラー

**症状**: コンパイルエラー、型が見つからない

**解決策**:
1. Unity を再起動
2. `Assets > Reimport All`
3. アセンブリ定義ファイルの参照を確認

#### パッケージが認識されない

**症状**: UPM パッケージが Package Manager に表示されない

**解決策**:
1. `Packages/manifest.json` を確認
2. `package.json` の形式が正しいか確認
3. Unity を再起動

#### サンプルシーンが動かない

**症状**: サンプルシーンでエラーが発生

**解決策**:
1. アセンブリ参照を確認
2. プレハブのリンク切れを確認
3. Scroller の設定を確認

## リソース

- **公式サイト**: https://setchi.jp/FancyScrollView/
- **API ドキュメント**: https://setchi.jp/FancyScrollView/api/FancyScrollView.html
- **デモ**: https://setchi.jp/FancyScrollView/demo
- **アップストリーム**: https://github.com/setchi/FancyScrollView
- **ライセンス**: MIT License

## 連絡先

質問や提案がある場合:
- Issue を作成
- Pull Request を送信
- アップストリームの Discussion を利用
