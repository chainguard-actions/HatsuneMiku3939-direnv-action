<!-- markdownlint-disable -->

# Hardening Report: HatsuneMiku3939--direnv-action/v1.3.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **HatsuneMiku3939--direnv-action/v1.3.6** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All uses: references in check-dirty.yml use mutable tags instead of pinned 40-character SHA hashes: `actions/checkout@v7` (line 13), `actions/setup-node@v6` (line 14). These can be silently updated by the upstream maintainer, enabling supply-chain attacks.

Locations:

- `.github/workflows/check-dirty.yml:13`
- `.github/workflows/check-dirty.yml:14`

### unpinned-uses (severity: high)

All uses: references in codeql.yml use mutable tags instead of pinned 40-character SHA hashes: `actions/checkout@v7` (line 30), `github/codeql-action/init@v4` (line 34), `github/codeql-action/autobuild@v4` (line 46), `github/codeql-action/analyze@v4` (line 55), `advanced-security/filter-sarif@v1` (line 61), `github/codeql-action/upload-sarif@v4` (line 68), `actions/upload-artifact@v7` (line 73).

Locations:

- `.github/workflows/codeql.yml:30`
- `.github/workflows/codeql.yml:34`
- `.github/workflows/codeql.yml:46`
- `.github/workflows/codeql.yml:55`
- `.github/workflows/codeql.yml:61`
- `.github/workflows/codeql.yml:68`
- `.github/workflows/codeql.yml:73`

### unpinned-uses (severity: high)

All uses: references in contributors.yml use mutable tags/branches instead of pinned 40-character SHA hashes: `actions/checkout@v7` (line 11), `BobAnkh/add-contributors@master` (line 12). The @master ref is especially dangerous as it tracks a moving branch head.

Locations:

- `.github/workflows/contributors.yml:11`
- `.github/workflows/contributors.yml:12`

### unpinned-uses (severity: high)

All uses: references in test.yaml use mutable tags instead of pinned 40-character SHA hashes: `actions/checkout@v7` (lines 17, 37, 49) and `actions/setup-node@v6` (lines 18, 38, 50).

Locations:

- `.github/workflows/test.yaml:17`
- `.github/workflows/test.yaml:18`
- `.github/workflows/test.yaml:37`
- `.github/workflows/test.yaml:38`
- `.github/workflows/test.yaml:49`
- `.github/workflows/test.yaml:50`

### unpinned-uses (severity: high)

All uses: references in unittest.yml use mutable tags instead of pinned 40-character SHA hashes: `actions/checkout@v7` and `actions/setup-node@v6` appear in multiple jobs without SHA pinning.

Locations:

- `.github/workflows/unittest.yml:14`
- `.github/workflows/unittest.yml:15`
- `.github/workflows/unittest.yml:25`
- `.github/workflows/unittest.yml:26`
- `.github/workflows/unittest.yml:38`
- `.github/workflows/unittest.yml:39`

### missing-permissions (severity: medium)

check-dirty.yml has no top-level `permissions:` key and no job-level `permissions:` key on the `check-dirty` job. Without explicit permissions, the workflow inherits the default repository permissions, which may be overly broad.

Locations:

- `.github/workflows/check-dirty.yml:1`

### missing-permissions (severity: medium)

contributors.yml has no top-level `permissions:` key and no job-level `permissions:` key on the `add-contributors` job. This workflow uses GITHUB_TOKEN and runs on a schedule/workflow_dispatch, making explicit minimal permissions especially important.

Locations:

- `.github/workflows/contributors.yml:1`

### missing-permissions (severity: medium)

test.yaml has no top-level `permissions:` key and none of its three jobs (`test`, `test-default-option`, `test-required-missing`) have job-level `permissions:` keys.

Locations:

- `.github/workflows/test.yaml:1`

### missing-permissions (severity: medium)

unittest.yml has no top-level `permissions:` key and none of its three jobs (`units`, `test`, `test-default-option`) have job-level `permissions:` keys.

Locations:

- `.github/workflows/unittest.yml:1`

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is interpolated directly inside a `run:` shell command. In test.yaml the step `run: test "${{ steps.missing_required.outcome }}" = "failure"` embeds `${{ steps.missing_required.outcome }}` directly into the shell command string. Although `steps.*.outputs.*` is not attacker-controlled in this specific case, any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the template engine before the shell ever sees it, bypassing shell quoting. The value should be passed via an `env:` variable and referenced as `"$ENV_VAR"` instead.

Locations:

- `.github/workflows/test.yaml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all 5 workflow files:

1. **check-dirty.yml**: Pinned actions/checkout@v7 → SHA 9c091bb21b7c1c1d1991bb908d89e4e9dddfe3e0, actions/setup-node@v6 → SHA 249970729cb0ef3589644e2896645e5dc5ba9c38. Added top-level `permissions: contents: read`.

2. **codeql.yml**: Pinned actions/checkout@v7, github/codeql-action/init@v4, github/codeql-action/autobuild@v4, github/codeql-action/analyze@v4 (all → SHA 7188fc363630916deb702c7fdcf4e481b751f97a), advanced-security/filter-sarif@v1 → SHA f3b8118a9349d88f7b1c0c488476411145b6270d, github/codeql-action/upload-sarif@v4 → SHA 7188fc363630916deb702c7fdcf4e481b751f97a, actions/upload-artifact@v7 → SHA 043fb46d1a93c77aae656e7c1c64a875d1fc6a0a. Already had job-level permissions.

3. **contributors.yml**: Pinned actions/checkout@v7 → SHA 9c091bb21b7c1c1d1991bb908d89e4e9dddfe3e0, BobAnkh/add-contributors@master → SHA 8bd02ce413bee09225ff2f27791ecb78ea273743. Added top-level `permissions: contents: write, pull-requests: write` (needed for creating PRs to update README).

4. **test.yaml**: Pinned actions/checkout@v7 and actions/setup-node@v6 in all 3 jobs. Added top-level `permissions: contents: read`. Fixed script-injection: moved `${{ steps.missing_required.outcome }}` into an `env:` block as `OUTCOME` and referenced it as `"$OUTCOME"` in the shell command.

5. **unittest.yml**: Pinned actions/checkout@v7 and actions/setup-node@v6 in all 3 jobs. Added top-level `permissions: contents: read`.

