<!-- markdownlint-disable -->

# Hardening Report: HatsuneMiku3939--direnv-action/v1.4.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **HatsuneMiku3939--direnv-action/v1.4.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of full 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the referenced tag or branch is moved to a malicious commit.

check-dirty.yml: actions/checkout@v7, actions/setup-node@v7
codeql.yml: actions/checkout@v7, github/codeql-action/init@v4, github/codeql-action/autobuild@v4, github/codeql-action/analyze@v4, advanced-security/filter-sarif@v1, github/codeql-action/upload-sarif@v4, actions/upload-artifact@v7
contributors.yml: actions/checkout@v7, BobAnkh/add-contributors@master
test.yaml: actions/checkout@v7, actions/setup-node@v7 (multiple jobs)
unittest.yml: actions/checkout@v7, actions/setup-node@v7 (multiple jobs)

Locations:

- `.github/workflows/check-dirty.yml:10`
- `.github/workflows/codeql.yml:30`
- `.github/workflows/contributors.yml:10`
- `.github/workflows/test.yaml:18`
- `.github/workflows/unittest.yml:14`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the default (often broad) repository token permissions, violating the principle of least privilege.

- check-dirty.yml: no permissions at top level or in the check-dirty job
- contributors.yml: no permissions at top level or in the add-contributors job
- test.yaml: no permissions at top level or in any of the four jobs (test, test-default-option, test-required-missing, test-checksum-mismatch)
- unittest.yml: no permissions at top level or in any of the three jobs (units, test, test-default-option)

Locations:

- `.github/workflows/check-dirty.yml:1`
- `.github/workflows/contributors.yml:1`
- `.github/workflows/test.yaml:1`
- `.github/workflows/unittest.yml:1`

### script-injection (severity: high)

Two `run:` steps in test.yaml directly interpolate `${{ steps.*.outcome }}` expressions into shell commands (sub-rule a). Although `steps.*.outcome` values are GitHub-controlled (limited to 'success', 'failure', 'cancelled', 'skipped'), any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the template engine before the shell ever sees it, bypassing shell quoting.

Offending lines:
  - `run: test "${{ steps.missing_required.outcome }}" = "failure"`
  - `run: test "${{ steps.checksum_mismatch.outcome }}" = "failure"`

Fix: move the value into an env var and reference it as a quoted shell variable, e.g.:
  env:
    OUTCOME: ${{ steps.missing_required.outcome }}
  run: test "$OUTCOME" = "failure"

Locations:

- `.github/workflows/test.yaml:56`
- `.github/workflows/test.yaml:73`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all findings across 5 workflow files:

1. unpinned-uses: Pinned all 9 action references to full 40-char commit SHAs with original tags preserved in comments. Actions pinned: actions/checkout@v7, actions/setup-node@v7, github/codeql-action/{init,autobuild,analyze,upload-sarif}@v4, advanced-security/filter-sarif@v1, actions/upload-artifact@v7, BobAnkh/add-contributors@master.

2. missing-permissions: Added top-level `permissions: contents: read` to check-dirty.yml, test.yaml, and unittest.yml. Added `permissions: contents: write` + `pull-requests: write` to contributors.yml (needed for the add-contributors action to create PRs). codeql.yml already had job-level permissions and was not flagged.

3. script-injection: Fixed both offending lines in test.yaml (lines 56 and 73) by moving `${{ steps.missing_required.outcome }}` and `${{ steps.checksum_mismatch.outcome }}` into step-level `env:` blocks as `OUTCOME`, then referencing `$OUTCOME` in the shell `run:` command.

