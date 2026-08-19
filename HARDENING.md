<!-- markdownlint-disable -->

# Hardening Report: hendrikmuhs--ccache-action/v1.2.23

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **hendrikmuhs--ccache-action/v1.2.23** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses: actions/checkout@v6` references in .github/workflows/tests.yml use a mutable tag (@v6) instead of a pinned 40-character commit SHA. This exposes the workflow to supply-chain attacks if the tag is moved. Every job in the workflow uses this unpinned reference.

Locations:

- `.github/workflows/tests.yml:23`
- `.github/workflows/tests.yml:32`
- `.github/workflows/tests.yml:52`
- `.github/workflows/tests.yml:100`
- `.github/workflows/tests.yml:122`
- `.github/workflows/tests.yml:147`
- `.github/workflows/tests.yml:163`
- `.github/workflows/tests.yml:169`
- `.github/workflows/tests.yml:175`
- `.github/workflows/tests.yml:183`
- `.github/workflows/tests.yml:192`
- `.github/workflows/tests.yml:201`
- `.github/workflows/tests.yml:218`
- `.github/workflows/tests.yml:226`
- `.github/workflows/tests.yml:232`

### missing-permissions (severity: medium)

The workflow file .github/workflows/tests.yml has no top-level `permissions:` key and no job-level `permissions:` key in any of its jobs. Without explicit permissions, the workflow inherits the default (potentially write) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/tests.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks in .github/workflows/tests.yml directly interpolate GitHub Actions expressions (`${{ ... }}`) into shell commands, enabling script injection. Specifically:

(a) `${{ steps.ccache.outputs.test-cache-hit }}` is interpolated directly into `[[ ... ]]` comparisons in run: blocks (e.g., `[[ ${{ steps.ccache.outputs.test-cache-hit }} = true ]]`). Step outputs can contain shell metacharacters.

(a) `${{ matrix.variant }}` is used directly as a shell command prefix in multiple run: blocks (e.g., `${{ matrix.variant }} gcc test.c -c -o test.o`, `${{ matrix.variant }} -sv || ...`). Matrix values flow through YAML template substitution before the shell sees them.

(a) `${{ matrix.create-symlink }}` is interpolated directly into an `if [ ... ]` condition in a run: block (e.g., `if [ ${{ matrix.create-symlink }} = true ]; then`).

All of these should be routed through env: variables and then double-quoted in the shell script.

Locations:

- `.github/workflows/tests.yml:63`
- `.github/workflows/tests.yml:64`
- `.github/workflows/tests.yml:73`
- `.github/workflows/tests.yml:75`
- `.github/workflows/tests.yml:77`
- `.github/workflows/tests.yml:80`
- `.github/workflows/tests.yml:83`
- `.github/workflows/tests.yml:107`
- `.github/workflows/tests.yml:129`
- `.github/workflows/tests.yml:157`
- `.github/workflows/tests.yml:209`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed hardened/action/.github/workflows/tests.yml: (1) Pinned all 15 `actions/checkout@v6` references to SHA `df4cb1c069e1874edd31b4311f1884172cec0e10` with `# v6` comment. (2) Added `permissions: {}` top-level block to enforce least privilege. (3) Moved all `${{ steps.ccache.outputs.test-cache-hit }}`, `${{ steps.output.outputs.test-cache-hit }}`, `${{ steps.restore-keys.outputs.test-cache-hit }}`, `${{ matrix.variant }}`, and `${{ matrix.create-symlink }}` expressions from `run:` shell strings into `env:` blocks (as CACHE_HIT, VARIANT, CREATE_SYMLINK), then referenced them as double-quoted shell variables to prevent script injection. The PowerShell step was updated to use `& "$env:VARIANT"` syntax.

