# FancyScrollView 使い方ガイド

## 概要

FancyScrollView は、Unity用の高度にカスタマイズ可能なスクロールビューコンポーネントです。無限スクロール、スナップ、グリッドレイアウト、カスタムアニメーションなどの機能を提供します。

### 主な特徴

- **無限スクロール**: セルを循環配置して無限にスクロール可能
- **スナップ機能**: セルに自動的にスナップ
- **パフォーマンス**: セルプーリングにより、表示中のセルのみ生成・更新
- **柔軟性**: 正規化された位置(0.0-1.0)を使用したカスタムアニメーション
- **多様なレイアウト**: 標準スクロール、ScrollRect形式、グリッドレイアウトに対応

## 必要要件

- Unity 6.0 以降
- .NET 4.x Scripting Runtime

## インストール

このパッケージは UPM (Unity Package Manager) パッケージとして提供されています。

### ローカルパッケージとして

プロジェクトの`Packages/`ディレクトリに配置されている`jp.setchi.fancyscrollview`パッケージが自動的に認識されます。

## 基本概念

### アーキテクチャ

FancyScrollView は以下の主要コンポーネントで構成されています:

1. **ScrollView** - スクロールビュー本体。セルの生成・配置・更新を管理
2. **Cell** - 各アイテムを表示するUI要素。データの表示とアニメーションを担当
3. **Scroller** - スクロール位置の制御とユーザー入力の処理
4. **Context** (オプション) - ScrollView と Cell 間でデータを共有するためのオブジェクト

### データフロー

```
ItemData[] → ScrollView → Cell.UpdateContent(itemData)
                        ↓
                  Cell.UpdatePosition(normalizedPosition)
```

### セルプーリング

パフォーマンス最適化のため、FancyScrollView は可視領域に必要な数のセルのみを生成します。スクロール時にセルは再利用され、新しいデータで更新されます。

### 正規化された位置

セルの位置は常に 0.0 ～ 1.0 の正規化された値として扱われます:

- `0.0` - ビューポートの開始位置
- `0.5` - ビューポートの中央
- `1.0` - ビューポートの終了位置

この値を使用してカスタムアニメーションを実装できます。

## 次のステップ

- [基本的な実装](basic-implementation.md) - 最小限の実装例
- [Context の使い方](using-context.md) - Cell と ScrollView 間の通信
- [無限スクロール](infinite-scroll.md) - 無限スクロールの実装方法
- [ScrollRect 形式](scrollrect-usage.md) - 従来型リストの実装
- [GridView](gridview-usage.md) - グリッドレイアウトの実装
- [カスタムアニメーション](custom-animations.md) - アニメーションのカスタマイズ
- [API リファレンス](https://setchi.jp/FancyScrollView/api/FancyScrollView.html)

## サンプルシーン

`Assets/FancyScrollView/Examples/`に以下のサンプルが含まれています:

1. **01_Basic** - 最小限の実装
2. **02_FocusOn** - Context を使ったセル選択
3. **03_InfiniteScroll** - 無限スクロール
4. **04_Metaball** - シェーダーベースのアニメーション
5. **05_Voronoi** - シェーダーベースのアニメーション
6. **06_LoopTabBar** - タブナビゲーション
7. **07_ScrollRect** - ScrollRect + スクロールバー
8. **08_GridView** - グリッドレイアウト
9. **09_LoadTexture** - 非同期テクスチャ読み込み

## リソース

- [デモサイト](https://setchi.jp/FancyScrollView/demo)
- [API ドキュメント](https://setchi.jp/FancyScrollView/api/FancyScrollView.html)
- [GitHub リポジトリ](https://github.com/setchi/FancyScrollView)
