<!-- markdownlint-disable -->

# Hardening Report: ruby--setup-ruby/v1.315.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ruby--setup-ruby/v1.315.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions by mutable version tags instead of pinned 40-character commit SHAs. Failing references include: `actions/checkout@v7` (used in all four workflow files) and `peter-evans/create-pull-request@v8` (in check-new-windows-versions.yml). These should be pinned to full SHA digests to prevent supply-chain attacks.

Locations:

- `.github/workflows/check-new-windows-versions.yml:8`
- `.github/workflows/check-new-windows-versions.yml:27`
- `.github/workflows/release.yml:10`
- `.github/workflows/test.yml:14`
- `.github/workflows/update-v1-branch.yml:9`

### missing-permissions (severity: medium)

The workflow file `check-new-windows-versions.yml` has no top-level `permissions:` key and its only job (`check_windows_versions`) also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the default (potentially broad) repository permissions.

Locations:

- `.github/workflows/check-new-windows-versions.yml:1`

### script-injection (severity: high)

Two `run:` blocks directly interpolate GitHub Actions expressions (`${{ ... }}`) into shell commands, violating sub-rule (a).

1. `.github/workflows/check-new-windows-versions.yml`: `run: echo "${{ steps.pr.outputs.pull-request-url }}"` — the `steps.pr.outputs.pull-request-url` value is interpolated directly into the shell command. A malicious PR URL containing shell metacharacters could lead to command injection.

2. `.github/workflows/test.yml`: `run: ruby -e 'puts Gem::VERSION; exit(Gem.rubygems_version >= Gem::Version.new("${{ matrix.expected_rubygems_version }}"))'` — the `matrix.expected_rubygems_version` value is interpolated directly into the shell command. Matrix values are workflow-controllable and must not be interpolated inline.

Locations:

- `.github/workflows/check-new-windows-versions.yml:40`
- `.github/workflows/test.yml:163`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings across four workflow files:

1. **unpinned-uses**: Pinned all `actions/checkout@v7` references (23 occurrences across check-new-windows-versions.yml, test.yml, release.yml, update-v1-branch.yml) to SHA `3d3c42e5aac5ba805825da76410c181273ba90b1`. Pinned `peter-evans/create-pull-request@v8` to SHA `5f6978faf089d4d20b00c7766989d076bb2fc7f1`.

2. **missing-permissions**: Added top-level `permissions: { contents: read, pull-requests: write }` to `check-new-windows-versions.yml` (minimum needed for git checkout and PR creation).

3. **script-injection**: (a) In `check-new-windows-versions.yml`, moved `${{ steps.pr.outputs.pull-request-url }}` to an `env:` block as `PR_URL` and used `echo "$PR_URL"` in the shell. (b) In `test.yml`, moved `${{ matrix.expected_rubygems_version }}` to an `env:` block as `EXPECTED_RUBYGEMS_VERSION` and referenced it via `ENV["EXPECTED_RUBYGEMS_VERSION"]` in the Ruby one-liner.

