<!-- markdownlint-disable -->

# Hardening Report: ruby--setup-ruby/v1.321.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ruby--setup-ruby/v1.321.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across all workflow files use mutable tag-based refs instead of pinned 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved or compromised. Affected references include: `actions/checkout@v7` (used in all 4 files), `peter-evans/create-pull-request@v8` (check-new-windows-versions.yml), and `./` (local action, test.yml). None are pinned to a full SHA digest.

Locations:

- `.github/workflows/test.yml:17`
- `.github/workflows/test.yml:27`
- `.github/workflows/check-new-windows-versions.yml:11`
- `.github/workflows/check-new-windows-versions.yml:27`
- `.github/workflows/release.yml:11`
- `.github/workflows/update-v1-branch.yml:9`

### missing-permissions (severity: medium)

The workflow file `check-new-windows-versions.yml` has no top-level `permissions:` key and no job-level `permissions:` key on its only job (`check_windows_versions`). Without explicit permissions, the workflow inherits the default repository permissions, which may be overly broad (e.g., write access to contents). The other three workflow files (test.yml, release.yml, update-v1-branch.yml) all have explicit `permissions:` blocks.

Locations:

- `.github/workflows/check-new-windows-versions.yml:1`

### script-injection (severity: high)

Two `run:` blocks directly interpolate `${{ }}` expressions into shell command strings, violating rule (a). (1) In test.yml, the expression `${{ matrix.expected_rubygems_version }}` is interpolated directly into a Ruby one-liner passed to the shell: `run: ruby -e 'puts Gem::VERSION; exit(Gem.rubygems_version >= Gem::Version.new("${{ matrix.expected_rubygems_version }}"))'`. The matrix value flows through YAML template substitution before the shell sees it, allowing an attacker-controlled matrix value to inject shell metacharacters. (2) In check-new-windows-versions.yml, the expression `${{ steps.pr.outputs.pull-request-url }}` is interpolated directly into an echo command: `run: echo "${{ steps.pr.outputs.pull-request-url }}"`. Step outputs are workflow-controllable and must not be interpolated directly into run: scripts.

Locations:

- `.github/workflows/test.yml:163`
- `.github/workflows/check-new-windows-versions.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

1. Pinned all actions/checkout@v7 references (23 occurrences across 4 workflow files) to SHA 3d3c42e5aac5ba805825da76410c181273ba90b1 # v7. Pinned peter-evans/create-pull-request@v8 to SHA 5f6978faf089d4d20b00c7766989d076bb2fc7f1 # v8 in check-new-windows-versions.yml. 2. Added top-level `permissions: contents: read` and job-level `permissions: contents: write, pull-requests: write` to check-new-windows-versions.yml. 3. Fixed script injection in test.yml by moving matrix.expected_rubygems_version to an env block (EXPECTED_RUBYGEMS_VERSION) and referencing it via ENV["EXPECTED_RUBYGEMS_VERSION"] in the Ruby one-liner. Fixed script injection in check-new-windows-versions.yml by moving steps.pr.outputs.pull-request-url to an env block (PR_URL) and referencing it as $PR_URL in the shell command.

