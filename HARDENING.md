<!-- markdownlint-disable -->

# Hardening Report: gacts--run-and-post-run/v1.4.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--run-and-post-run/v1.4.5** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in release.yml are pinned to mutable tags instead of full 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if the tag is moved. Failing references: `actions/checkout@v7` (line 15), `gacts/github-slug@v1` (line 16).

Locations:

- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:16`

### unpinned-uses (severity: high)

Multiple `uses:` references in tests.yml are pinned to mutable tags instead of full 40-character commit SHAs. Failing references: `actions/checkout@v7` (lines 24, 29, 43, 57, 67), `gacts/gitleaks@v1` (line 25), `actions/setup-node@v7` (lines 30, 44), `actions/upload-artifact@v7` (line 47), `actions/download-artifact@v8` (line 60), `stefanzweifel/git-auto-commit-action@v7` (line 62).

Locations:

- `.github/workflows/tests.yml:24`
- `.github/workflows/tests.yml:25`
- `.github/workflows/tests.yml:29`
- `.github/workflows/tests.yml:30`
- `.github/workflows/tests.yml:43`
- `.github/workflows/tests.yml:44`
- `.github/workflows/tests.yml:47`
- `.github/workflows/tests.yml:57`
- `.github/workflows/tests.yml:60`
- `.github/workflows/tests.yml:62`
- `.github/workflows/tests.yml:67`

### script-injection (severity: high)

Sub-rule (a): Direct GitHub Actions expression interpolation inside `run:` shell commands in release.yml. Line 20: `git config --local user.name "${{ github.actor }}"` — the `github.actor` value is interpolated directly into a shell command before the shell parses it. Line 21: `git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"` — both `github.actor` and `secrets.GITHUB_TOKEN` are interpolated directly. An attacker who can control `github.actor` (e.g. via a crafted username) could inject shell metacharacters.

Locations:

- `.github/workflows/release.yml:20`
- `.github/workflows/release.yml:21`

### missing-permissions (severity: medium)

release.yml has no top-level `permissions:` key and the only job (`update-git-tag`) also has no `permissions:` key. This means the workflow runs with the default (broad) token permissions.

Locations:

- `.github/workflows/release.yml:1`

### missing-permissions (severity: medium)

tests.yml has no top-level `permissions:` key, and the jobs `gitleaks`, `eslint`, `dist-built`, and `run-this-action` each have no `permissions:` key. Only `commit-and-push-fresh-dist` has job-level permissions. The remaining jobs run with default (broad) token permissions.

Locations:

- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all 5 findings across release.yml and tests.yml:

**release.yml:**
- Pinned `actions/checkout@v7` → SHA `3d3c42e5aac5ba805825da76410c181273ba90b1`
- Pinned `gacts/github-slug@v1` → SHA `c29c0ddd888a1703a6fc06c55a5f6ddd7beae490`
- Fixed script-injection: moved `${{ github.actor }}` and `${{ secrets.GITHUB_TOKEN }}` into the step's `env:` block as `ACTOR` and `GITHUB_TOKEN`; shell script now uses `$ACTOR` and `$GITHUB_TOKEN`
- Added top-level `permissions: contents: write` (minimum needed for pushing tags)

**tests.yml:**
- Pinned all 6 action references to full commit SHAs (actions/checkout, gacts/gitleaks, actions/setup-node x2, actions/upload-artifact, actions/download-artifact, stefanzweifel/git-auto-commit-action)
- Added top-level `permissions: {}` to deny all by default
- Added `permissions: contents: read` to `gitleaks`, `eslint`, `dist-built`, and `run-this-action` jobs
- `commit-and-push-fresh-dist` already had appropriate job-level permissions

