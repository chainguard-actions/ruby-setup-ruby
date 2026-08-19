<!-- markdownlint-disable -->

# Hardening Report: ruby--setup-ruby/v1.312.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ruby--setup-ruby/v1.312.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files use `uses:` references pinned to mutable version tags rather than immutable 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks if the referenced action tag is moved or compromised.

Failing references:
- `.github/workflows/check-new-windows-versions.yml`: `actions/checkout@v6`, `peter-evans/create-pull-request@v8`
- `.github/workflows/release.yml`: `actions/checkout@v6`
- `.github/workflows/test.yml`: `actions/checkout@v6` (multiple occurrences)
- `.github/workflows/update-v1-branch.yml`: `actions/checkout@v6`

Locations:

- `.github/workflows/check-new-windows-versions.yml:11`
- `.github/workflows/check-new-windows-versions.yml:27`
- `.github/workflows/release.yml:9`
- `.github/workflows/test.yml:20`
- `.github/workflows/update-v1-branch.yml:9`

### missing-permissions (severity: medium)

`.github/workflows/check-new-windows-versions.yml` has no top-level `permissions:` key and no job-level `permissions:` key on its only job (`check_windows_versions`). Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/check-new-windows-versions.yml:1`

### script-injection (severity: high)

Two `run:` blocks directly interpolate GitHub Actions expressions (`${{ ... }}`) into shell command strings, enabling script injection if the expression value contains shell metacharacters.

(a) `.github/workflows/test.yml`: The step `run: ruby -e 'puts Gem::VERSION; exit(Gem.rubygems_version >= Gem::Version.new("${{ matrix.expected_rubygems_version }}"))'` interpolates `${{ matrix.expected_rubygems_version }}` directly into the shell command. Matrix values are workflow-controllable and flow through YAML template substitution before the shell sees them.

(a) `.github/workflows/check-new-windows-versions.yml`: The step `run: echo "${{ steps.pr.outputs.pull-request-url }}"` interpolates `${{ steps.pr.outputs.pull-request-url }}` directly into the shell command. Step outputs are workflow-controllable and must not be interpolated directly into `run:` scripts.

Locations:

- `.github/workflows/test.yml:163`
- `.github/workflows/check-new-windows-versions.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings:
1. unpinned-uses: Pinned all actions/checkout@v6 (23 occurrences) to SHA d23441a48e516b6c34aea4fa41551a30e30af803 and peter-evans/create-pull-request@v8 to SHA 5f6978faf089d4d20b00c7766989d076bb2fc7f1 across check-new-windows-versions.yml, release.yml, test.yml, and update-v1-branch.yml.
2. missing-permissions: Added `permissions: contents: write, pull-requests: write` to check-new-windows-versions.yml.
3. script-injection: (a) In test.yml, moved matrix.expected_rubygems_version into an env var EXPECTED_RUBYGEMS_VERSION and referenced it via ENV["EXPECTED_RUBYGEMS_VERSION"] in the Ruby script. (b) In check-new-windows-versions.yml, moved steps.pr.outputs.pull-request-url into an env var PR_URL and referenced it as $PR_URL in the shell command.

