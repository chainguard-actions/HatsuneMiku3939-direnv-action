<!-- markdownlint-disable -->

# Hardening Report: HatsuneMiku3939--direnv-action/v1.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **HatsuneMiku3939--direnv-action/v1.3.1** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in check-dirty.yml use mutable version tags instead of pinned 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if the referenced action tags are moved. Failing references: `actions/checkout@v6`, `actions/setup-node@v6`.

Locations:

- `.github/workflows/check-dirty.yml:10`
- `.github/workflows/check-dirty.yml:11`

### unpinned-uses (severity: high)

All `uses:` references in codeql.yml use mutable version tags instead of pinned 40-character commit SHAs. Failing references: `actions/checkout@v6`, `github/codeql-action/init@v4`, `github/codeql-action/autobuild@v4`, `github/codeql-action/analyze@v4`, `advanced-security/filter-sarif@v1`, `github/codeql-action/upload-sarif@v4`, `actions/upload-artifact@v7`.

Locations:

- `.github/workflows/codeql.yml:30`
- `.github/workflows/codeql.yml:34`
- `.github/workflows/codeql.yml:47`
- `.github/workflows/codeql.yml:55`
- `.github/workflows/codeql.yml:63`
- `.github/workflows/codeql.yml:71`
- `.github/workflows/codeql.yml:77`

### unpinned-uses (severity: high)

All `uses:` references in contributors.yml use mutable version tags/branches instead of pinned 40-character commit SHAs. Notably, `BobAnkh/add-contributors@master` is pinned to a branch, which is especially risky. Failing references: `actions/checkout@v6`, `BobAnkh/add-contributors@master`.

Locations:

- `.github/workflows/contributors.yml:10`
- `.github/workflows/contributors.yml:11`

### unpinned-uses (severity: high)

All `uses:` references in test.yaml use mutable version tags instead of pinned 40-character commit SHAs. Failing references: `actions/checkout@v6` (×3), `actions/setup-node@v6` (×3).

Locations:

- `.github/workflows/test.yaml:18`
- `.github/workflows/test.yaml:19`
- `.github/workflows/test.yaml:38`
- `.github/workflows/test.yaml:39`
- `.github/workflows/test.yaml:52`
- `.github/workflows/test.yaml:53`

### unpinned-uses (severity: high)

All `uses:` references in unittest.yml use mutable version tags instead of pinned 40-character commit SHAs. Failing references: `actions/checkout@v6` (×3), `actions/setup-node@v6` (×3).

Locations:

- `.github/workflows/unittest.yml:13`
- `.github/workflows/unittest.yml:14`
- `.github/workflows/unittest.yml:24`
- `.github/workflows/unittest.yml:25`
- `.github/workflows/unittest.yml:38`
- `.github/workflows/unittest.yml:39`

### script-injection (severity: high)

Sub-rule (a): A `${{ }}` expression is interpolated directly inside a `run:` shell command. The expression `${{ steps.missing_required.outcome }}` is substituted into the shell command string before the shell executes it, allowing injection if the value contains shell metacharacters. Offending line: `run: test "${{ steps.missing_required.outcome }}" = "failure"`

Locations:

- `.github/workflows/test.yaml:62`

### permissions (severity: medium)

missing-permissions: check-dirty.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the workflow inherits the repository default (often `write-all`), granting excessive access.

Locations:

- `.github/workflows/check-dirty.yml:1`

### permissions (severity: medium)

missing-permissions: contributors.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the workflow inherits the repository default, granting excessive access.

Locations:

- `.github/workflows/contributors.yml:1`

### permissions (severity: medium)

missing-permissions: test.yaml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the workflow inherits the repository default, granting excessive access.

Locations:

- `.github/workflows/test.yaml:1`

### permissions (severity: medium)

missing-permissions: unittest.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the workflow inherits the repository default, granting excessive access.

Locations:

- `.github/workflows/unittest.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, permissions

**Notes:**

Fixed all 5 workflow files:

1. **check-dirty.yml**: Pinned `actions/checkout@v6` → SHA `d23441a48e516b6c34aea4fa41551a30e30af803` and `actions/setup-node@v6` → SHA `249970729cb0ef3589644e2896645e5dc5ba9c38`. Added `permissions: {}` top-level block.

2. **codeql.yml**: Pinned all 7 action references to full SHAs. Added `permissions: {}` top-level block (job already had specific permissions). codeql-action/* all resolved to SHA `7188fc363630916deb702c7fdcf4e481b751f97a`, filter-sarif to `f3b8118a9349d88f7b1c0c488476411145b6270d`, upload-artifact to `043fb46d1a93c77aae656e7c1c64a875d1fc6a0a`.

3. **contributors.yml**: Pinned `actions/checkout@v6` and `BobAnkh/add-contributors@master` → SHA `8bd02ce413bee09225ff2f27791ecb78ea273743`. Added `permissions: contents: write, pull-requests: write` (needed for the contributor action to update README and create PRs).

4. **test.yaml**: Pinned all 6 action references. Added `permissions: {}` top-level block. Fixed script-injection: moved `${{ steps.missing_required.outcome }}` out of the `run:` shell string into an `env:` block as `OUTCOME`, referenced as `$OUTCOME` in the shell command.

5. **unittest.yml**: Pinned all 6 action references. Added `permissions: {}` top-level block.

