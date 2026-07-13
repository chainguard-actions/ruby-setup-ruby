<!-- markdownlint-disable -->

# Hardening Report: ruby--setup-ruby/v1.301.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **ruby--setup-ruby/v1.301.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All uses: references across all workflow files use mutable version tags (e.g. @v6, @v8) instead of full 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks if the referenced action tags are moved. Affected references include: actions/checkout@v6 (multiple occurrences in test.yml, release.yml, update-v1-branch.yml, check-new-windows-versions.yml), peter-evans/create-pull-request@v8 (check-new-windows-versions.yml), and uses: ./ (local action, not applicable). All external action references must be pinned to a full SHA.

Locations:

- `.github/workflows/test.yml:18`
- `.github/workflows/release.yml:11`
- `.github/workflows/update-v1-branch.yml:11`
- `.github/workflows/check-new-windows-versions.yml:9`
- `.github/workflows/check-new-windows-versions.yml:26`

### missing-permissions (severity: medium)

The workflow file check-new-windows-versions.yml has no top-level permissions: key and no job-level permissions: key on its only job (check_windows_versions). Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/check-new-windows-versions.yml:1`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. (1) In test.yml, the run: block interpolates ${{ matrix.expected_rubygems_version }} directly inside a ruby -e shell string: `run: ruby -e 'puts Gem::VERSION; exit(Gem.rubygems_version >= Gem::Version.new("${{ matrix.expected_rubygems_version }}"))'`. A malicious matrix value could inject arbitrary shell or Ruby code. (2) In check-new-windows-versions.yml, the run: block interpolates ${{ steps.pr.outputs.pull-request-url }} directly: `run: echo "${{ steps.pr.outputs.pull-request-url }}"`. A crafted pull-request URL could inject shell metacharacters.

Locations:

- `.github/workflows/test.yml:161`
- `.github/workflows/check-new-windows-versions.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

1. unpinned-uses: Pinned all actions/checkout@v6 references (23 occurrences in test.yml, 1 in release.yml, 1 in update-v1-branch.yml, 1 in check-new-windows-versions.yml) to SHA df4cb1c069e1874edd31b4311f1884172cec0e10, and peter-evans/create-pull-request@v8 in check-new-windows-versions.yml to SHA 5f6978faf089d4d20b00c7766989d076bb2fc7f1. 2. missing-permissions: Added 'permissions: contents: read' top-level block to check-new-windows-versions.yml. 3. script-injection: (a) In test.yml, moved ${{ matrix.expected_rubygems_version }} into an env var EXPECTED_RUBYGEMS_VERSION and updated the Ruby code to read ENV["EXPECTED_RUBYGEMS_VERSION"]. (b) In check-new-windows-versions.yml, moved ${{ steps.pr.outputs.pull-request-url }} into an env var PR_URL and referenced it as $PR_URL in the shell command.

