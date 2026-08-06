<!-- markdownlint-disable -->

# Hardening Report: remarkablegames--setup-butler/v3.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **remarkablegames--setup-butler/v3.0.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All four workflow files reference GitHub Actions using mutable tag-based refs (@v7, @v5) instead of immutable 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks if a tag is moved or a dependency is compromised. Failing references include: actions/checkout@v7, actions/setup-node@v7, stefanzweifel/git-auto-commit-action@v7, googleapis/release-please-action@v5, codecov/codecov-action@v7.

Locations:

- `.github/workflows/build.yml:13`
- `.github/workflows/build.yml:18`
- `.github/workflows/build.yml:27`
- `.github/workflows/commitlint.yml:12`
- `.github/workflows/commitlint.yml:18`
- `.github/workflows/release-please.yml:22`
- `.github/workflows/release-please.yml:31`
- `.github/workflows/test.yml:10`
- `.github/workflows/test.yml:16`
- `.github/workflows/test.yml:30`

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings, allowing injection of shell metacharacters before the shell ever sees the value.

1. In release-please.yml, the 'Tag major and minor versions' step interpolates `${{ needs.release.outputs.major }}`, `${{ needs.release.outputs.minor }}` directly into git tag and git push commands (e.g. `git tag -d v${{ needs.release.outputs.major }} || true`). If the release-please action's outputs were tampered with or the step output contained shell metacharacters, arbitrary commands could execute.

2. In release-please.yml, the 'Tag latest release' step interpolates `${{ needs.release.outputs.tag_name }}` directly into `gh release edit ${{ needs.release.outputs.tag_name }} --latest`.

3. In test.yml, the 'Check version' step interpolates `${{ matrix.version }}` directly inside a run: shell string: `if [[ $(cat BUTLER_VERSION) != *'${{ matrix.version }}'* ]]; then`. The matrix value is workflow-controlled and flows through YAML template substitution before the shell parses it.

Locations:

- `.github/workflows/release-please.yml:40`
- `.github/workflows/release-please.yml:50`
- `.github/workflows/test.yml:55`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all unpinned action references by resolving to full SHA hashes (actions/checkout@3d3c42e5..., actions/setup-node@82076278..., stefanzweifel/git-auto-commit-action@4a55954c..., googleapis/release-please-action@45996ed1..., codecov/codecov-action@fb8b3582...). Fixed all three script injection points in release-please.yml and test.yml by moving ${{ }} expressions into step env: blocks and referencing them as plain environment variables in the shell scripts.

