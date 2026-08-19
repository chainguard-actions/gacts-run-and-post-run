<!-- markdownlint-disable -->

# Hardening Report: gacts--run-and-post-run/v1.4.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--run-and-post-run/v1.4.2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings. In release.yml, `${{ github.actor }}` and `${{ secrets.GITHUB_TOKEN }}` appear literally in a run: block — an attacker who can influence `github.actor` (e.g. via a crafted username) could inject shell metacharacters. In tests.yml, `${{ env.SHARED_VAR }}` and `${{ job.status }}` are interpolated directly in run: blocks passed as action inputs.

Locations:

- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:20`
- `.github/workflows/tests.yml:80`
- `.github/workflows/tests.yml:81`
- `.github/workflows/tests.yml:98`

### unpinned-uses (severity: high)

All uses: references in both workflow files use mutable tag refs instead of full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the upstream tag is moved or the repository is compromised. Failing references in release.yml: `actions/checkout@v4`, `gacts/github-slug@v1`. Failing references in tests.yml: `actions/checkout@v4` (multiple), `gacts/gitleaks@v1`, `actions/setup-node@v4` (multiple), `actions/upload-artifact@v4`, `actions/download-artifact@v4`, `stefanzweifel/git-auto-commit-action@v5`.

Locations:

- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:16`
- `.github/workflows/tests.yml:22`
- `.github/workflows/tests.yml:23`
- `.github/workflows/tests.yml:28`
- `.github/workflows/tests.yml:29`
- `.github/workflows/tests.yml:35`
- `.github/workflows/tests.yml:36`
- `.github/workflows/tests.yml:39`
- `.github/workflows/tests.yml:48`
- `.github/workflows/tests.yml:49`
- `.github/workflows/tests.yml:51`

### missing-permissions (severity: medium)

release.yml has no top-level `permissions:` key and the single job `update-git-tag` also has no job-level `permissions:` key, so the workflow runs with the default (broad) token permissions. In tests.yml there is no top-level `permissions:` key, and the jobs `gitleaks`, `eslint`, `dist-built`, and `run-this-action` all lack job-level `permissions:` blocks (only `commit-and-push-fresh-dist` has explicit permissions).

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across release.yml and tests.yml:

1. script-injection: Moved ${{ github.actor }} and ${{ secrets.GITHUB_TOKEN }} from inline run: strings in release.yml into the step env: block (as GIT_ACTOR and GITHUB_TOKEN). In tests.yml, moved ${{ env.SHARED_VAR }} and ${{ job.status }} from inline run:/post: strings into step env: blocks (as SHARED_VAR and JOB_STATUS), referencing them as plain shell variables.

2. unpinned-uses: Pinned all 7 action references to full 40-character SHA hashes with original tag preserved as comment: actions/checkout@v4→11d5960a..., gacts/github-slug@v1→83cd3d95..., gacts/gitleaks@v1→c9a03383..., actions/setup-node@v4→49933ea5..., actions/upload-artifact@v4→ea165f8d..., actions/download-artifact@v4→d3f86a10..., stefanzweifel/git-auto-commit-action@v5→b863ae19...

3. missing-permissions: Added top-level permissions: {} to both files. Added job-level permissions: {} to gitleaks, eslint, dist-built, and run-this-action jobs in tests.yml. Added job-level permissions: contents: write to update-git-tag in release.yml (needed to push tags). The commit-and-push-fresh-dist job already had explicit permissions.

