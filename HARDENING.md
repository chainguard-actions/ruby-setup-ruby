<!-- markdownlint-disable -->

# Hardening Report: ruby--setup-ruby/v1.292.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ruby--setup-ruby/v1.292.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across all workflow files use mutable tags instead of pinned 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved. Failing references include: `actions/checkout@v6` (used in all 4 workflow files), `peter-evans/create-pull-request@v8` (check-new-windows-versions.yml), and `./` local action references (test.yml, release.yml). None of these are pinned to a full commit SHA.

Locations:

- `.github/workflows/test.yml:17`
- `.github/workflows/test.yml:19`
- `.github/workflows/test.yml:30`
- `.github/workflows/release.yml:10`
- `.github/workflows/release.yml:12`
- `.github/workflows/check-new-windows-versions.yml:11`
- `.github/workflows/check-new-windows-versions.yml:27`
- `.github/workflows/update-v1-branch.yml:10`

### missing-permissions (severity: medium)

The workflow file `check-new-windows-versions.yml` has no top-level `permissions:` key and no job-level `permissions:` key on its only job (`check_windows_versions`). Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/check-new-windows-versions.yml:1`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are directly interpolated inside `run:` shell command strings, allowing an attacker to inject arbitrary shell commands.

1. In `check-new-windows-versions.yml`: `run: echo "${{ steps.pr.outputs.pull-request-url }}"` — the `steps.pr.outputs.pull-request-url` value (which could be influenced by a PR title or branch name) is interpolated directly into the shell command without quoting or sanitization.

2. In `test.yml`: `run: ruby -e 'puts Gem::VERSION; exit(Gem.rubygems_version >= Gem::Version.new("${{ matrix.expected_rubygems_version }}"))'` — the `matrix.expected_rubygems_version` value is interpolated directly into the shell command string. Matrix values flow through YAML template substitution before the shell processes them.

Locations:

- `.github/workflows/check-new-windows-versions.yml:40`
- `.github/workflows/test.yml:185`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all findings across 4 workflow files:

1. **unpinned-uses**: Pinned all `actions/checkout@v6` references to SHA `d23441a48e516b6c34aea4fa41551a30e30af803 # v6` in test.yml (21 occurrences), release.yml, check-new-windows-versions.yml, and update-v1-branch.yml. Pinned `peter-evans/create-pull-request@v8` to SHA `5f6978faf089d4d20b00c7766989d076bb2fc7f1 # v8` in check-new-windows-versions.yml. Local `./` references are not pinnable as they reference the local action.

2. **missing-permissions**: Added `permissions: contents: read` top-level block to `check-new-windows-versions.yml`.

3. **script-injection**: (a) In `test.yml`: moved `${{ matrix.expected_rubygems_version }}` into an `env:` block as `EXPECTED_RUBYGEMS_VERSION` and updated the Ruby one-liner to use `ENV["EXPECTED_RUBYGEMS_VERSION"]`. (b) In `check-new-windows-versions.yml`: moved `${{ steps.pr.outputs.pull-request-url }}` into an `env:` block as `PR_URL` and updated the echo command to use `"$PR_URL"`.

