# 改変箇所の記録

このドキュメントは、上流リポジトリ (setchi/FancyScrollView) から加えられた変更をまとめたものです。

## 概要

- **ベースリポジトリ**: [setchi/FancyScrollView](https://github.com/setchi/FancyScrollView)
- **フォーク**: [murnana/FancyScrollView](https://github.com/murnana/FancyScrollView)
- **分岐点**: コミット `8fce45d` (2024年頃)

## 変更履歴

### 1. Claude Code 統合 (2026-01-02)

**コミット**: `0177b46` - "Add Claude Code configuration files"

**追加ファイル**:
- `.claude/.gitignore` - Claude Code の設定ファイル用 gitignore
- `.claude/settings.json` - Claude Code のプロジェクト設定
- `CLAUDE.md` - Claude Code 向けプロジェクトドキュメント

**目的**: Claude Code を使った開発を効率化するための設定とドキュメントの追加

### 2. Unity バージョンアップデート (2026-01-02)

一連のコミットでUnityのバージョンを段階的にアップデート:

#### 2.1. Unity 2019.4.41f2 へのアップデート
**コミット**: `2b9ff00` - "Update Unity version to 2019.4.41f2"

**変更内容**:
- Unity Editor: `2019.4.7f1` → `2019.4.41f2`
- パッケージマニフェストとロックファイルの追加
- プロジェクト設定ファイルの更新

#### 2.2. Unity 2020.3.49f1 へのアップデート
**コミット**: `8eae7b7` - "Update Unity version from 2019.4.41f2 to 2020.3.49f1"

#### 2.3. Unity 2021.3.45f2 へのアップデート
**コミット**: `37610d7` - "Update Unity version to 2021.3.45f2"

#### 2.4. Unity 2022.3.62f3 へのアップデート
**コミット**: `b295620` - "Update Unity version to 2022.3.62f3"

#### 2.5. Unity 6.0.64f1 へのアップデート (最新)
**コミット**: `d71edb0` - "Update Unity version to 6.0.64f1"

**主な変更**:
- Unity Editor: `2022.3.62f3` → `6.0.64f1` (Unity 6 LTS)
- 最小要件: `Unity 2019.4+` → `Unity 6.0+`
- `com.unity.test-framework`: Unity 6 互換バージョンへ更新

**変更したファイル**:
- `README.md` - 必須バージョンのバッジを更新
- `CLAUDE.md` - Unity バージョン要件を更新
- `Packages/manifest.json` - パッケージ依存関係の更新
- `ProjectSettings/` - Unity 6 対応のプロジェクト設定

**理由**:
- Unity 2019.4 LTS、2020 LTS、2021 LTS、2022 LTS はすべてサポート終了
- Unity 6.0 LTS は 2026年10月までサポート (Enterprise/Industry は 2027年10月まで)

## コードレベルの変更

### TextureLoader.cs の Unity 6 対応

**ファイル**: `Assets/FancyScrollView/Examples/Sources/09_LoadTexture/TextureLoader.cs`

#### 変更1: FindObjectOfType の非推奨 API 対応

```csharp
// Unity 6.0 以降
#if UNITY_6000_0_OR_NEWER
    (instance = FindAnyObjectByType<Loader>() ??
#else
    (instance = FindObjectOfType<Loader>() ??
#endif
```

**理由**: Unity 6.0 で `FindObjectOfType` が非推奨となり、`FindAnyObjectByType` が推奨 API になった

#### 変更2: UnityWebRequest のエラーハンドリング更新

```csharp
#if UNITY_2020_2_OR_NEWER
    if(request.result != UnityWebRequest.Result.Success)
#else
    if (request.isNetworkError)
#endif
```

**理由**: Unity 2020.2 以降で `isNetworkError` が非推奨となり、`result` プロパティでの判定が推奨された

## 設定ファイルの変更

### 新規追加されたプロジェクト設定

以下の設定ファイルが Unity バージョンアップに伴い追加されました:

- `ProjectSettings/MemorySettings.asset` - Unity 6 のメモリ管理設定
- `ProjectSettings/MultiplayerManager.asset` - Unity 6 のマルチプレイヤー設定
- `ProjectSettings/PackageManagerSettings.asset` - パッケージマネージャー設定
- `ProjectSettings/SceneTemplateSettings.json` - シーンテンプレート設定
- `ProjectSettings/TimelineSettings.asset` - Timeline 設定
- `ProjectSettings/VersionControlSettings.asset` - バージョン管理設定
- `ProjectSettings/XRSettings.asset` - XR 設定

### パッケージの変更

`Packages/manifest.json` と `Packages/packages-lock.json` が Unity 6 対応のパッケージバージョンに更新されました。

主な変更:
- `com.unity.test-framework`: v1.1.14 → v1.6.0

## 上流リポジトリとの互換性

### 互換性のある変更

- コードレベルの変更は条件付きコンパイル (`#if`) を使用しているため、古い Unity バージョンでも動作する
- Unity 6 特化の機能は使用していない

### 非互換性

- **Unity バージョン要件**: 上流は `2019.4+`、このフォークは `6.0+`
- プロジェクト設定ファイルが Unity 6 用に更新されているため、古い Unity では開けない可能性がある

## 今後の方針

### マージ戦略

上流から変更を取り込む際は、以下に注意:

1. **Unity バージョンの違い**: 上流が古いバージョンを対象としている場合、プロジェクト設定の競合が発生する可能性
2. **API の違い**: Unity 6 で追加された条件付きコンパイルを維持する必要がある
3. **README/CLAUDE.md**: Unity バージョン要件の記載を維持する

### 推奨アプローチ

- 上流のバグ修正やコア機能の改善は積極的にマージする
- プロジェクト設定ファイルの競合は、このフォークの設定を優先する
- コードの変更は、条件付きコンパイルで互換性を保つ

## 参考情報

- **上流リポジトリ**: https://github.com/setchi/FancyScrollView
- **このフォーク**: https://github.com/murnana/FancyScrollView
- **Unity 6 LTS リリースノート**: https://unity.com/releases/lts
- **Unity API の非推奨・変更履歴**: Unity ドキュメント参照
