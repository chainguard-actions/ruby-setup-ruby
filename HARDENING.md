<!-- markdownlint-disable -->

# Hardening Report: ruby--setup-ruby/v1.320.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ruby--setup-ruby/v1.320.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of full 40-character commit SHA pins. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit. Affected references: `actions/checkout@v7` (all four workflow files) and `peter-evans/create-pull-request@v8` (check-new-windows-versions.yml). All should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/check-new-windows-versions.yml:9`
- `.github/workflows/check-new-windows-versions.yml:27`
- `.github/workflows/release.yml:9`
- `.github/workflows/test.yml:17`
- `.github/workflows/update-v1-branch.yml:9`

### missing-permissions (severity: medium)

The workflow file `.github/workflows/check-new-windows-versions.yml` has no top-level `permissions:` key and its only job (`check_windows_versions`) also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository default (typically `write-all` for private repos or `read-all` for public repos), granting broader access than necessary. A minimal permissions block such as `permissions: contents: read` should be added.

Locations:

- `.github/workflows/check-new-windows-versions.yml:1`

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks directly interpolate `${{ ... }}` expressions into shell command strings, bypassing shell quoting and enabling script injection if the values contain shell metacharacters.

1. `.github/workflows/test.yml`: The run step `ruby -e 'puts Gem::VERSION; exit(Gem.rubygems_version >= Gem::Version.new("${{ matrix.expected_rubygems_version }}"))'` injects `${{ matrix.expected_rubygems_version }}` directly into a shell command. A matrix value containing a single-quote or shell metacharacter would break out of the Ruby string literal and execute arbitrary shell code.

2. `.github/workflows/check-new-windows-versions.yml`: The run step `echo "${{ steps.pr.outputs.pull-request-url }}"` injects a step output directly into a shell command. Step outputs can contain attacker-influenced data (e.g. from PR branch names) and should be passed via an `env:` variable instead.

Locations:

- `.github/workflows/test.yml:155`
- `.github/workflows/check-new-windows-versions.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings:

1. **unpinned-uses**: Pinned all `actions/checkout@v7` references (23 occurrences across test.yml, release.yml, update-v1-branch.yml, check-new-windows-versions.yml) to SHA `3d3c42e5aac5ba805825da76410c181273ba90b1 # v7`. Pinned `peter-evans/create-pull-request@v8` to SHA `5f6978faf089d4d20b00c7766989d076bb2fc7f1 # v8`.

2. **missing-permissions**: Added `permissions: contents: read` top-level block to `check-new-windows-versions.yml`.

3. **script-injection**: (a) In `test.yml`, moved `${{ matrix.expected_rubygems_version }}` into an `env:` block as `EXPECTED_RUBYGEMS_VERSION` and referenced it via `ENV["EXPECTED_RUBYGEMS_VERSION"]` in the Ruby one-liner. (b) In `check-new-windows-versions.yml`, moved `${{ steps.pr.outputs.pull-request-url }}` into an `env:` block as `PR_URL` and referenced it as `$PR_URL` in the echo command.

