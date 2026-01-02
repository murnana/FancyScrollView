# FancyScrollView UPM ブランチ構造リファレンス

このドキュメントでは、アップストリーム [setchi/FancyScrollView](https://github.com/setchi/FancyScrollView/tree/upm) の `upm` ブランチから取得した完全なディレクトリ構造と主要なファイル情報を提供します。

**最終更新:** 2026-01-02
**アップストリームバージョン:** 1.9.0

## 概要

UPMブランチは、Unity Package Manager (UPM) 互換パッケージとして以下の特徴を持つように構成されています：

- パッケージ名: `jp.setchi.fancyscrollview`
- バージョン: 1.9.0
- Unity 最小要件: 2019.4
- ライセンス: MIT
- サンプルは `Samples~` ディレクトリで配布（UPM標準）

## アップストリーム UPM ブランチの完全なディレクトリ構造

```
FancyScrollView (UPM ブランチルート)
├── .gitignore
├── LICENSE
├── LICENSE.meta
├── README.md
├── README.md.meta
├── package.json
├── package.json.meta
├── Sources/
│   ├── Sources.meta
│   ├── Runtime/
│   │   ├── Runtime.meta
│   │   ├── FancyScrollView.asmdef
│   │   ├── FancyScrollView.asmdef.meta
│   │   ├── Core/
│   │   │   ├── Core.meta
│   │   │   ├── FancyCell.cs
│   │   │   ├── FancyCell.cs.meta
│   │   │   ├── FancyScrollView.cs
│   │   │   └── FancyScrollView.cs.meta
│   │   ├── Scroller/
│   │   │   ├── Scroller.meta
│   │   │   ├── EasingCore.cs
│   │   │   ├── EasingCore.cs.meta
│   │   │   ├── MovementDirection.cs
│   │   │   ├── MovementDirection.cs.meta
│   │   │   ├── MovementType.cs
│   │   │   ├── MovementType.cs.meta
│   │   │   ├── ScrollDirection.cs
│   │   │   ├── ScrollDirection.cs.meta
│   │   │   ├── Scroller.cs
│   │   │   └── Scroller.cs.meta
│   │   ├── ScrollRect/
│   │   │   ├── ScrollRect.meta
│   │   │   ├── FancyScrollRect.cs
│   │   │   ├── FancyScrollRect.cs.meta
│   │   │   ├── FancyScrollRectCell.cs
│   │   │   ├── FancyScrollRectCell.cs.meta
│   │   │   ├── FancyScrollRectContext.cs
│   │   │   ├── FancyScrollRectContext.cs.meta
│   │   │   ├── IFancyScrollRectContext.cs
│   │   │   └── IFancyScrollRectContext.cs.meta
│   │   └── GridView/
│   │       ├── GridView.meta
│   │       ├── FancyCellGroup.cs
│   │       ├── FancyCellGroup.cs.meta
│   │       ├── FancyGridView.cs
│   │       ├── FancyGridView.cs.meta
│   │       ├── FancyGridViewCell.cs
│   │       ├── FancyGridViewCell.cs.meta
│   │       ├── FancyGridViewContext.cs
│   │       ├── FancyGridViewContext.cs.meta
│   │       ├── IFancyCellGroupContext.cs
│   │       ├── IFancyCellGroupContext.cs.meta
│   │       ├── IFancyGridViewContext.cs
│   │       └── IFancyGridViewContext.cs.meta
│   └── Editor/
│       ├── Editor.meta
│       ├── FancyScrollView.Editor.asmdef
│       ├── FancyScrollView.Editor.asmdef.meta
│       ├── ScrollerEditor.cs
│       └── ScrollerEditor.cs.meta
└── Samples~/
    ├── Sources.meta
    ├── 01_Basic.unity
    ├── 01_Basic.unity.meta
    ├── 02_FocusOn.unity
    ├── 02_FocusOn.unity.meta
    ├── 03_InfiniteScroll.unity
    ├── 03_InfiniteScroll.unity.meta
    ├── 04_Metaball.unity
    ├── 04_Metaball.unity.meta
    ├── 05_Voronoi.unity
    ├── 05_Voronoi.unity.meta
    ├── 06_LoopTabBar.unity
    ├── 06_LoopTabBar.unity.meta
    ├── 07_ScrollRect.unity
    ├── 07_ScrollRect.unity.meta
    ├── 08_GridView.unity
    ├── 08_GridView.unity.meta
    ├── 09_LoadTexture.unity
    ├── 09_LoadTexture.unity.meta
    └── Sources/
        ├── 01_Basic/
        ├── 01_Basic.meta
        ├── 02_FocusOn/
        ├── 02_FocusOn.meta
        ├── 03_InfiniteScroll/
        ├── 03_InfiniteScroll.meta
        ├── 04_Metaball/
        ├── 04_Metaball.meta
        ├── 05_Voronoi/
        ├── 05_Voronoi.meta
        ├── 06_LoopTabBar/
        ├── 06_LoopTabBar.meta
        ├── 07_ScrollRect/
        ├── 07_ScrollRect.meta
        ├── 08_GridView/
        ├── 08_GridView.meta
        ├── 09_LoadTexture/
        ├── 09_LoadTexture.meta
        ├── Common/
        └── Common.meta
```

## ローカルパッケージの現在のディレクトリ構造

このリポジトリでは、アップストリームとは異なる構造を採用しています：

```
Packages/jp.setchi.fancyscrollview/
├── package.json
├── LICENSE
├── LICENSE.meta
├── README.md
├── README.md.meta
├── Documentation~/              # アップストリームには存在しない（UPM標準のドキュメント配置）
│   └── modifications/
│       ├── branch-strategy.md
│       ├── modifications.md
│       ├── upm-migration-summary.md
│       └── upm-structure-reference.md (このファイル)
├── Runtime/                     # アップストリームは Sources/Runtime/
│   ├── Runtime.meta
│   ├── FancyScrollView.asmdef
│   ├── FancyScrollView.asmdef.meta
│   ├── Core/
│   ├── Scroller/
│   ├── ScrollRect/
│   └── GridView/
└── Editor/                      # アップストリームは Sources/Editor/
    ├── Editor.meta
    ├── FancyScrollView.Editor.asmdef
    ├── FancyScrollView.Editor.asmdef.meta
    ├── ScrollerEditor.cs
    └── ScrollerEditor.cs.meta

Assets/FancyScrollView/
└── Examples/                    # アップストリームは Samples~/
    ├── 01_Basic.unity
    ├── 02_FocusOn.unity
    ├── ... (その他のサンプル)
    ├── 01_Basic/
    ├── 02_FocusOn/
    └── ... (その他のサンプルソース)
```

### 主な違い

1. **Sources/ ディレクトリの省略**: Runtime/ と Editor/ をパッケージルート直下に配置
2. **Documentation~/ の追加**: UPM標準のドキュメントディレクトリを追加
3. **サンプルの場所**: Samples~/ ではなく Assets/FancyScrollView/Examples/ に配置（開発とテストを容易にするため）

## 主要なファイルと内容

### package.json（アップストリーム）

```json
{
  "name": "jp.setchi.fancyscrollview",
  "displayName": "FancyScrollView",
  "description": "A scrollview component that can implement highly flexible animation.",
  "version": "1.9.0",
  "unity": "2019.4",
  "license": "MIT",
  "author": "setchi",
  "dependencies": {},
  "samples": [
    {
      "displayName": "Demo",
      "path": "Samples~"
    }
  ]
}
```

### package.json（ローカル）

```json
{
  "name": "jp.setchi.fancyscrollview",
  "displayName": "FancyScrollView",
  "description": "A scrollview component that can implement highly flexible animation.",
  "version": "1.9.0",
  "unity": "6000.0",
  "license": "MIT",
  "author": "setchi",
  "documentationUrl": "https://setchi.jp/FancyScrollView/",
  "unityRelease": "64f1"
}
```

**差異**: ローカル版では Unity 6.0.64f1 を最小要件とし、`samples` フィールドを削除（サンプルは Assets に配置）、`documentationUrl` と `unityRelease` を追加。

### アセンブリ定義ファイル

#### Runtime/FancyScrollView.asmdef

```json
{
	"name": "FancyScrollView"
}
```

#### Editor/FancyScrollView.Editor.asmdef

```json
{
    "name": "FancyScrollView.Editor",
    "references": [
        "FancyScrollView"
    ],
    "optionalUnityReferences": [],
    "includePlatforms": [
        "Editor"
    ],
    "excludePlatforms": [],
    "allowUnsafeCode": false,
    "overrideReferences": false,
    "precompiledReferences": [],
    "autoReferenced": true,
    "defineConstraints": []
}
```

## 重要な GUID

### ルートレベル

| ファイル | GUID |
|------|------|
| LICENSE.meta | `572727be5c765482ca314fb2581d9392` |
| README.md.meta | `96981b0d2ee8645cc94d70563bda2999` |
| package.json.meta | `1c75442fd24454d849623c0a0701b45c` |
| Sources.meta（アップストリームのみ） | `d19ac89da996d1843aafe828e87e81ac` |

### Sources ディレクトリ（アップストリーム）/ ルート直下（ローカル）

| ファイル | GUID | 備考 |
|------|------|------|
| Sources/Runtime.meta（アップストリーム）<br>Runtime.meta（ローカル） | `dd41a20ffdea64364905efac7fb4aa9b` | ✅ 一致 |
| Sources/Editor.meta（アップストリーム）<br>Editor.meta（ローカル） | `3b7adbc21b737494d970fcbd5dbe1175` | ✅ 一致 |

### アセンブリ定義

| ファイル | GUID | 備考 |
|------|------|------|
| Runtime/FancyScrollView.asmdef.meta | `59f6770f7492c42ff827a64ac010ac49` | ✅ 一致 |
| Editor/FancyScrollView.Editor.asmdef.meta | `310eb4609be1c41459e5ebb1bde5ac5a` | ✅ 一致 |

### Runtime モジュールディレクトリ

| ファイル | GUID | 備考 |
|------|------|------|
| Runtime/Core.meta | `aca07f0cadcbc467ca910639faf19bd9` | ✅ 一致 |
| Runtime/Scroller.meta | `691a9f5ee4aec4112a01b9a4332cac51` | ✅ 一致 |
| Runtime/ScrollRect.meta | `e2c5beedd885c4490a86bb4973f965bf` | ✅ 一致 |
| Runtime/GridView.meta | `c7768f2982b0142ab876d2bb4b597646` | ✅ 一致 |

### サンプルディレクトリ

| ファイル | GUID |
|------|------|
| Samples~/Sources.meta（アップストリームのみ） | `411d74237da48401ab1e38e01a92fd6d` |
| Samples~/01_Basic.unity.meta（アップストリームのみ） | `f5666b6c719a9b544ab322e1066aef5f` |

**注記**: ローカル版ではサンプルを `Assets/FancyScrollView/Examples/` に配置しているため、Samples~/ の GUID は使用していません。

## マスターブランチとの主な違い

UPM ブランチは、マスターブランチといくつかの重要な点で異なります：

1. **ディレクトリ構造**
   - ルートにはパッケージファイルのみを含む（package.json、LICENSE、README、Sources、Samples~）
   - Assets/ ディレクトリは存在しない
   - サンプルは Samples~ に配置（UPM 規約のチルダ接尾辞）

2. **パッケージメタデータ**
   - UPM登録のために package.json を含む
   - すべてのファイルに対応する .meta ファイルと GUID が存在

3. **アセンブリ定義**
   - クリーンなコンパイルのためにアセンブリ定義ファイル（.asmdef）を使用
   - Runtime と Editor コードで別々のアセンブリ

4. **サンプルの配布**
   - Samples~ ディレクトリは、ユーザーがインポートするまで Unity により非表示
   - ユーザーは Package Manager UI 経由でサンプルをインポート
   - サンプルにはシーンとソースコードの両方が含まれる

## ローカルでの再現時の注意事項

この構造をローカルで再現する場合：

1. すべての .meta ファイルは、アセット参照を維持するために正確な GUID で保存する必要があります
2. Samples~ ディレクトリ（チルダ付き）は UPM 規約です - Unity はインポートされるまでこれを非表示にします
3. アセンブリ定義ファイルにより、高速なコンパイルと明確な依存関係が可能になります
4. package.json により Unity Package Manager 経由での検出が可能になります
5. UPM が有効なパッケージとして認識するには、フォルダ構造が正確に一致する必要があります

## ローカルパッケージのカスタマイズ

このリポジトリでは、開発とテストを容易にするために以下のカスタマイズを行っています：

1. **Sources/ ディレクトリの省略**: Runtime/ と Editor/ をパッケージルート直下に配置し、構造を簡素化
2. **Documentation~/ の追加**: UPM標準に従い、ドキュメントをパッケージ内に配置
3. **サンプルの配置変更**:
   - Samples~/ ではなく Assets/FancyScrollView/Examples/ に配置
   - Unity エディタで直接開いてテスト可能
   - サンプルの編集と実行が容易
4. **Unity バージョン**: Unity 6.0.64f1 を最小要件として指定（アップストリームは 2019.4）

## 参照

- [アップストリーム UPM ブランチ](https://github.com/setchi/FancyScrollView/tree/upm)
- [OpenUPM のパッケージ](https://openupm.com/packages/jp.setchi.fancyscrollview/)
- [Unity Package Manager ドキュメント](https://docs.unity3d.com/Manual/Packages.html)
- [UPM Package Layout](https://docs.unity3d.com/Manual/cus-layout.html)
