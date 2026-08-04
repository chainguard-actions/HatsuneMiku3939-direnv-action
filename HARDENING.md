<!-- markdownlint-disable -->

# Hardening Report: HatsuneMiku3939--direnv-action/v1.4.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **HatsuneMiku3939--direnv-action/v1.4.5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags or version strings instead of full 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the tag is moved to a different commit.

check-dirty.yml: actions/checkout@v7, actions/setup-node@v7
codeql.yml: actions/checkout@v7, github/codeql-action/init@v4.37.4, github/codeql-action/autobuild@v4.37.4, github/codeql-action/analyze@v4.37.4, advanced-security/filter-sarif@v1, github/codeql-action/upload-sarif@v4.37.4, actions/upload-artifact@v7
contributors.yml: actions/checkout@v7, BobAnkh/add-contributors@master
test.yaml: actions/checkout@v7, actions/setup-node@v7
unittest.yml: actions/checkout@v7, actions/setup-node@v7

Locations:

- `.github/workflows/check-dirty.yml:11`
- `.github/workflows/check-dirty.yml:12`
- `.github/workflows/codeql.yml:30`
- `.github/workflows/codeql.yml:35`
- `.github/workflows/codeql.yml:50`
- `.github/workflows/codeql.yml:59`
- `.github/workflows/codeql.yml:63`
- `.github/workflows/codeql.yml:69`
- `.github/workflows/codeql.yml:74`
- `.github/workflows/contributors.yml:10`
- `.github/workflows/contributors.yml:11`
- `.github/workflows/test.yaml:20`
- `.github/workflows/test.yaml:21`
- `.github/workflows/unittest.yml:14`
- `.github/workflows/unittest.yml:15`

### missing-permissions (severity: medium)

Four workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any job. Without explicit permissions, workflows run with the default repository permissions, which may be overly broad (e.g. write access to contents).

Affected files: check-dirty.yml, contributors.yml, test.yaml, unittest.yml

Locations:

- `.github/workflows/check-dirty.yml:1`
- `.github/workflows/contributors.yml:1`
- `.github/workflows/test.yaml:1`
- `.github/workflows/unittest.yml:1`

### script-injection (severity: high)

Two `run:` steps in test.yaml directly interpolate `${{ steps.*.outcome }}` expressions inside shell commands. Although `steps.*.outcome` values are GitHub-controlled (typically 'success', 'failure', 'cancelled', or 'skipped'), any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection risk because the value is substituted into the shell command string before the shell parses it (sub-rule a). The safe pattern is to pass the value via an `env:` variable and reference it as a quoted shell variable.

Offending lines:
  - `run: test "${{ steps.missing_required.outcome }}" = "failure"`
  - `run: test "${{ steps.checksum_mismatch.outcome }}" = "failure"`

Locations:

- `.github/workflows/test.yaml:57`
- `.github/workflows/test.yaml:80`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across five workflow files:

1. unpinned-uses: Pinned all 9 unique action references to full 40-char commit SHAs with original tags preserved as comments. Actions pinned: actions/checkout@v7, actions/setup-node@v7, github/codeql-action/{init,autobuild,analyze,upload-sarif}@v4.37.4, advanced-security/filter-sarif@v1, actions/upload-artifact@v7, BobAnkh/add-contributors@master.

2. missing-permissions: Added top-level `permissions:` blocks to check-dirty.yml (contents: read), contributors.yml (contents: write + pull-requests: write, needed for the add-contributors action), test.yaml (contents: read), and unittest.yml (contents: read). codeql.yml already had job-level permissions.

3. script-injection: Fixed both offending run steps in test.yaml (lines 57 and 80) by moving `${{ steps.missing_required.outcome }}` and `${{ steps.checksum_mismatch.outcome }}` into `env:` blocks as OUTCOME variables, then referencing them as `"$OUTCOME"` in the shell commands.

