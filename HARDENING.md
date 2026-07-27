<!-- markdownlint-disable -->

# Hardening Report: HatsuneMiku3939--direnv-action/v1.4.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **HatsuneMiku3939--direnv-action/v1.4.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files reference actions using mutable tags or branch names instead of full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks. Unpinned references found:
- check-dirty.yml: actions/checkout@v7, actions/setup-node@v7
- codeql.yml: actions/checkout@v7, github/codeql-action/init@v4, github/codeql-action/autobuild@v4, github/codeql-action/analyze@v4, advanced-security/filter-sarif@v1, github/codeql-action/upload-sarif@v4, actions/upload-artifact@v7
- contributors.yml: actions/checkout@v7, BobAnkh/add-contributors@master
- test.yaml: actions/checkout@v7, actions/setup-node@v7
- unittest.yml: actions/checkout@v7, actions/setup-node@v7

Locations:

- `.github/workflows/check-dirty.yml:9`
- `.github/workflows/codeql.yml:30`
- `.github/workflows/contributors.yml:9`
- `.github/workflows/test.yaml:19`
- `.github/workflows/unittest.yml:13`

### missing-permissions (severity: medium)

Four workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the default (potentially broad) repository permissions. Affected files: check-dirty.yml, contributors.yml, test.yaml, unittest.yml.

Locations:

- `.github/workflows/check-dirty.yml:1`
- `.github/workflows/contributors.yml:1`
- `.github/workflows/test.yaml:1`
- `.github/workflows/unittest.yml:1`

### script-injection (severity: high)

Two `run:` steps in test.yaml directly interpolate `${{ steps.*.outcome }}` expressions into shell commands (sub-rule a). The `steps.*` context is workflow-controllable and any expression interpolated directly into a `run:` block is a script-injection risk — the value flows through YAML template substitution before the shell sees it. Offending lines:
- `run: test "${{ steps.missing_required.outcome }}" = "failure"`
- `run: test "${{ steps.checksum_mismatch.outcome }}" = "failure"`
These should be moved to an `env:` block and referenced as shell variables.

Locations:

- `.github/workflows/test.yaml:57`
- `.github/workflows/test.yaml:76`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all five workflow files: (1) Pinned all action references to full 40-char SHAs with tag comments preserved. (2) Added top-level permissions blocks to check-dirty.yml (contents: read), test.yaml (contents: read), unittest.yml (contents: read), and contributors.yml (contents: write, pull-requests: write). codeql.yml already had job-level permissions. (3) Fixed script injection in test.yaml lines 57 and 76 by moving ${{ steps.missing_required.outcome }} and ${{ steps.checksum_mismatch.outcome }} into env: blocks and referencing them as $OUTCOME shell variables.

