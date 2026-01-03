---
paths: **/*.md, Packages/jp.setchi.fancyscrollview/package.json, .github/workflows/**/*.{yml,yaml}
---

# Versioning Rules

This document defines versioning rules for the FancyScrollView fork.

## Policy

This repository is a fork of [setchi/FancyScrollView](https://github.com/setchi/FancyScrollView). To distinguish from upstream, all versions use the **`-fork`** suffix.

### Format

```
X.Y.Z-fork.W
```

- **X.Y.Z**: Semantic version (same as upstream)
- **-fork.W**: Pre-release identifier per [SemVer 2.0.0](https://semver.org/)
  - `fork`: Fork identifier
  - `W`: Fork-specific version number (0, 1, 2, ...)

### Examples

| Upstream | This Fork | Description |
|---|---|---|
| `v1.9.0` | `v1.9.0-fork.0` | Initial fork release based on upstream v1.9.0 |
| `v1.9.0` | `v1.9.0-fork.1` | Fork-specific fix/feature on top of v1.9.0 |
| `v1.9.0` | `v1.9.0-fork.2` | Second fork-specific update on top of v1.9.0 |
| `v2.0.0` | `v2.0.0-fork.0` | Initial fork release based on upstream v2.0.0 |

## UPM Package Version

[package.json](../../Packages/jp.setchi.fancyscrollview/package.json) includes `-fork.W` suffix:

```json
{
  "name": "jp.setchi.fancyscrollview",
  "version": "1.9.0-fork.0"
}
```

## Git Tags

### Naming Convention

```bash
v<major>.<minor>.<patch>-fork.<number>
```

### Release Creation

1. Update `package.json` version
2. Commit changes
3. Create and push tag:
   ```bash
   git tag v1.9.0-fork.0
   git push origin v1.9.0-fork.0
   ```
4. GitHub Actions automatically:
   - Builds Windows executable
   - Creates GitHub Release
   - Uploads build artifacts (ZIP)

## GitHub Actions

### Release Workflow

[`.github/workflows/release.yml`](../../.github/workflows/release.yml) triggers on `-fork` suffixed tags:

```yaml
on:
  push:
    tags:
      - 'v*.*.*-fork'
```

### Version Extraction

```bash
# Tag: v1.9.0-fork.0
# Extracted: 1.9.0-fork.0
VERSION=${GITHUB_REF#refs/tags/v}
```

Output filename: `FancyScrollView-1.9.0-fork.0-Windows.zip`

## Upstream Sync Strategy

### Workflow

1. **Fetch upstream updates**
   ```bash
   git checkout master
   git fetch upstream
   git merge upstream/master
   git push origin master
   ```

2. **Merge to develop**
   ```bash
   git checkout develop
   git merge master
   ```

3. **Resolve conflicts and test**

4. **Update version**
   - Upstream releases `v2.0.0`
   - This fork updates to `v2.0.0-fork.0`
   - Update `package.json` and release

### Version Number Decision

- **Major/Minor/Patch**: Same as upstream
- **Fork number (W)**: Reset to `.0` when syncing with upstream, increment for fork-specific changes

#### When syncing with upstream:
- Upstream: `v1.9.0` → `v1.10.0`
- This fork: `v1.9.0-fork.3` → `v1.10.0-fork.0` (reset)

#### When making fork-specific changes:
- This fork: `v1.9.0-fork.0` → `v1.9.0-fork.1` → `v1.9.0-fork.2` (increment)

## Semantic Versioning Compliance

`-fork.W` functions as a **pre-release identifier** per [SemVer 2.0.0](https://semver.org/).

Version precedence (from highest to lowest):
```
1.9.0 > 1.9.0-fork.2 > 1.9.0-fork.1 > 1.9.0-fork.0
```

The `-fork.W` suffix accurately reflects that this is a variant of the upstream stable release, with the fork number tracking fork-specific changes.

## Benefits

- ✅ Clear distinction from upstream
- ✅ SemVer compliant
- ✅ Easy to track upstream updates
- ✅ Consistent with UPM
- ✅ Track fork-specific changes with fork number

## Troubleshooting

### Incorrect Tag

```bash
# Delete local tag
git tag -d v1.9.0-fork.0

# Delete remote tag
git push origin :refs/tags/v1.9.0-fork.0

# Recreate correct tag
git tag v1.9.0-fork.0
git push origin v1.9.0-fork.0
```

### Release Not Created

1. Verify tag matches `v*.*.*-fork` pattern
2. Check GitHub Actions workflow status
3. Verify Secrets configuration:
   - `UNITY_LICENSE`
   - `UNITY_EMAIL`
   - `UNITY_PASSWORD`

### Version Mismatch

`package.json` and Git tag versions must match:

- ❌ Wrong: `package.json: 1.9.0-fork.0` / Tag: `v2.0.0-fork.0`
- ✅ Correct: `package.json: 1.9.0-fork.0` / Tag: `v1.9.0-fork.0`
