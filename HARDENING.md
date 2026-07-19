<!-- markdownlint-disable -->

# Hardening Report: ruby--setup-ruby/v1.316.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ruby--setup-ruby/v1.316.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All four workflow files use tag-based (non-SHA-pinned) `uses:` references, making them vulnerable to supply-chain attacks if the referenced action tags are moved. Failing references include: `actions/checkout@v7` (used in test.yml, release.yml, check-new-windows-versions.yml, update-v1-branch.yml) and `peter-evans/create-pull-request@v8` (check-new-windows-versions.yml). All should be pinned to full 40-character commit SHAs.

Locations:

- `.github/workflows/test.yml:21`
- `.github/workflows/test.yml:38`
- `.github/workflows/release.yml:11`
- `.github/workflows/check-new-windows-versions.yml:11`
- `.github/workflows/check-new-windows-versions.yml:27`
- `.github/workflows/update-v1-branch.yml:9`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside `run:` shell command strings, allowing script injection. (1) In check-new-windows-versions.yml line 42: `run: echo "${{ steps.pr.outputs.pull-request-url }}"` — the step output (which may contain attacker-influenced content such as a PR URL with embedded shell metacharacters) is injected directly into the shell command. (2) In test.yml around line 162: `run: ruby -e 'puts Gem::VERSION; exit(Gem.rubygems_version >= Gem::Version.new("${{ matrix.expected_rubygems_version }}"))' ` — the matrix value `matrix.expected_rubygems_version` is interpolated directly into the shell command string. Both should be moved to `env:` variables and referenced as quoted shell variables.

Locations:

- `.github/workflows/check-new-windows-versions.yml:42`
- `.github/workflows/test.yml:162`

### missing-permissions (severity: medium)

The workflow file `check-new-windows-versions.yml` has no top-level `permissions:` key and no job-level `permissions:` key on its only job (`check_windows_versions`). Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. A minimal `permissions:` block (e.g., `contents: read`) should be added.

Locations:

- `.github/workflows/check-new-windows-versions.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across four workflow files:

1. **unpinned-uses**: Pinned all `actions/checkout@v7` references to SHA `9c091bb21b7c1c1d1991bb908d89e4e9dddfe3e0 # v7` in test.yml (multiple occurrences), release.yml, check-new-windows-versions.yml, and update-v1-branch.yml. Pinned `peter-evans/create-pull-request@v8` to SHA `5f6978faf089d4d20b00c7766989d076bb2fc7f1 # v8` in check-new-windows-versions.yml.

2. **script-injection**: (a) In check-new-windows-versions.yml, moved `${{ steps.pr.outputs.pull-request-url }}` into an `env:` block as `PR_URL` and referenced it as `"$PR_URL"` in the shell command. (b) In test.yml, moved `${{ matrix.expected_rubygems_version }}` into an `env:` block as `EXPECTED_RUBYGEMS_VERSION` and referenced it via `ENV["EXPECTED_RUBYGEMS_VERSION"]` in the Ruby one-liner.

3. **missing-permissions**: Added `permissions: contents: read` top-level block to check-new-windows-versions.yml.

