<!-- markdownlint-disable -->

# Hardening Report: HatsuneMiku3939--direnv-action/v1.3.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **HatsuneMiku3939--direnv-action/v1.3.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across workflow files use mutable tag or branch refs instead of pinned 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved or compromised. Failing references include: actions/checkout@v6, actions/setup-node@v6, github/codeql-action/init@v4, github/codeql-action/autobuild@v4, github/codeql-action/analyze@v4, advanced-security/filter-sarif@v1, github/codeql-action/upload-sarif@v4, actions/upload-artifact@v7, BobAnkh/add-contributors@master.

Locations:

- `.github/workflows/test.yaml:14`
- `.github/workflows/test.yaml:16`
- `.github/workflows/test.yaml:31`
- `.github/workflows/test.yaml:33`
- `.github/workflows/test.yaml:47`
- `.github/workflows/test.yaml:49`
- `.github/workflows/check-dirty.yml:13`
- `.github/workflows/check-dirty.yml:15`
- `.github/workflows/codeql.yml:33`
- `.github/workflows/codeql.yml:38`
- `.github/workflows/codeql.yml:47`
- `.github/workflows/codeql.yml:65`
- `.github/workflows/codeql.yml:71`
- `.github/workflows/codeql.yml:77`
- `.github/workflows/codeql.yml:83`
- `.github/workflows/contributors.yml:10`
- `.github/workflows/contributors.yml:11`
- `.github/workflows/unittest.yml:14`
- `.github/workflows/unittest.yml:16`
- `.github/workflows/unittest.yml:27`
- `.github/workflows/unittest.yml:29`
- `.github/workflows/unittest.yml:41`
- `.github/workflows/unittest.yml:43`

### missing-permissions (severity: medium)

These workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows run with the default (potentially broad) token permissions. Each file should declare minimal required permissions.

Locations:

- `.github/workflows/test.yaml:1`
- `.github/workflows/check-dirty.yml:1`
- `.github/workflows/contributors.yml:1`
- `.github/workflows/unittest.yml:1`

### script-injection (severity: high)

Sub-rule (a): A `${{ ... }}` expression is interpolated directly inside a `run:` shell command string. In test.yaml, the step `run: test "${{ steps.missing_required.outcome }}" = "failure"` embeds a steps context expression directly into the shell command. Any `${{ ... }}` inside a `run:` block is a script-injection risk because the value is substituted into the shell command string before the shell parses it. The value should be passed via an `env:` variable and referenced as a quoted shell variable instead.

Locations:

- `.github/workflows/test.yaml:60`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across 5 workflow files:

1. unpinned-uses: Pinned all 9 action references to full 40-char SHA digests with original tag as comment. Actions pinned: actions/checkout@v6, actions/setup-node@v6, github/codeql-action/{init,autobuild,analyze,upload-sarif}@v4, advanced-security/filter-sarif@v1, actions/upload-artifact@v7, BobAnkh/add-contributors@master.

2. missing-permissions: Added top-level `permissions:` blocks to test.yaml (contents: read), check-dirty.yml (contents: read), contributors.yml (contents: write + pull-requests: write for PR creation), and unittest.yml (contents: read). codeql.yml already had job-level permissions.

3. script-injection: In test.yaml line 60, moved `${{ steps.missing_required.outcome }}` from the `run:` shell string into an `env:` block as OUTCOME, then referenced it as `"$OUTCOME"` in the shell command.

