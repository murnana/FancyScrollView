# バージョニング規則

## 概要

このリポジトリは [setchi/FancyScrollView](https://github.com/setchi/FancyScrollView) のフォークです。上流リポジトリとの区別を明確にするため、すべてのバージョンに **`-fork`** サフィックスを使用し、さらにフォーク固有の変更を追跡するための番号を付けます。

## バージョン形式

```
X.Y.Z-fork.W
```

- **X.Y.Z**: セマンティックバージョン (上流と同じ)
- **-fork**: フォーク識別子 (プレリリース識別子として機能)
- **W**: フォーク固有のバージョン番号

### 例

| 上流リポジトリ | このフォーク | 説明 |
|---|---|---|
| `v1.9.0` | `v1.9.0-fork.0` | 上流 v1.9.0 に基づく初回リリース |
| `v1.9.0` | `v1.9.0-fork.1` | 上流 v1.9.0 に基づくフォーク独自の修正 |
| `v1.9.0` | `v1.9.0-fork.2` | 上流 v1.9.0 に基づくフォーク独自の2回目の修正 |
| `v2.0.0` | `v2.0.0-fork.0` | 上流 v2.0.0 に基づく初回リリース |

## フォーク番号 (W) の使い方

### インクリメントするケース

フォーク番号 `.W` は以下の場合にインクリメントします:

1. **フォーク固有のバグ修正**
   - 上流にない独自機能のバグを修正した場合
   - 例: `1.9.0-fork.0` → `1.9.0-fork.1`

2. **フォーク固有の機能追加**
   - 上流にない新機能を追加した場合
   - 例: `1.9.0-fork.1` → `1.9.0-fork.2`

3. **フォーク固有の改善・最適化**
   - パフォーマンス改善やリファクタリングなど
   - 例: `1.9.0-fork.2` → `1.9.0-fork.3`

### リセットするケース

フォーク番号 `.W` は以下の場合に `.0` にリセットします:

1. **上流の新バージョンを取り込んだ場合**
   - 上流が `v2.0.0` をリリース → `2.0.0-fork.0`
   - 上流が `v1.10.0` をリリース → `1.10.0-fork.0`

## UPM パッケージバージョン

[package.json](../Packages/jp.setchi.fancyscrollview/package.json) には `-fork.W` サフィックスを含めます:

```json
{
  "name": "jp.setchi.fancyscrollview",
  "version": "1.9.0-fork.1"
}
```

## Git タグ

### 命名規則

```bash
v<major>.<minor>.<patch>-fork.<number>
```

### リリース作成手順

1. `package.json` のバージョンを更新
2. 変更をコミット
3. タグを作成してプッシュ:
   ```bash
   git tag v1.9.0-fork.1
   git push origin v1.9.0-fork.1
   ```
4. GitHub Actions が自動的に実行:
   - Windows 実行ファイルをビルド
   - GitHub Release を作成
   - ビルド成果物 (ZIP) をアップロード

## GitHub Actions

### リリースワークフロー

[`.github/workflows/release.yml`](../.github/workflows/release.yml) は `-fork.*` サフィックス付きタグでトリガーされます:

```yaml
on:
  push:
    tags:
      - 'v*.*.*-fork'
```

### バージョン抽出

```bash
# タグ: v1.9.0-fork.1
# 抽出結果: 1.9.0-fork.1
VERSION=${GITHUB_REF#refs/tags/v}
```

出力ファイル名: `FancyScrollView-1.9.0-fork.1-Windows.zip`

## 上流リポジトリとの同期戦略

### ワークフロー

1. **上流の更新を取得**
   ```bash
   git checkout master
   git fetch upstream
   git merge upstream/master
   git push origin master
   ```

2. **develop にマージ**
   ```bash
   git checkout develop
   git merge master
   ```

3. **コンフリクト解決とテスト**

4. **バージョン更新**
   - 上流が `v2.0.0` をリリース
   - このフォークは `v2.0.0-fork.0` に更新
   - `package.json` を更新してリリース

### バージョン番号の決定

- **Major/Minor/Patch**: 上流と同じ
- **Fork サフィックス**: 常に `-fork.W`
- **Fork 番号 (W)**: 上流の新バージョン取り込み時は `.0` にリセット、それ以外はインクリメント

#### 例: 上流を追従する場合

- 上流: `v1.9.0` → `v1.10.0`
- このフォーク: `v1.9.0-fork.3` → `v1.10.0-fork.0` (リセット)

#### 例: フォーク独自の変更の場合

- このフォーク: `v1.9.0-fork.0` → `v1.9.0-fork.1` → `v1.9.0-fork.2` (インクリメント)

## セマンティックバージョニング準拠

`-fork.W` は [SemVer 2.0.0](https://semver.org/) の**プレリリース識別子**として機能します。

バージョン優先順位:
```
1.9.0 > 1.9.0-fork.2 > 1.9.0-fork.1 > 1.9.0-fork.0
```

`-fork` サフィックスは、これが上流の安定版リリースの派生版であることを正確に示します。

## メリット

- ✅ 上流リポジトリとの明確な区別
- ✅ SemVer 準拠
- ✅ 上流の更新を容易に追跡可能
- ✅ UPM との整合性
- ✅ フォーク固有の変更履歴を追跡可能

## トラブルシューティング

### 誤ったタグを作成した場合

```bash
# ローカルタグを削除
git tag -d v1.9.0-fork.1

# リモートタグを削除
git push origin :refs/tags/v1.9.0-fork.1

# 正しいタグを再作成
git tag v1.9.0-fork.1
git push origin v1.9.0-fork.1
```

### リリースが作成されない場合

1. タグが `v*.*.*-fork` パターンに一致するか確認
2. GitHub Actions ワークフローのステータスを確認
3. Secrets の設定を確認:
   - `UNITY_LICENSE`
   - `UNITY_EMAIL`
   - `UNITY_PASSWORD`

### バージョン不一致

`package.json` と Git タグのバージョンは一致している必要があります:

- ❌ 誤り: `package.json: 1.9.0-fork.1` / タグ: `v2.0.0-fork.0`
- ✅ 正しい: `package.json: 1.9.0-fork.1` / タグ: `v1.9.0-fork.1`

## バージョン履歴の例

実際の運用例:

```
v1.9.0-fork.0  - 上流 v1.9.0 に基づく初回フォークリリース
v1.9.0-fork.1  - GitHub Actions ワークフロー追加
v1.9.0-fork.2  - カスタムアニメーション機能追加
v1.9.0-fork.3  - パフォーマンス最適化
v1.10.0-fork.0 - 上流 v1.10.0 を取り込み (フォーク番号リセット)
v1.10.0-fork.1 - 新しいサンプルシーン追加
v2.0.0-fork.0  - 上流 v2.0.0 を取り込み (フォーク番号リセット)
```

## 参考リンク

- [元プロジェクト](https://github.com/setchi/FancyScrollView)
- [セマンティックバージョニング 2.0.0](https://semver.org/lang/ja/)
- [GameCI ドキュメント](https://game.ci/docs/)
