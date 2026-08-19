<!-- markdownlint-disable -->

# Hardening Report: ruby--setup-ruby/v1.311.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ruby--setup-ruby/v1.311.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across all workflow files use mutable version tags instead of pinned 40-character SHA commits, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved or compromised. Failing references include: `actions/checkout@v6` (used in test.yml, release.yml, check-new-windows-versions.yml, update-v1-branch.yml) and `peter-evans/create-pull-request@v8` (used in check-new-windows-versions.yml).

Locations:

- `.github/workflows/test.yml:18`
- `.github/workflows/test.yml:20`
- `.github/workflows/release.yml:10`
- `.github/workflows/check-new-windows-versions.yml:11`
- `.github/workflows/check-new-windows-versions.yml:28`
- `.github/workflows/update-v1-branch.yml:10`

### script-injection (severity: high)

Two `run:` blocks directly interpolate `${{ }}` expressions, allowing workflow-controllable values to be injected into shell commands before the shell ever sees them (sub-rule a).

1. `.github/workflows/check-new-windows-versions.yml`: `run: echo "${{ steps.pr.outputs.pull-request-url }}"` — `steps.*.outputs.*` is a workflow-controllable context interpolated directly into a shell command. A malicious pull-request URL containing shell metacharacters could lead to command injection.

2. `.github/workflows/test.yml`: `run: ruby -e 'puts Gem::VERSION; exit(Gem.rubygems_version >= Gem::Version.new("${{ matrix.expected_rubygems_version }}"))'` — `matrix.*` values are workflow-controllable and are interpolated directly into the shell command string.

Locations:

- `.github/workflows/check-new-windows-versions.yml:47`
- `.github/workflows/test.yml:163`

### missing-permissions (severity: medium)

`.github/workflows/check-new-windows-versions.yml` has no top-level `permissions:` key and its only job (`check_windows_versions`) also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g., `write` access to contents). Explicit minimal permissions should be declared.

Locations:

- `.github/workflows/check-new-windows-versions.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

1. Pinned all `actions/checkout@v6` references to SHA `d23441a48e516b6c34aea4fa41551a30e30af803` across test.yml (23 occurrences), release.yml, update-v1-branch.yml, and check-new-windows-versions.yml. Pinned `peter-evans/create-pull-request@v8` to SHA `5f6978faf089d4d20b00c7766989d076bb2fc7f1` in check-new-windows-versions.yml. 2. Fixed script injection in check-new-windows-versions.yml by moving `${{ steps.pr.outputs.pull-request-url }}` to an `env:` block (`PR_URL`) and referencing it as `$PR_URL` in the shell. Fixed script injection in test.yml by moving `${{ matrix.expected_rubygems_version }}` to an `env:` block (`EXPECTED_RUBYGEMS_VERSION`) and updating the Ruby one-liner to read from `ENV["EXPECTED_RUBYGEMS_VERSION"]`. 3. Added `permissions: contents: read` top-level block to check-new-windows-versions.yml (the workflow only reads the repo to check for new versions; the PR creation uses a separate token from secrets).

