# トラブルシューティング

## よくある問題と解決方法

### ビルドが失敗する

#### Unity License エラー

```
Error: Invalid Unity license
```

**解決方法:**
- `UNITY_LICENSE` Secret が正しく設定されているか確認
- ライセンスファイルの有効期限を確認
- [GameCI のアクティベーションガイド](https://game.ci/docs/github/activation) を再度確認

#### Unity Email/Password エラー

```
Error: Authentication failed
```

**解決方法:**
- `UNITY_EMAIL` と `UNITY_PASSWORD` が正しいか確認
- Unity アカウントが有効か確認
- 2段階認証を無効化 (CI 専用アカウントを推奨)

### リリースが作成されない

#### タグのパターンが一致しない

**確認事項:**
- タグが `v*.*.*-fork.*` パターンに一致しているか
- 例: `v1.9.0-fork.0` は OK、`v1.9.0-fork` は NG

**正しい例:**
- ✅ `v1.9.0-fork.0`
- ✅ `v2.0.0-fork.1`
- ✅ `v1.10.0-fork.15`

**誤った例:**
- ❌ `v1.9.0-fork` (fork 番号がない)
- ❌ `1.9.0-fork.0` (先頭の `v` がない)
- ❌ `v1.9.0-fork-0` (ドットではなくハイフン)

#### 権限エラー

```
Error: Resource not accessible by integration
```

**解決方法:**
- ワークフローファイルの `permissions` 設定を確認
- `contents: write` が設定されているか確認

### ビルド時間が長い

#### キャッシュが効いていない

**原因:**
- 初回ビルドはキャッシュがないため 15-25 分かかる
- `Library/` フォルダの内容が変わるとキャッシュが無効化される

**対策:**
- キャッシュキーの設定を確認 (`hashFiles` の範囲)
- 2回目以降は 5-10 分程度に短縮される
