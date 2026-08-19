<!-- markdownlint-disable -->

# Hardening Report: gacts--run-and-post-run/v1.4.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **gacts--run-and-post-run/v1.4.4** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): ${{ github.actor }} is directly interpolated inside a run: shell block in release.yml. GitHub Actions substitutes the expression value before the shell parses the command, so a username containing shell metacharacters (e.g. "; malicious_cmd; echo ") could break out of the quoted context and execute arbitrary commands. Offending lines:
  git config --local user.name "${{ github.actor }}"
  git remote set-url origin "https://${{ github.actor }}:${{ secrets.GITHUB_TOKEN }}@github.com/$REPO_PATH.git"
Fix: move github.actor into an env: variable and reference it as "$ACTOR" in the shell.

Locations:

- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:22`

### unpinned-uses (severity: high)

All uses: references in release.yml are pinned to mutable version tags instead of immutable 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if the upstream tag is moved or hijacked:
  - actions/checkout@v6 (line 15)
  - gacts/github-slug@v1 (line 16)

Locations:

- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:16`

### unpinned-uses (severity: high)

All uses: references in tests.yml are pinned to mutable version tags instead of immutable 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks:
  - actions/checkout@v4 (line 23)
  - gacts/gitleaks@v1 (line 25)
  - actions/checkout@v6 (lines 31, 42, 58, 65)
  - actions/setup-node@v4 (lines 32, 43)
  - actions/upload-artifact@v7 (line 47)
  - actions/download-artifact@v8 (line 59)
  - stefanzweifel/git-auto-commit-action@v7 (line 60)

Locations:

- `.github/workflows/tests.yml:23`
- `.github/workflows/tests.yml:25`
- `.github/workflows/tests.yml:31`
- `.github/workflows/tests.yml:32`
- `.github/workflows/tests.yml:42`
- `.github/workflows/tests.yml:43`
- `.github/workflows/tests.yml:47`
- `.github/workflows/tests.yml:58`
- `.github/workflows/tests.yml:59`
- `.github/workflows/tests.yml:60`
- `.github/workflows/tests.yml:65`

### missing-permissions (severity: medium)

release.yml has no top-level permissions: key and no job-level permissions: key on any job. Without explicit permissions, the workflow runs with the default (potentially broad) GITHUB_TOKEN permissions. Add a top-level permissions: block with the minimum required scopes (e.g. contents: write for tag pushing).

Locations:

- `.github/workflows/release.yml:1`

### missing-permissions (severity: medium)

tests.yml has no top-level permissions: key, and only the commit-and-push-fresh-dist job defines job-level permissions. The remaining jobs (gitleaks, eslint, dist-built, run-this-action) have no permissions: block, so they inherit the default GITHUB_TOKEN permissions. Add a restrictive top-level permissions: block (e.g. permissions: read-all or contents: read) and override per-job where write access is needed.

Locations:

- `.github/workflows/tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 5 findings across release.yml and tests.yml:

1. script-injection (release.yml): Moved `${{ github.actor }}` and `${{ secrets.GITHUB_TOKEN }}` into the step's `env:` block as `ACTOR` and `GITHUB_TOKEN`, referenced as plain shell variables in the `run:` block.

2. unpinned-uses (release.yml): Pinned actions/checkout@v6 → SHA df4cb1c and gacts/github-slug@v1 → SHA 83cd3d9.

3. unpinned-uses (tests.yml): Pinned all 8 action references to their immutable commit SHAs: actions/checkout@v4 (34e1148), gacts/gitleaks@v1 (c9a0338), actions/checkout@v6 (df4cb1c, all occurrences), actions/setup-node@v4 (49933ea), actions/upload-artifact@v7 (043fb46), actions/download-artifact@v8 (3e5f45b), stefanzweifel/git-auto-commit-action@v7 (4a55954).

4. missing-permissions (release.yml): Added top-level `permissions: contents: write` (minimum needed for pushing tags).

5. missing-permissions (tests.yml): Added top-level `permissions: contents: read` as a restrictive default; the commit-and-push-fresh-dist job retains its existing job-level `contents: write` override.

