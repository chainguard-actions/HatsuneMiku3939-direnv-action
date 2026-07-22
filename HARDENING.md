<!-- markdownlint-disable -->

# Hardening Report: HatsuneMiku3939--direnv-action/v1.4.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **HatsuneMiku3939--direnv-action/v1.4.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files use tag-based or branch-based `uses:` references instead of pinned 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if the referenced action tags are moved or compromised.

Failing references:
- check-dirty.yml: `actions/checkout@v7`, `actions/setup-node@v7`
- codeql.yml: `actions/checkout@v7`, `github/codeql-action/init@v4`, `github/codeql-action/autobuild@v4`, `github/codeql-action/analyze@v4`, `advanced-security/filter-sarif@v1`, `github/codeql-action/upload-sarif@v4`, `actions/upload-artifact@v7`
- contributors.yml: `actions/checkout@v7`, `BobAnkh/add-contributors@master`
- test.yaml: `actions/checkout@v7`, `actions/setup-node@v7` (multiple jobs)
- unittest.yml: `actions/checkout@v7`, `actions/setup-node@v7` (multiple jobs)

Locations:

- `.github/workflows/check-dirty.yml:10`
- `.github/workflows/codeql.yml:30`
- `.github/workflows/contributors.yml:8`
- `.github/workflows/test.yaml:18`
- `.github/workflows/unittest.yml:12`

### missing-permissions (severity: medium)

Four workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any of their jobs. Without explicit permissions, workflows run with the default (potentially broad) token permissions.

- check-dirty.yml: no permissions block at all
- contributors.yml: no permissions block at all
- test.yaml: no permissions block at all
- unittest.yml: no permissions block at all

(codeql.yml passes because it has job-level `permissions:` on its only job.)

Locations:

- `.github/workflows/check-dirty.yml:1`
- `.github/workflows/contributors.yml:1`
- `.github/workflows/test.yaml:1`
- `.github/workflows/unittest.yml:1`

### script-injection (severity: high)

Two `run:` steps in test.yaml directly interpolate `${{ steps.*.outcome }}` expressions inside shell command strings (sub-rule a). Although `steps.*.outcome` values are typically `success` or `failure`, any `${{ ... }}` expression interpolated directly into a `run:` block is a script-injection risk because the value flows through YAML template substitution before the shell parses it.

Offending lines:
- `run: test "${{ steps.missing_required.outcome }}" = "failure"`
- `run: test "${{ steps.checksum_mismatch.outcome }}" = "failure"`

Fix: move the value into an `env:` variable and reference it as a quoted shell variable, e.g.:
```yaml
env:
  OUTCOME: ${{ steps.missing_required.outcome }}
run: test "$OUTCOME" = "failure"
```

Locations:

- `.github/workflows/test.yaml:49`
- `.github/workflows/test.yaml:64`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all five workflow files:

**unpinned-uses**: Pinned all action references to full 40-char SHAs:
- actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1
- actions/setup-node@v7 → @820762786026740c76f36085b0efc47a31fe5020
- github/codeql-action/init@v4 → @e4fba868fa4b1b91e1fdab776edc8cfbe6e9fb81
- github/codeql-action/autobuild@v4 → @e4fba868fa4b1b91e1fdab776edc8cfbe6e9fb81
- github/codeql-action/analyze@v4 → @e4fba868fa4b1b91e1fdab776edc8cfbe6e9fb81
- github/codeql-action/upload-sarif@v4 → @e4fba868fa4b1b91e1fdab776edc8cfbe6e9fb81
- advanced-security/filter-sarif@v1 → @f3b8118a9349d88f7b1c0c488476411145b6270d
- actions/upload-artifact@v7 → @043fb46d1a93c77aae656e7c1c64a875d1fc6a0a
- BobAnkh/add-contributors@master → @8bd02ce413bee09225ff2f27791ecb78ea273743

**missing-permissions**: Added top-level `permissions:` blocks:
- check-dirty.yml: `contents: read`
- contributors.yml: `contents: write`, `pull-requests: write` (needed for the add-contributors action to push and create PRs)
- test.yaml: `contents: read`
- unittest.yml: `contents: read`

**script-injection**: Fixed two steps in test.yaml by moving `${{ steps.*.outcome }}` expressions into `env:` blocks and referencing them as `$OUTCOME` shell variables.

