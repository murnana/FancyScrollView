# FancyScrollView 開発者ドキュメント

このディレクトリには、FancyScrollView パッケージの開発・メンテナンスに関するドキュメントが含まれています。

## ドキュメント一覧

### 開発ガイド

- [**開発ガイド**](development-guide.md) - 開発環境のセットアップ、コーディング規約、ビルド手順
- [**アーキテクチャ**](architecture.md) - 内部構造と設計思想の詳細解説
- [**API リファレンス**](api-reference.md) - 主要なクラスとメソッドのリファレンス
- [**ブランチ戦略**](branch-strategy.md) - Git ブランチの運用方針
- [**バージョニング規則**](versioning.md) - バージョン番号の付け方とリリースフロー
- [**CI/CD ガイド**](github/workflows/ci-cd.md) - GitHub Actions によるビルドとリリースの自動化

## 対象読者

このドキュメントは以下の方を対象としています:

- FancyScrollView のコードベースに貢献したい開発者
- パッケージの内部動作を理解したい開発者
- カスタマイズや拡張を行いたい開発者

## ユーザー向けドキュメント

FancyScrollView の**使い方**については、以下を参照してください:

- [パッケージドキュメント](../Packages/jp.setchi.fancyscrollview/Documentation~/)

## クイックスタート

### 開発環境のセットアップ

```bash
# リポジトリのクローン
git clone <repository-url>
cd FancyScrollView

# Unity Hub でプロジェクトを開く
# Unity 6.0.64f1 以降を使用
```

### 開発ブランチ

- **`master`**: アップストリーム同期用(直接開発しない)
- **`develop`**: メイン開発ブランチ

詳細は [ブランチ戦略](branch-strategy.md) を参照。

### コーディング規約

```csharp
// クラス名: PascalCase
public class FancyScrollView { }

// メソッド: PascalCase
public void UpdatePosition() { }

// フィールド: camelCase
[SerializeField] GameObject cellPrefab;

// コメント: 日本語
/// <summary>
/// スクロール位置を更新します.
/// </summary>
```

詳細は [開発ガイド](development-guide.md#コーディング規約) を参照。

## プロジェクト構造

```
FancyScrollView/
├── Packages/
│   └── jp.setchi.fancyscrollview/  # UPM パッケージ本体
│       ├── Runtime/                # ランタイムコード
│       ├── Editor/                 # エディタ拡張
│       └── Documentation~/         # ユーザー向けドキュメント
├── Assets/
│   └── FancyScrollView/
│       └── Examples/               # サンプルシーン・コード
└── docs/                           # 開発者向けドキュメント(このディレクトリ)
```

## アーキテクチャ概要

FancyScrollView は以下の主要コンポーネントで構成されています:

### Core モジュール

- **FancyScrollView**: スクロールビューの基底クラス
- **FancyCell**: セルの基底クラス

### Scroller モジュール

- **Scroller**: スクロール制御とユーザー入力処理
- **Draggable**: ドラッグ処理

### ScrollRect / GridView モジュール

- **FancyScrollRect**: 従来型スクロールリスト
- **FancyGridView**: グリッドレイアウト

詳細は [アーキテクチャ](architecture.md) を参照。

## 主要な設計パターン

### セルプーリング

パフォーマンス最適化のため、表示中のセルのみ生成し、再利用します。

```csharp
// セルの生成は最小限
void ResizePool(float firstPosition)
{
    var requiredCells = CalculateRequiredCells(firstPosition);
    // 不足分のみ生成
}

// セルは破棄せず再利用
void UpdateCells()
{
    foreach (var cell in pool)
    {
        // 表示範囲に応じて更新または非表示
    }
}
```

### Context パターン

ScrollView と Cell 間のデータ共有に Context を使用:

```csharp
// 同じインスタンスを共有
protected TContext Context { get; } = new TContext();

// Cell でアクセス可能
public override void UpdateContent(ItemData data)
{
    var isSelected = Context.SelectedIndex == Index;
}
```

### 正規化座標系

セル位置を 0.0 ～ 1.0 の正規化値で扱い、柔軟なアニメーションを実現:

```csharp
public override void UpdatePosition(float position)
{
    // position: 0.0(画面外) ～ 0.5(中央) ～ 1.0(画面外)
    var distance = Mathf.Abs(0.5f - position);
    transform.localScale = Vector3.one * (1f - distance);
}
```

詳細は [アーキテクチャ](architecture.md) を参照。

## よくある開発タスク

### 新機能の追加

1. 適切なモジュール(Core/Scroller/ScrollRect/GridView)を選択
2. 既存のクラスを継承または拡張
3. サンプルシーンで動作確認
4. ドキュメントを更新

### バグ修正

1. 該当するサンプルシーンで再現
2. 修正を実装
3. 関連する全サンプルシーンで回帰テスト
4. 必要に応じてドキュメント更新

### アップストリームとの同期

```bash
# fetch
git fetch upstream

# master にマージ
git checkout master
git merge upstream/master

# develop に反映
git checkout develop
git merge master
```

詳細は [ブランチ戦略](branch-strategy.md) を参照。

## テストとデバッグ

### サンプルシーンの活用

`Assets/FancyScrollView/Examples/` のサンプルシーンを使用:

1. 変更を加える
2. 該当するサンプルシーンで動作確認
3. 複数のサンプルで回帰テスト

### Unity Test Framework

```
Window > General > Test Runner
```

テストコードの追加:
```csharp
using NUnit.Framework;

public class FancyScrollViewTests
{
    [Test]
    public void TestCellPooling()
    {
        // テスト実装
    }
}
```

## パフォーマンス考慮事項

- セルプーリングで GC Alloc を削減
- `UpdateContent` は最小限の呼び出し
- `UpdatePosition` は毎フレーム呼ばれるため軽量に
- レイアウト計算結果のキャッシュ

詳細は [アーキテクチャ](architecture.md#パフォーマンス最適化) を参照。

## ライセンスとクレジット

このプロジェクトは MIT ライセンス下にあります:

```
Copyright (c) 2020 setchi
Licensed under MIT
```

**重要**:
- オリジナルの LICENSE ファイルを保持する必要があります
- フォークであることを明記してください
- カスタム変更を加えてもライセンスは変更できません

## 外部リソース

- [公式サイト](https://setchi.jp/FancyScrollView/)
- [API ドキュメント](https://setchi.jp/FancyScrollView/api/FancyScrollView.html)
- [アップストリーム](https://github.com/setchi/FancyScrollView)

## 貢献

### フォーク固有の機能

- `develop` ブランチで開発

### アップストリームへの貢献

- [setchi/FancyScrollView](https://github.com/setchi/FancyScrollView) へ PR を送る

## サポート

質問や提案がある場合:
- Issue を作成
- Pull Request を送信
- アップストリームの Discussion を利用
