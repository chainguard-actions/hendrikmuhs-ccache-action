<!-- markdownlint-disable -->

# Hardening Report: hendrikmuhs--ccache-action/v1.2.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **hendrikmuhs--ccache-action/v1.2.22** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` steps in `.github/workflows/tests.yml` directly interpolate `${{ ... }}` expressions inside shell command strings, violating rule (a). This allows expression values to be parsed as shell syntax before the shell ever sees them.

Affected steps and representative offending lines:
- Job `test_ccache`, step "Test ccache 1/2" (line ~66): `[[ ${{ steps.ccache.outputs.test-cache-hit }} = true ]] || ...` and `if [ ${{ matrix.variant }} = sccache ]; then` and `${{ matrix.variant }} gcc test.c -c -o test.o`
- Job `test_ccache`, step "Re-compile test program in Bash" (line ~78): `run: ${{ matrix.variant }} gcc test.c -c -o test.o`
- Job `test_ccache`, step "Re-compile test program in Bash" (line ~80): `run: ${{ matrix.variant }} gcc test.c -c -o test.o`
- Job `test_ccache`, step "Re-compile test program in PowerShell" (line ~84): `${{ matrix.variant }} gcc test.c -c -o test.o`
- Job `test_ccache`, step "Test ccache 2/2" (line ~89): `${{ matrix.variant }} -sv || ${{ matrix.variant }} -s || true`
- Job `test_cache_hit`, step "Test output true" (line ~107): `[[ ${{ steps.output.outputs.test-cache-hit }} = true ]]`
- Job `test_cache_miss`, step "Test output false" (line ~130): `[[ ${{ steps.output.outputs.test-cache-hit }} = false ]]`
- Job `test_restore_keys`, step "Test restore-keys" (line ~155): `[[ ${{ steps.restore-keys.outputs.test-cache-hit }} = true ]]`
- Job `test_option_create_symlink`, step "Test symlink" (line ~207): `if [ ${{ matrix.create-symlink }} = true ]; then`

Fix: move the values into `env:` variables and reference them as quoted shell variables (e.g., `"$MATRIX_VARIANT"`) inside `run:` blocks.

Locations:

- `.github/workflows/tests.yml:66`
- `.github/workflows/tests.yml:67`
- `.github/workflows/tests.yml:76`
- `.github/workflows/tests.yml:78`
- `.github/workflows/tests.yml:80`
- `.github/workflows/tests.yml:84`
- `.github/workflows/tests.yml:89`
- `.github/workflows/tests.yml:107`
- `.github/workflows/tests.yml:130`
- `.github/workflows/tests.yml:155`
- `.github/workflows/tests.yml:207`

### unpinned-uses (severity: high)

Every job in `.github/workflows/tests.yml` uses `actions/checkout@v6`, which is a mutable tag reference rather than a pinned 40-character commit SHA. If the tag is moved (e.g., by a supply-chain compromise), the workflow will silently execute different code. All `uses:` references should be pinned to a full SHA, e.g. `actions/checkout@<40-char-sha> # v6`.

Locations:

- `.github/workflows/tests.yml:23`
- `.github/workflows/tests.yml:33`
- `.github/workflows/tests.yml:53`
- `.github/workflows/tests.yml:100`
- `.github/workflows/tests.yml:119`
- `.github/workflows/tests.yml:143`
- `.github/workflows/tests.yml:163`
- `.github/workflows/tests.yml:170`
- `.github/workflows/tests.yml:177`
- `.github/workflows/tests.yml:186`
- `.github/workflows/tests.yml:194`
- `.github/workflows/tests.yml:201`
- `.github/workflows/tests.yml:215`
- `.github/workflows/tests.yml:224`
- `.github/workflows/tests.yml:231`

### missing-permissions (severity: medium)

`.github/workflows/tests.yml` has no top-level `permissions:` key and none of its jobs define a `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be `write-all` for older repositories). A minimal `permissions: {}` or specific scopes (e.g., `contents: read`) should be declared at the top level or per job.

Locations:

- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/tests.yml: (1) Pinned all 15 `actions/checkout@v6` references to the full SHA `d23441a48e516b6c34aea4fa41551a30e30af803` with `# v6` comment. (2) Added top-level `permissions: contents: read` block. (3) Moved all `${{ }}` expressions from `run:` shell strings into `env:` blocks and referenced them as quoted shell variables (`"$TEST_CACHE_HIT"`, `"$MATRIX_VARIANT"`, `"$CREATE_SYMLINK"`). The PowerShell step was updated to use `$env:MATRIX_VARIANT` syntax appropriate for PowerShell.

