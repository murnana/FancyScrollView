# FancyScrollView ドキュメント

FancyScrollView の使い方ドキュメントへようこそ。

## ドキュメント一覧

### 入門ガイド

- [**使い方ガイド**](getting-started.md) - FancyScrollView の概要と基本概念
- [**基本的な実装**](basic-implementation.md) - 最小限のスクロールビューの実装方法

### 機能別ガイド

- [**Context の使い方**](using-context.md) - Cell と ScrollView 間の通信
- [**無限スクロール**](infinite-scroll.md) - ループするスクロールビューの実装
- [**ScrollRect 形式**](scrollrect-usage.md) - 従来型リストの実装
- [**GridView**](gridview-usage.md) - グリッドレイアウトの実装
- [**カスタムアニメーション**](custom-animations.md) - アニメーションのカスタマイズ

### 変更履歴・参考資料

- [**変更履歴**](../modifications/) - UPM 移行やカスタム変更の記録

## はじめ方

1. まず [使い方ガイド](getting-started.md) で全体像を把握
2. [基本的な実装](basic-implementation.md) で実装手順を学習
3. 必要に応じて機能別ガイドを参照

## 実装の流れ

FancyScrollView を使った実装は以下の流れで進めます:

```
1. データモデルの定義 (ItemData)
    ↓
2. Cell の実装 (FancyCell を継承)
    ↓
3. ScrollView の実装 (FancyScrollView を継承)
    ↓
4. Scene でのセットアップ
    ↓
5. データの設定とスクロール制御
```

詳細は [基本的な実装](basic-implementation.md) を参照してください。

## どの機能を使うべきか

### FancyScrollView (標準)

以下の場合に使用:
- ✓ 無限スクロールが必要
- ✓ スナップ機能が必要
- ✓ カスタムアニメーションを実装したい
- ✓ 柔軟なレイアウト制御が必要

### FancyScrollRect

以下の場合に使用:
- ✓ 従来型の縦/横スクロールリスト
- ✓ スクロールバーが必要
- ✓ 可変サイズのセル
- ✓ シンプルな実装を求める

### FancyGridView

以下の場合に使用:
- ✓ グリッド(格子状)レイアウト
- ✓ 複数列/行の表示

## サンプルシーン

`Assets/FancyScrollView/Examples/` に以下のサンプルがあります:

| シーン | 説明 |
|--------|------|
| 01_Basic | 最小限の実装 |
| 02_FocusOn | Context を使ったセル選択 |
| 03_InfiniteScroll | 無限スクロール |
| 04_Metaball | シェーダーアニメーション |
| 05_Voronoi | シェーダーアニメーション |
| 06_LoopTabBar | タブナビゲーション |
| 07_ScrollRect | ScrollRect + スクロールバー |
| 08_GridView | グリッドレイアウト |
| 09_LoadTexture | 非同期テクスチャ読み込み |

## 外部リソース

- [公式サイト](https://setchi.jp/FancyScrollView/)
- [API ドキュメント](https://setchi.jp/FancyScrollView/api/FancyScrollView.html)
- [デモサイト](https://setchi.jp/FancyScrollView/demo)
- [GitHub](https://github.com/setchi/FancyScrollView)

## 開発者向け情報

パッケージの開発やカスタマイズについては、プロジェクトルートの `docs/` ディレクトリを参照してください:

- [開発ガイド](../../../../docs/development-guide.md)
- [アーキテクチャ](../../../../docs/architecture.md)
- [ブランチ戦略](../../../../docs/branch-strategy.md)

## ライセンス

FancyScrollView は MIT ライセンス下で提供されています。

```
Copyright (c) 2020 setchi
Licensed under MIT (https://github.com/setchi/FancyScrollView/blob/master/LICENSE)
```
