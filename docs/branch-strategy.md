# ブランチ戦略ガイド

## 概要

このプロジェクトは [setchi/FancyScrollView](https://github.com/setchi/FancyScrollView) のフォークであり、カスタマイズを加えた派生版です。上流リポジトリからの更新を取り込みつつ、独自の変更を管理するために、以下のブランチ戦略を採用しています。

## ブランチ構成

### `master` ブランチ
- **用途**: 上流リポジトリ (upstream) との同期専用
- **特徴**:
  - 元のプロジェクト (setchi/FancyScrollView) の状態を保持
  - 直接コミットは行わない
  - 上流のバグ修正やアップデートを取り込むためのクリーンな状態を維持

### `develop` ブランチ
- **用途**: カスタマイズ・改変作業のメインブランチ
- **特徴**:
  - 日常的な開発作業はここで行う
  - 独自の機能追加や変更を実装
  - 必要に応じて `master` からの更新をマージ

## この戦略を採用した理由

### メリット

1. **上流の更新を取り込みやすい**
   - 元リポジトリで重要なバグ修正や機能追加があった場合、`master` にマージしてから `develop` に取り込める
   - マージコンフリクトを段階的に解決できる

2. **変更履歴が明確**
   - どこまでがオリジナルで、どこからが独自の変更かが一目瞭然
   - 将来的に変更を見直す際に役立つ

3. **リスク管理**
   - 大きな実験的変更を試す際、いつでも元の状態に戻れる
   - フィーチャーブランチを `develop` から分岐させることも可能

### デメリット (master に直接コミットした場合との比較)

- `master` に直接コミットすると、上流との同期時にマージコンフリクトが複雑になる
- 「どこまでがオリジナル」かの判別が困難になる

## 日常の作業フロー

### 基本的な開発作業

```bash
# develop ブランチで作業
git checkout develop

# 変更を加える
# ... コーディング作業 ...

# コミット
git add .
git commit -m "機能追加: ..."

# 自分のリポジトリにプッシュ
git push origin develop
```

### フィーチャーブランチを使う場合 (オプション)

大きな機能を追加する際は、`develop` からフィーチャーブランチを作成することも可能です。

```bash
# develop から新しいフィーチャーブランチを作成
git checkout develop
git checkout -b feature/new-feature

# 開発作業
# ... コーディング ...

# develop にマージ
git checkout develop
git merge feature/new-feature

# フィーチャーブランチを削除 (任意)
git branch -d feature/new-feature
```

## 上流リポジトリとの同期方法

上流 (setchi/FancyScrollView) に重要な更新があった場合の手順:

### 1. 上流の最新版を master に取り込む

```bash
# master ブランチに移動
git checkout master

# 上流の最新版を取得してマージ
git fetch upstream
git merge upstream/master

# 自分のリポジトリの master を更新
git push origin master
```

### 2. master の更新を develop にマージ

```bash
# develop ブランチに移動
git checkout develop

# master の変更を develop にマージ
git merge master

# コンフリクトが発生した場合は解決する
# ... コンフリクト解決 ...

# 自分のリポジトリの develop を更新
git push origin develop
```

### コンフリクト解決のヒント

- コンフリクトが発生した場合、独自の変更を優先するか、上流の変更を優先するかを慎重に判断してください
- 重要なバグ修正は上流の変更を優先し、独自機能は自分の変更を維持するのが一般的です

## ライセンスに関する注意事項

### MIT ライセンスの要件

このプロジェクトは MIT ライセンス (Copyright (c) 2020 setchi) のもとで公開されています。

**必須事項:**
- 元の LICENSE ファイルを保持する (削除・変更しない)
- 元の著作権表示を維持する

**推奨事項:**
- README やドキュメントに元プロジェクトへのクレジットを記載
- 大きな変更を加えた場合、独自の著作権表示を追加することも可能

### 配布時の注意

このカスタマイズ版を他者に配布する場合:
- 元の MIT ライセンスファイルを必ず含める
- 元プロジェクトへのリンクと著作権表示を明記する

詳細は [LICENSE](../../LICENSE) ファイルを参照してください。

## 参考リンク

- 元プロジェクト: https://github.com/setchi/FancyScrollView
- 元プロジェクトのライセンス: https://github.com/setchi/FancyScrollView/blob/master/LICENSE
