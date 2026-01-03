---
paths: **/*.md
---

# Documentation Guidelines

This document defines documentation rules for the FancyScrollView project. These guidelines are optimized for AI consumption to reduce token usage.

## Language Policy

### Japanese Documentation (User & Developer Facing)
- All documentation in `Documentation~/user-guides/` and `docs/` MUST be in **Japanese**
- Target audience: Beginners to intermediate Unity developers
- Use clear, simple Japanese with abundant code examples

### English Documentation (AI-Optimized)
- `CLAUDE.md` and files in `.claude/` directory MUST be in **English**
- Optimized for AI/LLM token efficiency
- Concise, technical writing without beginner explanations
- No need for verbose descriptions

## Directory Structure

### 1. User Guides (Japanese)
**Location**: `Packages/jp.setchi.fancyscrollview/Documentation~/user-guides/`

**Content**:
- Feature usage tutorials
- Implementation guides
- Code examples with Japanese comments
- Troubleshooting sections

**Example Structure**:
```
Documentation~/user-guides/
├── getting-started.md
├── basic-implementation.md
├── using-context.md
├── infinite-scroll.md
├── scrollrect-usage.md
├── gridview-usage.md
└── custom-animations.md
```

### 2. Developer Guides (Japanese)
**Location**: `docs/`

**Content**:
- Development environment setup
- Coding conventions
- Architecture details
- API reference
- Build/release procedures

**Example Structure**:
```
docs/
├── README.md
├── development-guide.md
├── architecture.md
├── api-reference.md
└── branch-strategy.md
```

### 3. Upstream Modifications (Japanese)
**Location**: `Packages/jp.setchi.fancyscrollview/Documentation~/modifications/`

**Content**:
- Differences from upstream (setchi/FancyScrollView)
- Customization rationale
- UPM migration records
- Merge considerations

**Example Structure**:
```
Documentation~/modifications/
├── modifications.md
├── upm-migration-summary.md
└── upm-structure-reference.md
```

### 4. AI Instructions (English)
**Location**: `.claude/` and `CLAUDE.md`

**Content**:
- Project structure overview
- Development commands
- Architecture summary
- File organization
- Branch strategy summary

**Key Requirements**:
- **English only**
- **Concise technical writing**
- **Token-efficient format**
- Use bullet points, tables, code blocks
- No beginner-friendly explanations needed

## User Guide Template (Japanese)

```markdown
# [機能名]

[1-2文の概要]

## 概要

[機能の説明]

## 実装方法

### ステップ1: [ステップ名]

```csharp
// 日本語コメント
public class Example { }
```

### ステップ2: [ステップ名]

...

## 完全な実装例

[動作するコード]

## トラブルシューティング

### 問題: [問題]

**原因**: [原因]
**解決策**: [解決方法]

## まとめ

- [ポイント1]
- [ポイント2]
```

## Developer Guide Template (Japanese)

```markdown
# [トピック名]

## 概要

[説明]

## [セクション]

[詳細]

## ベストプラクティス

1. [推奨事項]

## 参考資料

- [リンク]
```

## AI Instruction Template (English)

```markdown
# Project Name

Brief description in 1-2 sentences.

## Structure

```
path/to/
├── component1/
└── component2/
```

## Key Commands

- Command 1: Description
- Command 2: Description

## Architecture

- Component A: Purpose
- Component B: Purpose

## Notes

- Important note 1
- Important note 2
```

## Code Example Rules

### Japanese Documentation
- **All code comments in Japanese**
- Include complete working examples
- Use Bad/Good comparisons when appropriate

```csharp
// Good: 理由の説明
public void GoodExample()
{
    // 推奨実装
}

// Bad: 理由の説明
public void BadExample()
{
    // 非推奨
}
```

### English Documentation (AI)
- Minimal or no comments
- Focus on structure over explanation

```csharp
// Concise comment if necessary
public class Example {
    void Method() { }
}
```

## Update Triggers

Update documentation when:
1. **New feature**: Add user guide (Japanese)
2. **API change**: Update reference + related docs (Japanese)
3. **Bug fix**: Update troubleshooting if needed (Japanese)
4. **Upstream sync**: Record in modifications/ (Japanese)
5. **Project structure change**: Update CLAUDE.md (English)

## Checklist

### New User Guide (Japanese)
- [ ] Written in Japanese
- [ ] Beginner-friendly explanations
- [ ] Working code examples
- [ ] Japanese code comments
- [ ] Troubleshooting section
- [ ] Placed in `user-guides/`
- [ ] Added to README.md

### New Developer Guide (Japanese)
- [ ] Written in Japanese
- [ ] Technically accurate
- [ ] Follows coding conventions
- [ ] Placed in `docs/`
- [ ] Added to README.md

### AI Instruction Update (English)
- [ ] Written in English
- [ ] Concise and token-efficient
- [ ] No beginner explanations
- [ ] Updated CLAUDE.md or `.claude/` files

### Upstream Modification Record (Japanese)
- [ ] Written in Japanese
- [ ] Clear diff from upstream
- [ ] Includes rationale
- [ ] Placed in `modifications/`

## Summary

**3 Core Principles**:

1. **Language**: Japanese for users/developers, English for AI
2. **Location**: user-guides/ | docs/ | modifications/ | .claude/
3. **Style**: Beginner-friendly (Japanese) | Token-efficient (English)

Japanese docs prioritize clarity and examples. English docs prioritize conciseness and token efficiency.
