<!-- markdownlint-disable -->

# Hardening Report: HatsuneMiku3939--direnv-action/v1.3.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **HatsuneMiku3939--direnv-action/v1.3.7** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of immutable 40-character SHA digests, making them vulnerable to supply-chain attacks.

.github/workflows/check-dirty.yml: actions/checkout@v7, actions/setup-node@v6
.github/workflows/codeql.yml: actions/checkout@v7, github/codeql-action/init@v4, github/codeql-action/autobuild@v4, github/codeql-action/analyze@v4, advanced-security/filter-sarif@v1, github/codeql-action/upload-sarif@v4, actions/upload-artifact@v7
.github/workflows/contributors.yml: actions/checkout@v7, BobAnkh/add-contributors@master
.github/workflows/test.yaml: actions/checkout@v7, actions/setup-node@v6
.github/workflows/unittest.yml: actions/checkout@v7, actions/setup-node@v6

Locations:

- `.github/workflows/check-dirty.yml:10`
- `.github/workflows/check-dirty.yml:11`
- `.github/workflows/codeql.yml:32`
- `.github/workflows/codeql.yml:36`
- `.github/workflows/codeql.yml:46`
- `.github/workflows/codeql.yml:57`
- `.github/workflows/codeql.yml:63`
- `.github/workflows/codeql.yml:68`
- `.github/workflows/codeql.yml:73`
- `.github/workflows/contributors.yml:10`
- `.github/workflows/contributors.yml:11`
- `.github/workflows/test.yaml:17`
- `.github/workflows/test.yaml:18`
- `.github/workflows/unittest.yml:12`
- `.github/workflows/unittest.yml:13`

### missing-permissions (severity: medium)

Four workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any job. Without explicit permissions, workflows run with the default (potentially broad) token permissions. Affected files: check-dirty.yml, contributors.yml, test.yaml, unittest.yml.

Locations:

- `.github/workflows/check-dirty.yml:1`
- `.github/workflows/contributors.yml:1`
- `.github/workflows/test.yaml:1`
- `.github/workflows/unittest.yml:1`

### script-injection (severity: high)

Sub-rule (a): A `${{ }}` expression is interpolated directly inside a `run:` shell command string. In .github/workflows/test.yaml, the step `run: test "${{ steps.missing_required.outcome }}" = "failure"` embeds the expression directly in the shell command. Although `steps.*.outputs.*` is typically not attacker-controlled in this context, any `${{ ... }}` in a `run:` block is processed by the template engine before the shell sees it, bypassing shell quoting and enabling injection if the value ever contains shell metacharacters. The offending line: `run: test "${{ steps.missing_required.outcome }}" = "failure"`

Locations:

- `.github/workflows/test.yaml:62`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all 5 workflow files:

1. **unpinned-uses**: Pinned all action references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v7 → 3d3c42e5aac5ba805825da76410c181273ba90b1
   - actions/setup-node@v6 → 249970729cb0ef3589644e2896645e5dc5ba9c38
   - github/codeql-action/{init,autobuild,analyze,upload-sarif}@v4 → 7188fc363630916deb702c7fdcf4e481b751f97a
   - advanced-security/filter-sarif@v1 → f3b8118a9349d88f7b1c0c488476411145b6270d
   - actions/upload-artifact@v7 → 043fb46d1a93c77aae656e7c1c64a875d1fc6a0a
   - BobAnkh/add-contributors@master → 8bd02ce413bee09225ff2f27791ecb78ea273743

2. **missing-permissions**: Added top-level `permissions:` blocks to check-dirty.yml (contents: read), contributors.yml (contents: write + pull-requests: write), test.yaml (contents: read), and unittest.yml (contents: read). codeql.yml already had job-level permissions.

3. **script-injection**: In test.yaml line 62, moved `${{ steps.missing_required.outcome }}` from the `run:` shell string into an `env:` block as `OUTCOME`, then referenced it as `$OUTCOME` in the shell command.

