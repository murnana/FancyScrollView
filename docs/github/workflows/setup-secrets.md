# Secrets の設定

GitHub Actions を実行するには、以下の Secrets を設定する必要があります。

## 設定方法

1. GitHub リポジトリの **Settings** タブを開く
2. **Secrets and variables > Actions** を選択
3. **New repository secret** をクリック
4. 以下の3つの Secret を追加

## 必要な Secrets

### `UNITY_LICENSE`

- **内容:** Unity ライセンスファイル (.ulf) の内容
- **取得方法:**
  1. [GameCI のアクティベーションガイド](https://game.ci/docs/github/activation) に従う
  2. Personal License または Professional License のアクティベーションファイルを取得
  3. `.ulf` ファイルの内容をそのままコピー

### `UNITY_EMAIL`

- **内容:** Unity アカウントのメールアドレス
- **注意:** Unity ID にログインする際のメールアドレス

### `UNITY_PASSWORD`

- **内容:** Unity アカウントのパスワード
- **注意:** セキュリティのため、個人アカウントではなく CI 専用アカウントの使用を推奨

## Secrets 設定の確認

設定が正しいか確認するには:

1. テスト用のブランチで `.github/workflows/build-samples.yml` を手動実行
2. **Actions** タブでワークフローの実行結果を確認
3. エラーが出た場合は Secrets の設定を見直す

