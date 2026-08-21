<!-- markdownlint-disable -->

# Hardening Report: HatsuneMiku3939--direnv-action/v1.4.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **HatsuneMiku3939--direnv-action/v1.4.6** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or version strings instead of pinned 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the referenced tag is moved or overwritten.

Failing references:
- check-dirty.yml: actions/checkout@v7, actions/setup-node@v7
- codeql.yml: actions/checkout@v7, github/codeql-action/init@v4.37.7, github/codeql-action/autobuild@v4.37.7, github/codeql-action/analyze@v4.37.7, advanced-security/filter-sarif@v1, github/codeql-action/upload-sarif@v4.37.7, actions/upload-artifact@v7
- contributors.yml: actions/checkout@v7, BobAnkh/add-contributors@master
- test.yaml: actions/checkout@v7, actions/setup-node@v7 (multiple jobs)
- unittest.yml: actions/checkout@v7, actions/setup-node@v7 (multiple jobs)

Locations:

- `.github/workflows/check-dirty.yml:10`
- `.github/workflows/check-dirty.yml:11`
- `.github/workflows/codeql.yml:30`
- `.github/workflows/codeql.yml:36`
- `.github/workflows/codeql.yml:46`
- `.github/workflows/codeql.yml:56`
- `.github/workflows/codeql.yml:61`
- `.github/workflows/codeql.yml:67`
- `.github/workflows/codeql.yml:73`
- `.github/workflows/contributors.yml:9`
- `.github/workflows/contributors.yml:10`
- `.github/workflows/test.yaml:22`
- `.github/workflows/test.yaml:23`
- `.github/workflows/unittest.yml:15`
- `.github/workflows/unittest.yml:16`

### missing-permissions (severity: medium)

Four workflow files have no top-level `permissions:` key and no job-level `permissions:` on any of their jobs. Without explicit permissions, workflows inherit the default repository token permissions, which may be broader than necessary.

- check-dirty.yml: no permissions defined at any level
- contributors.yml: no permissions defined at any level
- test.yaml: no permissions defined at any level
- unittest.yml: no permissions defined at any level

Locations:

- `.github/workflows/check-dirty.yml:1`
- `.github/workflows/contributors.yml:1`
- `.github/workflows/test.yaml:1`
- `.github/workflows/unittest.yml:1`

### script-injection (severity: high)

Two `run:` steps in test.yaml directly interpolate `${{ steps.*.outcome }}` expressions (a `steps` context value) into shell command strings. Any `${{ ... }}` expression interpolated directly inside a `run:` block is a script-injection risk because the value is substituted by the template engine before the shell ever sees it, bypassing shell quoting.

Offending lines:
- `- run: test "${{ steps.missing_required.outcome }}" = "failure"` (sub-rule a: direct expression interpolation)
- `- run: test "${{ steps.checksum_mismatch.outcome }}" = "failure"` (sub-rule a: direct expression interpolation)

Fix: move the value into an `env:` variable and reference it as a quoted shell variable, e.g.:
```yaml
env:
  OUTCOME: ${{ steps.missing_required.outcome }}
run: test "$OUTCOME" = "failure"
```

Locations:

- `.github/workflows/test.yaml:63`
- `.github/workflows/test.yaml:80`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across five workflow files:

1. unpinned-uses: Pinned all action references to full 40-char commit SHAs with original tags as comments. Actions pinned: actions/checkout@v7, actions/setup-node@v7, github/codeql-action/{init,autobuild,analyze,upload-sarif}@v4.37.7, advanced-security/filter-sarif@v1, actions/upload-artifact@v7, BobAnkh/add-contributors@master.

2. missing-permissions: Added top-level permissions blocks to check-dirty.yml (contents: read), contributors.yml (contents: write + pull-requests: write for PR creation), test.yaml (contents: read), and unittest.yml (contents: read). codeql.yml already had job-level permissions.

3. script-injection: Fixed two run: steps in test.yaml that interpolated ${{ steps.*.outcome }} directly. Moved expressions to env: blocks (OUTCOME variable) and referenced as $OUTCOME in the shell command.

