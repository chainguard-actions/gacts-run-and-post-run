<!-- markdownlint-disable -->

# Hardening Report: gacts--run-and-post-run/v1.4.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--run-and-post-run/v1.4.3** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in release.yml use mutable version tags instead of full 40-character SHA digests, making the workflow vulnerable to supply-chain attacks if the referenced action tags are moved or compromised. Failing references: `actions/checkout@v5` (line 15), `gacts/github-slug@v1` (line 16).

Locations:

- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:16`

### unpinned-uses (severity: high)

All `uses:` references in tests.yml use mutable version tags instead of full 40-character SHA digests. Failing references: `actions/checkout@v4` (line 24), `gacts/gitleaks@v1` (line 25), `actions/checkout@v5` (lines 30, 40, 55, 62), `actions/setup-node@v4` (lines 31, 41), `actions/upload-artifact@v5` (line 44), `actions/download-artifact@v6` (line 56), `stefanzweifel/git-auto-commit-action@v7` (line 57).

Locations:

- `.github/workflows/tests.yml:24`
- `.github/workflows/tests.yml:25`
- `.github/workflows/tests.yml:30`
- `.github/workflows/tests.yml:31`
- `.github/workflows/tests.yml:40`
- `.github/workflows/tests.yml:41`
- `.github/workflows/tests.yml:44`
- `.github/workflows/tests.yml:55`
- `.github/workflows/tests.yml:56`
- `.github/workflows/tests.yml:57`
- `.github/workflows/tests.yml:62`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside `run:` shell command strings. In the unnamed step (line 18), `${{ github.actor }}` is embedded directly in two shell commands: `git config --local user.name "${{ github.actor }}"` (line 20) and `git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"` (line 21). An attacker who controls the actor name could inject shell metacharacters.

Locations:

- `.github/workflows/release.yml:20`
- `.github/workflows/release.yml:21`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside `run:` shell command strings in the 'Inline variables' step. `${{ env.SHARED_VAR }}` is embedded directly in shell commands: `echo "${{ env.SHARED_VAR }}"` (line 93) and `if [ "${{ env.SHARED_VAR }}" = "value" ]` (line 94). Additionally, in the 'Post run condition' step, `${{ job.status }}` is embedded directly in a shell command: `if [ "${{ job.status }}" = "success" ]` (line 113). These expressions are substituted by the template engine before the shell sees them, allowing injection of shell metacharacters.

Locations:

- `.github/workflows/tests.yml:93`
- `.github/workflows/tests.yml:94`
- `.github/workflows/tests.yml:113`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and the single job `update-git-tag` also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the default repository permissions (which may be broad), violating the principle of least privilege.

Locations:

- `.github/workflows/release.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key. Jobs `gitleaks` (line 20), `eslint` (line 27), `dist-built` (line 35), and `run-this-action` (line 61) have no job-level `permissions:` key. Only the `commit-and-push-fresh-dist` job (line 51) defines explicit permissions. All other jobs inherit potentially broad default permissions.

Locations:

- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all findings in both workflow files:

release.yml:
- Pinned actions/checkout@v5 → SHA fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09
- Pinned gacts/github-slug@v1 → SHA 83cd3d95888be8777f0d9c810c1266da1c5c3ef1
- Fixed script injection: moved ${{ github.actor }} to GIT_ACTOR env var and ${{ secrets.GITHUB_TOKEN }} to GITHUB_TOKEN env var; shell now references $GIT_ACTOR and $GITHUB_TOKEN
- Added top-level permissions: {} and job-level permissions: contents: write (required for git tag push)

tests.yml:
- Pinned all 8 action references to full SHA digests (actions/checkout@v4, gacts/gitleaks@v1, actions/checkout@v5 ×5, actions/setup-node@v4 ×2, actions/upload-artifact@v5, actions/download-artifact@v6, stefanzweifel/git-auto-commit-action@v7)
- Fixed script injection in 'Inline variables' step: moved ${{ env.SHARED_VAR }} to SHARED_VAR env var; shell now references $SHARED_VAR
- Fixed script injection in 'Post run condition' step: moved ${{ job.status }} to JOB_STATUS env var; shell now references $JOB_STATUS
- Added top-level permissions: {} and job-level permissions: {} to gitleaks, eslint, dist-built, and run-this-action jobs

