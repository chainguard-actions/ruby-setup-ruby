<!-- markdownlint-disable -->

# Hardening Report: ruby--setup-ruby/v1.313.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ruby--setup-ruby/v1.313.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across every workflow file use mutable tag or version refs instead of immutable 40-character SHA digests. This exposes the action to supply-chain attacks if any upstream action is compromised or its tag is moved. Failing references include: `actions/checkout@v6` (used in all four workflow files) and `peter-evans/create-pull-request@v8` (check-new-windows-versions.yml). None of the 27 `uses:` references are SHA-pinned.

Locations:

- `.github/workflows/test.yml:21`
- `.github/workflows/test.yml:33`
- `.github/workflows/release.yml:11`
- `.github/workflows/check-new-windows-versions.yml:11`
- `.github/workflows/check-new-windows-versions.yml:27`
- `.github/workflows/update-v1-branch.yml:9`

### script-injection (severity: high)

Two `run:` blocks directly interpolate GitHub Actions expressions (`${{ ... }}`) into shell commands, enabling script injection. (a) In test.yml, `${{ matrix.expected_rubygems_version }}` is interpolated directly into a Ruby one-liner shell command: `run: ruby -e 'puts Gem::VERSION; exit(Gem.rubygems_version >= Gem::Version.new("${{ matrix.expected_rubygems_version }}"))'`. A matrix value controlled by workflow configuration is injected into the shell before quoting. (b) In check-new-windows-versions.yml, `${{ steps.pr.outputs.pull-request-url }}` is interpolated directly into an echo command: `run: echo "${{ steps.pr.outputs.pull-request-url }}"`. Step outputs can contain attacker-influenced content (e.g. from PR metadata) and should be passed via env vars instead.

Locations:

- `.github/workflows/test.yml:163`
- `.github/workflows/check-new-windows-versions.yml:42`

### missing-permissions (severity: medium)

The workflow file `check-new-windows-versions.yml` has no top-level `permissions:` key and no job-level `permissions:` key on its only job (`check_windows_versions`). Without an explicit permissions block, the workflow inherits the repository's default token permissions, which may be broader than necessary (e.g. write access to contents). A minimal permissions block such as `permissions: {}` or scoped permissions (e.g. `contents: read`) should be added.

Locations:

- `.github/workflows/check-new-windows-versions.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

1. Pinned all `actions/checkout@v6` references to SHA `d23441a48e516b6c34aea4fa41551a30e30af803` across all four workflow files (test.yml had 23 occurrences, plus one each in release.yml, update-v1-branch.yml, and check-new-windows-versions.yml). Pinned `peter-evans/create-pull-request@v8` to SHA `5f6978faf089d4d20b00c7766989d076bb2fc7f1` in check-new-windows-versions.yml. All original tags preserved as inline comments.
2. Fixed script injection in test.yml by moving `${{ matrix.expected_rubygems_version }}` into an `env:` block as `EXPECTED_RUBYGEMS_VERSION` and referencing it via `ENV["EXPECTED_RUBYGEMS_VERSION"]` in the Ruby one-liner. Fixed script injection in check-new-windows-versions.yml by moving `${{ steps.pr.outputs.pull-request-url }}` into an `env:` block as `PR_URL` and referencing it as `$PR_URL`.
3. Added `permissions: contents: read` top-level block to check-new-windows-versions.yml.

