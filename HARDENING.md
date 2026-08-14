<!-- markdownlint-disable -->

# Hardening Report: HatsuneMiku3939--direnv-action/v1.3.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **HatsuneMiku3939--direnv-action/v1.3.5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tags or branch names instead of full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks. Failing references include: check-dirty.yml: actions/checkout@v6, actions/setup-node@v6; codeql.yml: actions/checkout@v6, github/codeql-action/init@v4, github/codeql-action/autobuild@v4, github/codeql-action/analyze@v4, advanced-security/filter-sarif@v1, github/codeql-action/upload-sarif@v4, actions/upload-artifact@v7; contributors.yml: actions/checkout@v6, BobAnkh/add-contributors@master; test.yaml: actions/checkout@v6, actions/setup-node@v6 (multiple jobs); unittest.yml: actions/checkout@v6, actions/setup-node@v6 (multiple jobs).

Locations:

- `.github/workflows/check-dirty.yml:8`
- `.github/workflows/check-dirty.yml:9`
- `.github/workflows/codeql.yml:30`
- `.github/workflows/codeql.yml:34`
- `.github/workflows/codeql.yml:44`
- `.github/workflows/codeql.yml:54`
- `.github/workflows/codeql.yml:60`
- `.github/workflows/codeql.yml:68`
- `.github/workflows/codeql.yml:73`
- `.github/workflows/contributors.yml:9`
- `.github/workflows/contributors.yml:10`
- `.github/workflows/test.yaml:14`
- `.github/workflows/test.yaml:15`
- `.github/workflows/unittest.yml:12`
- `.github/workflows/unittest.yml:13`

### missing-permissions (severity: medium)

Four workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows run with the default (potentially broad) token permissions. Affected files: check-dirty.yml, contributors.yml, test.yaml, unittest.yml.

Locations:

- `.github/workflows/check-dirty.yml:1`
- `.github/workflows/contributors.yml:1`
- `.github/workflows/test.yaml:1`
- `.github/workflows/unittest.yml:1`

### script-injection (severity: high)

Sub-rule (a): A `run:` block in test.yaml directly interpolates a `${{ ... }}` expression inside a shell command string. The offending line is: `run: test "${{ steps.missing_required.outcome }}" = "failure"`. The `steps.*.outputs.*` / step context values flow through YAML template substitution before the shell sees them, making this a script-injection risk. The value should be passed via an `env:` variable and referenced as `"$ENV_VAR"` in the shell instead.

Locations:

- `.github/workflows/test.yaml:50`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across five workflow files:

1. unpinned-uses: Pinned all 9 unique action references to full 40-char SHAs with tag comments preserved. Actions pinned: actions/checkout@v6, actions/setup-node@v6, github/codeql-action/{init,autobuild,analyze,upload-sarif}@v4, advanced-security/filter-sarif@v1, actions/upload-artifact@v7, BobAnkh/add-contributors@master.

2. missing-permissions: Added top-level `permissions:` blocks to check-dirty.yml (contents: read), contributors.yml (contents: write + pull-requests: write), test.yaml (contents: read), and unittest.yml (contents: read). codeql.yml already had job-level permissions.

3. script-injection: In test.yaml, moved `${{ steps.missing_required.outcome }}` from the `run:` shell string into an `env:` block as `OUTCOME`, then referenced it as `"$OUTCOME"` in the shell command.

