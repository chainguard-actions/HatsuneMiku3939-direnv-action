<!-- markdownlint-disable -->

# Hardening Report: HatsuneMiku3939--direnv-action/v1.3.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **HatsuneMiku3939--direnv-action/v1.3.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is directly interpolated inside a run: shell command. The line `run: test "${{ steps.missing_required.outcome }}" = "failure"` injects the steps context value directly into the shell command string before the shell ever sees it, enabling script injection if the value contains shell metacharacters.

Locations:

- `.github/workflows/test.yaml:57`

### unpinned-uses (severity: high)

Multiple uses: references are pinned to mutable tags or branches instead of immutable 40-character SHA digests, making the workflow vulnerable to supply-chain attacks. Failing references include: actions/checkout@v6, actions/setup-node@v6 (check-dirty.yml, test.yaml, unittest.yml); github/codeql-action/init@v4, github/codeql-action/autobuild@v4, github/codeql-action/analyze@v4, advanced-security/filter-sarif@v1, github/codeql-action/upload-sarif@v4, actions/upload-artifact@v7 (codeql.yml); BobAnkh/add-contributors@master (contributors.yml).

Locations:

- `.github/workflows/check-dirty.yml:13`
- `.github/workflows/codeql.yml:36`
- `.github/workflows/contributors.yml:9`
- `.github/workflows/test.yaml:15`
- `.github/workflows/unittest.yml:15`

### missing-permissions (severity: medium)

These workflow files have no top-level permissions: key and no job-level permissions: key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially write) permissions, violating least-privilege. Each file should declare a top-level permissions: block with only the minimal scopes required.

Locations:

- `.github/workflows/check-dirty.yml:1`
- `.github/workflows/contributors.yml:1`
- `.github/workflows/test.yaml:1`
- `.github/workflows/unittest.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across 5 workflow files: (1) script-injection in test.yaml line 57 — moved ${{ steps.missing_required.outcome }} into an env: block as OUTCOME and referenced it as $OUTCOME in the shell; (2) unpinned-uses — pinned all 9 action references to full 40-char SHA digests with tag comments preserved (actions/checkout@v6, actions/setup-node@v6, github/codeql-action/{init,autobuild,analyze,upload-sarif}@v4, advanced-security/filter-sarif@v1, actions/upload-artifact@v7, BobAnkh/add-contributors@master); (3) missing-permissions — added top-level permissions blocks to check-dirty.yml (contents:read), contributors.yml (contents:write + pull-requests:write for PR creation), test.yaml (contents:read), unittest.yml (contents:read), and codeql.yml (contents:read at top-level, existing job-level permissions preserved).

