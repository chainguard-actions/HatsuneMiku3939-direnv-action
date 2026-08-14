<!-- markdownlint-disable -->

# Hardening Report: HatsuneMiku3939--direnv-action/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **HatsuneMiku3939--direnv-action/v1.4.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Every `uses:` reference across all workflow files uses a mutable tag or branch name instead of a pinned 40-character SHA digest, making the action vulnerable to supply-chain attacks if the referenced tag is moved or the repository is compromised.

Failing references:
- check-dirty.yml: `actions/checkout@v7`, `actions/setup-node@v6`
- codeql.yml: `actions/checkout@v7`, `github/codeql-action/init@v4`, `github/codeql-action/autobuild@v4`, `github/codeql-action/analyze@v4`, `advanced-security/filter-sarif@v1`, `github/codeql-action/upload-sarif@v4`, `actions/upload-artifact@v7`
- contributors.yml: `actions/checkout@v7`, `BobAnkh/add-contributors@master`
- test.yaml: `actions/checkout@v7`, `actions/setup-node@v6` (multiple jobs)
- unittest.yml: `actions/checkout@v7`, `actions/setup-node@v6` (multiple jobs)

Locations:

- `.github/workflows/check-dirty.yml:13`
- `.github/workflows/check-dirty.yml:14`
- `.github/workflows/codeql.yml:30`
- `.github/workflows/codeql.yml:34`
- `.github/workflows/codeql.yml:46`
- `.github/workflows/codeql.yml:57`
- `.github/workflows/codeql.yml:62`
- `.github/workflows/codeql.yml:67`
- `.github/workflows/codeql.yml:72`
- `.github/workflows/contributors.yml:11`
- `.github/workflows/contributors.yml:12`
- `.github/workflows/test.yaml:20`
- `.github/workflows/test.yaml:21`
- `.github/workflows/unittest.yml:14`
- `.github/workflows/unittest.yml:15`

### missing-permissions (severity: medium)

Four workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, workflows run with the default (often broad) token permissions, violating the principle of least privilege.

- check-dirty.yml: no permissions at any level
- contributors.yml: no permissions at any level
- test.yaml: no permissions at any level
- unittest.yml: no permissions at any level

(codeql.yml is the only file that correctly declares job-level permissions.)

Locations:

- `.github/workflows/check-dirty.yml:1`
- `.github/workflows/contributors.yml:1`
- `.github/workflows/test.yaml:1`
- `.github/workflows/unittest.yml:1`

### script-injection (severity: high)

Two `run:` steps in test.yaml directly interpolate `${{ steps.*.outcome }}` expressions into shell command strings (sub-rule a). Although `steps.*.outcome` is not directly attacker-controlled, any `${{ ... }}` expression inside a `run:` block is expanded by the template engine before the shell sees it, bypassing shell quoting. If the value ever contains shell metacharacters it would be executed. The safe pattern is to pass the value through an `env:` variable and reference it as a quoted shell variable.

Offending lines:
- `- run: test "${{ steps.missing_required.outcome }}" = "failure"`
- `- run: test "${{ steps.checksum_mismatch.outcome }}" = "failure"`

Locations:

- `.github/workflows/test.yaml:57`
- `.github/workflows/test.yaml:80`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all findings across 5 workflow files:

1. unpinned-uses: Pinned all `uses:` references to full 40-char SHAs with tag comments:
   - actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38
   - github/codeql-action/{init,autobuild,analyze,upload-sarif}@v4 → @e4fba868fa4b1b91e1fdab776edc8cfbe6e9fb81
   - advanced-security/filter-sarif@v1 → @f3b8118a9349d88f7b1c0c488476411145b6270d
   - actions/upload-artifact@v7 → @043fb46d1a93c77aae656e7c1c64a875d1fc6a0a
   - BobAnkh/add-contributors@master → @8bd02ce413bee09225ff2f27791ecb78ea273743

2. missing-permissions: Added top-level `permissions:` blocks:
   - check-dirty.yml: contents: read
   - contributors.yml: contents: write, pull-requests: write (needed for add-contributors)
   - test.yaml: contents: read
   - unittest.yml: contents: read

3. script-injection: Fixed two `run:` steps in test.yaml by moving `${{ steps.*.outcome }}` expressions into `env:` blocks as OUTCOME variables, then referencing `"$OUTCOME"` in the shell commands.

