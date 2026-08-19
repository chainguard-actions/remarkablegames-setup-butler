<!-- markdownlint-disable -->

# Hardening Report: remarkablegames--setup-butler/v2.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remarkablegames--setup-butler/v2.0.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags instead of pinned full-length SHA commit hashes. This exposes the workflow to supply-chain attacks if the referenced tag is moved or the action is compromised.

build.yml:
  - uses: actions/checkout@v6
  - uses: actions/setup-node@v6
  - uses: stefanzweifel/git-auto-commit-action@v7

commitlint.yml:
  - uses: actions/checkout@v6
  - uses: actions/setup-node@v6

release-please.yml:
  - uses: googleapis/release-please-action@v4
  - uses: actions/checkout@v6

test.yml:
  - uses: actions/checkout@v6
  - uses: actions/setup-node@v6
  - uses: codecov/codecov-action@v5

Locations:

- `.github/workflows/build.yml:14`
- `.github/workflows/build.yml:17`
- `.github/workflows/build.yml:27`
- `.github/workflows/commitlint.yml:12`
- `.github/workflows/commitlint.yml:17`
- `.github/workflows/release-please.yml:22`
- `.github/workflows/release-please.yml:31`
- `.github/workflows/test.yml:12`
- `.github/workflows/test.yml:15`
- `.github/workflows/test.yml:33`

### script-injection (severity: high)

GitHub Actions expressions are interpolated directly inside run: shell command strings, violating rule (a). This allows expression values to be interpreted as shell code before the shell ever sees them.

1. release-please.yml — 'Tag major and minor versions' step: `${{ needs.release.outputs.major }}` and `${{ needs.release.outputs.minor }}` are interpolated directly into git tag and git push commands (e.g. `git tag -d v${{ needs.release.outputs.major }} || true`). Although these values come from a prior release-please step, they still flow through YAML template substitution before the shell, making them a script-injection risk.

2. release-please.yml — 'Tag latest release' step: `${{ needs.release.outputs.tag_name }}` is interpolated directly into `gh release edit ${{ needs.release.outputs.tag_name }} --latest`.

3. test.yml — 'Check version' step: `${{ matrix.version }}` is interpolated directly inside a bash run block: `if [[ $(cat BUTLER_VERSION) != *'${{ matrix.version }}'* ]]; then`. Matrix values are workflow-controllable and must not be interpolated directly into run: scripts.

Locations:

- `.github/workflows/release-please.yml:38`
- `.github/workflows/release-please.yml:39`
- `.github/workflows/release-please.yml:40`
- `.github/workflows/release-please.yml:41`
- `.github/workflows/release-please.yml:42`
- `.github/workflows/release-please.yml:43`
- `.github/workflows/release-please.yml:47`
- `.github/workflows/test.yml:55`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 10 unpinned action references across build.yml, commitlint.yml, release-please.yml, and test.yml by resolving each tag to its full commit SHA (preserving the tag in a comment). Fixed all script-injection issues in release-please.yml (MAJOR, MINOR, TAG_NAME env vars for the tag/push/release steps) and test.yml (BUTLER_VERSION env var for the 'Check version' step, with output file renamed to BUTLER_VERSION_FILE to avoid name collision).

