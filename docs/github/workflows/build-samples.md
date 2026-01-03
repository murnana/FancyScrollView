# PR ビルド検証

**ファイル:** [`.github/workflows/build-samples.yml`](../../../.github/workflows/build-samples.yml)

## トリガー条件

- `develop` ブランチへのプルリクエスト
- 手動実行 (workflow_dispatch)

## 実行内容

1. Unity プロジェクトをビルド (Windows 64bit)
2. ビルド成果物をアーティファクトとしてアップロード (7日間保持)
3. ビルドが成功したことを検証

## 用途

- プルリクエストのビルドが正常に完了することを確認
- マージ前に問題を早期発見
- ビルド成果物をレビュー時にダウンロード可能
