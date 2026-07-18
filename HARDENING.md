<!-- markdownlint-disable -->

# Hardening Report: ruby--setup-ruby/v1.301.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ruby--setup-ruby/v1.301.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in workflow files are pinned to mutable version tags rather than immutable 40-character commit SHAs, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved. Failing references include: `actions/checkout@v6` (used in all 4 workflow files) and `peter-evans/create-pull-request@v8` (in check-new-windows-versions.yml).

Locations:

- `.github/workflows/check-new-windows-versions.yml:12`
- `.github/workflows/check-new-windows-versions.yml:27`
- `.github/workflows/release.yml:10`
- `.github/workflows/test.yml:21`
- `.github/workflows/update-v1-branch.yml:10`

### missing-permissions (severity: medium)

`check-new-windows-versions.yml` has no top-level `permissions:` key and its only job (`check_windows_versions`) also has no job-level `permissions:` key. This means the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/check-new-windows-versions.yml:1`

### script-injection (severity: high)

Sub-rule (a): A `${{ ... }}` expression is interpolated directly inside a `run:` shell command string. (1) In `test.yml`, the step `run: ruby -e 'puts Gem::VERSION; exit(Gem.rubygems_version >= Gem::Version.new("${{ matrix.expected_rubygems_version }}"))'` injects the matrix context value directly into a shell command. (2) In `check-new-windows-versions.yml`, the step `run: echo "${{ steps.pr.outputs.pull-request-url }}"` injects a step output directly into a shell command. Both allow YAML template substitution to occur before the shell parses the string, enabling potential command injection.

Locations:

- `.github/workflows/test.yml:175`
- `.github/workflows/check-new-windows-versions.yml:40`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across four workflow files:

1. **unpinned-uses**: Pinned all `actions/checkout@v6` references (23 occurrences in test.yml, 1 each in release.yml, update-v1-branch.yml, check-new-windows-versions.yml) to SHA `df4cb1c069e1874edd31b4311f1884172cec0e10 # v6`. Pinned `peter-evans/create-pull-request@v8` to SHA `5f6978faf089d4d20b00c7766989d076bb2fc7f1 # v8`.

2. **missing-permissions**: Added `permissions: contents: read` top-level block to `check-new-windows-versions.yml`.

3. **script-injection**: (a) In `test.yml`, moved `${{ matrix.expected_rubygems_version }}` into an `env:` block as `EXPECTED_RUBYGEMS_VERSION` and referenced it via `ENV["EXPECTED_RUBYGEMS_VERSION"]` in the Ruby one-liner. (b) In `check-new-windows-versions.yml`, moved `${{ steps.pr.outputs.pull-request-url }}` into an `env:` block as `PR_URL` and referenced it as `$PR_URL` in the shell command.

