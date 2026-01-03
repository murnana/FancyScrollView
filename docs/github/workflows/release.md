# リリース作成

**ファイル:** [`.github/workflows/release.yml`](../../../.github/workflows/release.yml)

## リリースの作成手順

### 1. バージョン番号を決定

[バージョニング規則](../../versioning.md) に従ってバージョン番号を決定します。

- 上流の新バージョンを取り込んだ場合: `.0` にリセット (例: `2.0.0-fork.0`)
- フォーク独自の変更の場合: インクリメント (例: `1.9.0-fork.0` → `1.9.0-fork.1`)

### 2. package.json を更新

[`Packages/jp.setchi.fancyscrollview/package.json`](../../../Packages/jp.setchi.fancyscrollview/package.json) のバージョンを更新:

```json
{
  "version": "1.9.0-fork.1"
}
```

### 3. 変更をコミット

```bash
git add Packages/jp.setchi.fancyscrollview/package.json
git commit -m "Bump version to 1.9.0-fork.1"
```

### 4. タグを作成してプッシュ

```bash
# タグを作成 (package.json のバージョンと一致させる)
git tag v1.9.0-fork.1

# タグをプッシュ (これが GitHub Actions をトリガーします)
git push origin v1.9.0-fork.1
```

### 5. GitHub Actions の実行を確認

1. GitHub リポジトリの **Actions** タブを開く
2. **Create Release** ワークフローの実行状況を確認
3. 成功すると、**Releases** ページに新しいリリースが作成される

### 6. リリースを確認

1. GitHub リポジトリの **Releases** ページを開く
2. 新しいリリースが作成されていることを確認
3. ZIP ファイルがアップロードされていることを確認
4. リリースノートが自動生成されていることを確認

## トリガー条件

`v*.*.*-fork.*` パターンに一致するタグがプッシュされた時 (例: `v1.9.0-fork.0`, `v2.0.0-fork.1`)

## 実行内容

1. リポジトリをチェックアウト
2. タグからバージョン番号を抽出
3. Unity Library フォルダをキャッシュ (ビルド時間短縮)
4. Unity プロジェクトをビルド (Windows 64bit)
5. ビルド成果物を検証
6. ZIP パッケージを作成
   - デバッグフォルダを除外
   - README.txt を追加
7. GitHub Release を作成
   - ZIP ファイルをアップロード
   - リリースノートを自動生成

## ビルド成果物

- **ファイル名:** `FancyScrollView-{version}-Windows.zip`
- **例:** `FancyScrollView-1.9.0-fork.0-Windows.zip`
- **内容:**
  - `FancyScrollView.exe` (実行ファイル)
  - `FancyScrollView_Data/` (ゲームデータ)
  - `MonoBleedingEdge/` (Mono ランタイム)
  - `UnityCrashHandler64.exe`
  - `UnityPlayer.dll`
  - `README.txt` (実行方法、システム要件、サンプルシーン説明)
