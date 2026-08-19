<!-- markdownlint-disable -->

# Hardening Report: gacts--run-and-post-run/v1.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--run-and-post-run/v1.4.1** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files are pinned to mutable tags rather than immutable 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced tag is moved or compromised.

.github/workflows/tests.yml:
- `actions/checkout@v4` (lines 23, 30, 39, 53, 59)
- `gacts/gitleaks@v1` (line 24)
- `gacts/setup-node-with-cache@v1` (lines 31, 40)
- `actions/upload-artifact@v4` (line 44)
- `actions/download-artifact@v4` (line 54)
- `stefanzweifel/git-auto-commit-action@v5` (line 55)

.github/workflows/release.yml:
- `actions/checkout@v4` (line 15)
- `gacts/github-slug@v1` (line 16)

Locations:

- `.github/workflows/tests.yml:23`
- `.github/workflows/tests.yml:24`
- `.github/workflows/tests.yml:31`
- `.github/workflows/tests.yml:40`
- `.github/workflows/tests.yml:44`
- `.github/workflows/tests.yml:54`
- `.github/workflows/tests.yml:55`
- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:16`

### script-injection (severity: high)

Sub-rule (a): `${{ github.actor }}` is directly interpolated inside a `run:` shell command string in release.yml. This allows an attacker who can control their GitHub username (e.g., with shell metacharacters) to inject arbitrary shell commands. The offending lines are:
  `git config --local user.name "${{ github.actor }}"`
  `git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"`
These expressions should be moved to an `env:` block and the env vars should be used in the shell script instead.

Locations:

- `.github/workflows/release.yml:20`
- `.github/workflows/release.yml:21`

### missing-permissions (severity: medium)

Neither workflow file has a top-level `permissions:` key, and several jobs also lack job-level `permissions:` keys. This means those jobs run with the default (potentially broad) token permissions.

.github/workflows/tests.yml: No top-level permissions; jobs `gitleaks`, `eslint`, `dist-built`, and `run-this-action` all lack job-level permissions blocks (only `commit-and-push-fresh-dist` has job-level permissions).

.github/workflows/release.yml: No top-level permissions; the `update-git-tag` job has no job-level permissions block.

Locations:

- `.github/workflows/tests.yml:1`
- `.github/workflows/release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across both workflow files:

1. **unpinned-uses**: Pinned all 7 unique action references to full 40-char SHAs with original tags as comments. Affected: actions/checkout, gacts/gitleaks, gacts/setup-node-with-cache, actions/upload-artifact, actions/download-artifact, stefanzweifel/git-auto-commit-action, gacts/github-slug.

2. **script-injection** (release.yml): Moved `${{ github.actor }}` out of the `run:` shell string into the step's `env:` block as `GIT_ACTOR`. The shell script now uses `$GIT_ACTOR` for both `git config --local user.name` and the remote URL construction.

3. **missing-permissions**: Added top-level `permissions: {}` to both workflow files. Added job-level permissions to all jobs that lacked them: `gitleaks`, `eslint`, `dist-built`, and `run-this-action` (all get `contents: read`); `update-git-tag` gets `contents: write` (required for pushing tags). The `commit-and-push-fresh-dist` job already had `contents: write` and `pull-requests: write`.

### Iteration 2

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

1. Fixed script-injection in .github/workflows/release.yml (line 27): Added `GH_TOKEN: "${{ secrets.GITHUB_TOKEN }}"` to the step's env: block and replaced the inline `${{ secrets.GITHUB_TOKEN }}` expression in the run: command with `$GH_TOKEN`. 2. Fixed unpinned-uses in .github/workflows/dependabot.yml (line 19): Pinned `dependabot/fetch-metadata@v2` to its full commit SHA `dependabot/fetch-metadata@21025c705c08248db411dc16f3619e6b5f9ea21a # v2`.

