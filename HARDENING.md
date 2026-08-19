<!-- markdownlint-disable -->

# Hardening Report: ruby--setup-ruby/v1.310.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ruby--setup-ruby/v1.310.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags (e.g. @v6, @v8) instead of pinned 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the tag is moved to a different commit. Affected references include: `actions/checkout@v6` (used in all four workflow files) and `peter-evans/create-pull-request@v8` (in check-new-windows-versions.yml).

Locations:

- `.github/workflows/test.yml:17`
- `.github/workflows/test.yml:33`
- `.github/workflows/release.yml:13`
- `.github/workflows/update-v1-branch.yml:9`
- `.github/workflows/check-new-windows-versions.yml:10`
- `.github/workflows/check-new-windows-versions.yml:27`

### missing-permissions (severity: medium)

The workflow file `.github/workflows/check-new-windows-versions.yml` has no top-level `permissions:` key and no job-level `permissions:` key on its only job (`check_windows_versions`). Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/check-new-windows-versions.yml:1`

### script-injection (severity: high)

Sub-rule (a): A `${{ }}` expression is interpolated directly inside a `run:` shell command string, allowing the value to be parsed by the shell before quoting can protect it.

1. In `test.yml`: the `run:` block interpolates `${{ matrix.expected_rubygems_version }}` directly into a Ruby one-liner: `run: ruby -e 'puts Gem::VERSION; exit(Gem.rubygems_version >= Gem::Version.new("${{ matrix.expected_rubygems_version }}"))'`. A matrix value controlled by the workflow caller could inject shell metacharacters.

2. In `check-new-windows-versions.yml`: the `run:` block interpolates `${{ steps.pr.outputs.pull-request-url }}` directly: `run: echo "${{ steps.pr.outputs.pull-request-url }}"`  — step outputs are workflow-controllable and must not be interpolated directly into shell commands.

Locations:

- `.github/workflows/test.yml:163`
- `.github/workflows/check-new-windows-versions.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all findings across 4 workflow files:

1. **unpinned-uses**: Pinned all `actions/checkout@v6` references to SHA `df4cb1c069e1874edd31b4311f1884172cec0e10` (in test.yml, release.yml, update-v1-branch.yml, check-new-windows-versions.yml) and `peter-evans/create-pull-request@v8` to SHA `5f6978faf089d4d20b00c7766989d076bb2fc7f1` (in check-new-windows-versions.yml). Original tags preserved as comments.

2. **missing-permissions**: Added `permissions: contents: read` top-level block to check-new-windows-versions.yml.

3. **script-injection**: (a) In test.yml, moved `${{ matrix.expected_rubygems_version }}` out of the Ruby one-liner `run:` command into an `env:` block as `EXPECTED_RUBYGEMS_VERSION`, referenced via `ENV["EXPECTED_RUBYGEMS_VERSION"]` in the Ruby script. (b) In check-new-windows-versions.yml, moved `${{ steps.pr.outputs.pull-request-url }}` out of the `run: echo` command into an `env:` block as `PR_URL`, referenced as `$PR_URL` in the shell command.

