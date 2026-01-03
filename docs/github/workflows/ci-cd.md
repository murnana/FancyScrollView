# CI/CD ガイド

## 概要

このプロジェクトでは GitHub Actions を使用して、ビルドとリリースを自動化しています。

## ワークフロー一覧

### PR ビルド検証

詳細は [PR ビルド検証](build-samples.md) を参照してください。

- **トリガー:** `develop` ブランチへのプルリクエスト、手動実行
- **実行内容:** Unity プロジェクトをビルドし、成果物をアーティファクトとしてアップロード
- **用途:** マージ前のビルド検証

### リリース作成

詳細は [リリース作成](release.md) を参照してください。

- **トリガー:** `v*.*.*-fork.*` パターンのタグがプッシュされた時
- **実行内容:** ビルド、ZIPパッケージ作成、GitHub Releasesへの公開
- **成果物:** `FancyScrollView-{version}-Windows.zip`

### Secrets 設定

GitHub Actions を実行するには、以下の Secrets が必要です。設定方法は [Secrets 設定ガイド](setup-secrets.md) を参照してください。

- `UNITY_LICENSE`
- `UNITY_EMAIL`
- `UNITY_PASSWORD`

### トラブルシューティング

よくある問題と解決方法については [トラブルシューティング](troubleshootings.md) を参照してください。

## 参考リンク

- [バージョニング規則](../../versioning.md)
- [GameCI ドキュメント](https://game.ci/docs/)
- [GameCI Unity Builder](https://game.ci/docs/github/builder)
- [GameCI アクティベーション](https://game.ci/docs/github/activation)
- [GitHub Actions ドキュメント](https://docs.github.com/ja/actions)
