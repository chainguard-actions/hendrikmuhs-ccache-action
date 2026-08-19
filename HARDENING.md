<!-- markdownlint-disable -->

# Hardening Report: hendrikmuhs--ccache-action/v.12.17

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **hendrikmuhs--ccache-action/v.12.17** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses: actions/checkout@v4` references in .github/workflows/tests.yml use a mutable tag (@v4) instead of a pinned 40-character SHA commit hash. This exposes the workflow to supply-chain attacks if the tag is moved. Affected lines: 23, 31, 47, 84, 100, 118, 135, 152, 157, 163, 168, 186.

Locations:

- `.github/workflows/tests.yml:23`
- `.github/workflows/tests.yml:31`
- `.github/workflows/tests.yml:47`
- `.github/workflows/tests.yml:84`
- `.github/workflows/tests.yml:100`
- `.github/workflows/tests.yml:118`
- `.github/workflows/tests.yml:135`
- `.github/workflows/tests.yml:152`
- `.github/workflows/tests.yml:157`
- `.github/workflows/tests.yml:163`
- `.github/workflows/tests.yml:168`
- `.github/workflows/tests.yml:186`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ }}` expressions into shell commands (rule a), allowing template substitution before the shell parses the string. Affected steps and expressions:
- 'Test ccache 1/2' (line 58): `[[ ${{ steps.ccache.outputs.test-cache-hit }} = true ]]` and `if [ ${{ matrix.variant }} = sccache ]` and `${{ matrix.variant }} gcc test.c -c -o test.o`
- 'Re-compile test program in Bash' (line 70): `run: ${{ matrix.variant }} gcc test.c -c -o test.o`
- 'Re-compile test program in Bash' (line 72): `run: ${{ matrix.variant }} gcc test.c -c -o test.o`
- 'Re-compile test program in PowerShell' (line 76): `${{ matrix.variant }} gcc test.c -c -o test.o`
- 'Test ccache 2/2' (line 80): `${{ matrix.variant }} -sv || ${{ matrix.variant }} -s || true`
- 'Test output true' (line 87): `[[ ${{ steps.output.outputs.test-cache-hit }} = true ]]`
- 'Test output false' (line 113): `[[ ${{ steps.output.outputs.test-cache-hit }} = false ]]`
- 'Test restore-keys' (line 130): `[[ ${{ steps.restore-keys.outputs.test-cache-hit }} = true ]]`
- 'Test symlink' (line 175): `if [ ${{ matrix.create-symlink }} = true ]`

Locations:

- `.github/workflows/tests.yml:58`
- `.github/workflows/tests.yml:70`
- `.github/workflows/tests.yml:72`
- `.github/workflows/tests.yml:76`
- `.github/workflows/tests.yml:80`
- `.github/workflows/tests.yml:87`
- `.github/workflows/tests.yml:113`
- `.github/workflows/tests.yml:130`
- `.github/workflows/tests.yml:175`

### missing-permissions (severity: medium)

The workflow file .github/workflows/tests.yml has no top-level `permissions:` key and none of its 12 jobs (build, unit_tests, test_ccache, test_cache_hit, test_cache_miss, test_restore_keys, test_docker_ubuntu, test_docker_alpine, test_option_save, test_option_append_timestamp, test_option_create_symlink, test_option_evict) define a `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings in .github/workflows/tests.yml:
1. unpinned-uses: Pinned all 12 `actions/checkout@v4` references to the full SHA `11d5960a326750d5838078e36cf38b85af677262` with `# v4` comment for readability.
2. script-injection: Moved all ${{ }} expressions from run: blocks into env: blocks. Variables introduced: TEST_CACHE_HIT (for steps.*.outputs.test-cache-hit), MATRIX_VARIANT (for matrix.variant), CREATE_SYMLINK (for matrix.create-symlink). The PowerShell step uses `$env:MATRIX_VARIANT` syntax. The `matrix.variant` value used as a command name is now safely referenced as `"$MATRIX_VARIANT"` in bash and `& "$env:MATRIX_VARIANT"` in PowerShell.
3. missing-permissions: Added `permissions: {}` at the top level of the workflow to enforce least-privilege (the workflow only runs tests and doesn't need any GitHub token permissions).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all 4 reported locations (5 total substitutions) in hardened/action/.github/workflows/tests.yml where $TEST_CACHE_HIT was used unquoted inside [[ ]] bash conditional tests. Changed `[[ $TEST_CACHE_HIT = true ]]` and `[[ $TEST_CACHE_HIT = false ]]` to `[[ "$TEST_CACHE_HIT" = true ]]` and `[[ "$TEST_CACHE_HIT" = false ]]` respectively in: (1) 'Test ccache 1/2' step in test_ccache job (2 occurrences on one line), (2) 'Test output true' step in test_cache_hit job, (3) 'Test output false' step in test_cache_miss job, (4) 'Test restore-keys' step in test_restore_keys job.

