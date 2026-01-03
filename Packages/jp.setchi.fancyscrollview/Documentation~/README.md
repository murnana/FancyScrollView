# FancyScrollView ドキュメント

FancyScrollView の公式ドキュメントへようこそ。

## ドキュメントの構成

### 📘 [ユーザーガイド](user-guides/)

FancyScrollView の使い方や実装方法を学ぶためのドキュメントです。

- [使い方ガイド](user-guides/getting-started.md) - FancyScrollView の概要と基本概念
- [基本的な実装](user-guides/basic-implementation.md) - 最小限のスクロールビューの実装方法
- [Context の使い方](user-guides/using-context.md) - Cell と ScrollView 間の通信
- [無限スクロール](user-guides/infinite-scroll.md) - ループするスクロールビューの実装
- [ScrollRect 形式](user-guides/scrollrect-usage.md) - 従来型リストの実装
- [GridView](user-guides/gridview-usage.md) - グリッドレイアウトの実装
- [カスタムアニメーション](user-guides/custom-animations.md) - アニメーションのカスタマイズ

### 📝 [変更履歴](modifications/)

上流リポジトリ (setchi/FancyScrollView) からの変更点や、UPM 移行に関する記録です。

- [変更履歴](modifications/modifications.md) - カスタマイズ内容の一覧
- [UPM 移行](modifications/upm-migration-summary.md) - UPM パッケージ化の記録
- [構造リファレンス](modifications/upm-structure-reference.md) - UPM パッケージ構造の説明

## はじめ方

1. [使い方ガイド](user-guides/getting-started.md) で全体像を把握
2. [基本的な実装](user-guides/basic-implementation.md) で実装手順を学習
3. 目的に応じて機能別ガイドを参照

## サンプルシーン

プロジェクトの `Assets/FancyScrollView/Examples/` に以下のサンプルがあります:

- **01_Basic** - 最小限の実装
- **02_FocusOn** - Context を使ったセル選択
- **03_InfiniteScroll** - 無限スクロール
- **04_Metaball** - シェーダーアニメーション
- **05_Voronoi** - シェーダーアニメーション
- **06_LoopTabBar** - タブナビゲーション
- **07_ScrollRect** - ScrollRect + スクロールバー
- **08_GridView** - グリッドレイアウト
- **09_LoadTexture** - 非同期テクスチャ読み込み

## 外部リソース

- [公式サイト](https://setchi.jp/FancyScrollView/)
- [API ドキュメント](https://setchi.jp/FancyScrollView/api/FancyScrollView.html)
- [デモサイト](https://setchi.jp/FancyScrollView/demo)
- [GitHub](https://github.com/setchi/FancyScrollView)

## 開発者向け情報

パッケージの開発については、プロジェクトルートの `docs/` ディレクトリを参照してください。

## ライセンス

FancyScrollView は MIT ライセンス下で提供されています。

```
Copyright (c) 2020 setchi
Licensed under MIT
```
