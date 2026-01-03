---
paths: .github/workflows/**/*.{yml,yaml}
---

# GitHub Actions Rules

## Security Requirements

### Action Pinning
- **MUST** pin actions to commit SHA (not tags)
- Format: `uses: org/action@<commit-sha> # vX.Y.Z`
- Example: `uses: actions/checkout@8e8c483db84b4bee98b60c0593521ed34d9990e8 # v6.0.1`

### Permissions
- **MUST** set `permissions: {}` at workflow level
- **MUST** grant minimum required permissions per job
- Example:
  ```yaml
  permissions: {}
  jobs:
    build:
      permissions:
        contents: read
  ```

### Timeout
- **MUST** set `timeout-minutes` for all jobs
- Default: `60` minutes
- Prevents infinite execution

### Input Validation
- **MUST** validate user inputs (tags, branch names, etc.)
- Use regex validation for version tags:
  ```bash
  if [[ ! "$VERSION" =~ ^[0-9]+\.[0-9]+\.[0-9]+-fork\.[0-9]+$ ]]; then
    echo "❌ Invalid version format"
    exit 1
  fi
  ```

### Secrets
- **MUST NOT** expose secrets in logs
- Use `env:` block, not inline parameters
- Verify third-party actions don't leak secrets

## Version Requirements

### Current Versions (2026-01-03)
- `actions/checkout`: v6.0.1 (`8e8c483db84b4bee98b60c0593521ed34d9990e8`)
- `actions/cache`: v5.0.1 (`9255dc7a253b0ccc959486e2bca901246202afeb`)
- `actions/upload-artifact`: v6.0.0 (`b7c566a772e6b6bfb58ed0dc250532a479d7789f`)
- `game-ci/unity-builder`: v4.8.1 (`1d4ee0697f193f54668e98961d79907911f4b4f2`)
- `softprops/action-gh-release`: v2.5.0 (`a06a81a03ee405af7f2048a818ed3f03bbf83c7b`)

### Deprecated
- `actions/cache@v1`, `@v2` - Discontinued 2024-12-05

## Workflow Standards

### Naming
- Use descriptive job/step names in English or Japanese
- Include emoji for visual clarity (📦, ✅, ❌, 📊, 📁)

### Efficiency
- Use `concurrency` to cancel outdated runs
- Cache Unity `Library/` folder with content hash keys
- Run independent jobs in parallel

### Triggers
- `pull_request`: For PR validation
- `push`: For main branch commits
- `workflow_dispatch`: For manual testing
- `tags`: For releases (format: `v*.*.*-fork.*`)

## File Structure

```
.github/
├── workflows/
│   ├── build-samples.yml   # PR/push validation
│   └── release.yml         # Tag-based releases
└── dependabot.yml          # Auto-update actions weekly
```

## Example Template

```yaml
name: Example Workflow

on:
  push:
    branches: [develop]

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

permissions: {}

jobs:
  example:
    runs-on: ubuntu-latest
    timeout-minutes: 60
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@8e8c483db84b4bee98b60c0593521ed34d9990e8 # v6.0.1
      - name: Run task
        run: echo "Task completed"
```

## Dependabot Integration

Dependabot auto-updates action versions weekly (Saturdays 09:00 JST).
Configuration: `.github/dependabot.yml`

When updating actions:
1. Review Dependabot PR
2. Verify commit SHA matches version tag
3. Check for breaking changes in changelog
4. Merge if tests pass
