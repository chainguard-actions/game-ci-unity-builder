<!-- markdownlint-disable -->

# Hardening Report: game-ci--unity-builder/v4.6.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **game-ci--unity-builder/v4.6.3** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### hardcoded-credentials (severity: high)

A literal CODECOV_TOKEN UUID is hardcoded in the workflow env block: `CODECOV_TOKEN: '2f2eb890-30e2-4724-83eb-7633832cf0de'`. This exposes the token in the repository and should be stored as a GitHub Actions secret instead.

Locations:

- `.github/workflows/integrity-check.yml:8`

### hardcoded-credentials (severity: high)

A full Unity license XML is hardcoded as a literal value for `UNITY_LICENSE` in the top-level `env:` block. This exposes a real license file (including serial hash `2033b8ac3e6faa3742ca9f0bfae44d18f2a96b80` and developer data) in the repository. It should be stored as a GitHub Actions secret.

Locations:

- `.github/workflows/build-tests-ubuntu.yml:13`

### unsafe-shell (severity: high)

The run step executes `bash <(curl -s https://codecov.io/bash)`, which downloads a remote script and pipes it directly into bash via process substitution. This is equivalent to `curl ... | bash` and allows arbitrary code execution if the remote URL is compromised.

Locations:

- `.github/workflows/integrity-check.yml:25`

### script-injection (severity: high)

Sub-rule (a): The expression `${{ matrix.test }}` is interpolated directly inside a `run:` shell command string: `run: yarn run test "${{ matrix.test }}" --detectOpenHandles --forceExit --runInBand`. Although `matrix.test` values are defined in the workflow YAML, any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the template engine before the shell sees it. This pattern appears in three jobs: `tests`, `k8sTests`, and `awsTests`.

Locations:

- `.github/workflows/cloud-runner-ci-pipeline.yml:62`
- `.github/workflows/cloud-runner-ci-pipeline.yml:97`
- `.github/workflows/cloud-runner-ci-pipeline.yml:131`

### unpinned-uses (severity: high)

Multiple workflow files reference actions by mutable tags or version strings instead of full 40-character commit SHAs. Failing references include: `game-ci/unity-request-activation-file@v2.0-alpha-1`, `actions/upload-artifact@v4`, `actions/checkout@v4`, `actions/checkout@v2`, `actions/cache@v4`, `actions/setup-node@v4`, `jlumbroso/free-disk-space@v1.3.1`, `kolpav/purge-artifacts-action@v1`, `aws-actions/configure-aws-credentials@v1`, `debianmaster/actions-k3s@v1.0.5`, `ruairidhwm/action-cats@1.0.2`, `Actions-R-Us/actions-tagger@v2`.

Locations:

- `.github/workflows/activation.yml:11`
- `.github/workflows/activation.yml:13`
- `.github/workflows/build-tests-mac.yml:34`
- `.github/workflows/build-tests-mac.yml:40`
- `.github/workflows/build-tests-mac.yml:79`
- `.github/workflows/build-tests-ubuntu.yml:72`
- `.github/workflows/build-tests-ubuntu.yml:78`
- `.github/workflows/build-tests-ubuntu.yml:84`
- `.github/workflows/build-tests-ubuntu.yml:148`
- `.github/workflows/build-tests-windows.yml:40`
- `.github/workflows/build-tests-windows.yml:46`
- `.github/workflows/build-tests-windows.yml:113`
- `.github/workflows/cats.yml:13`
- `.github/workflows/cleanup.yml:9`
- `.github/workflows/cleanup.yml:14`
- `.github/workflows/cleanup.yml:17`
- `.github/workflows/cloud-runner-ci-pipeline.yml:49`
- `.github/workflows/cloud-runner-ci-pipeline.yml:52`
- `.github/workflows/cloud-runner-ci-pipeline.yml:86`
- `.github/workflows/cloud-runner-ci-pipeline.yml:89`
- `.github/workflows/cloud-runner-ci-pipeline.yml:120`
- `.github/workflows/cloud-runner-ci-pipeline.yml:123`
- `.github/workflows/cloud-runner-ci-pipeline.yml:152`
- `.github/workflows/cloud-runner-ci-pipeline.yml:175`
- `.github/workflows/cloud-runner-ci-pipeline.yml:186`
- `.github/workflows/integrity-check.yml:19`
- `.github/workflows/integrity-check.yml:20`
- `.github/workflows/versioning.yml:11`

### missing-permissions (severity: medium)

These workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any job. Without explicit permissions, workflows inherit the default repository token permissions (which may be broad). Each file should declare minimal required permissions.

Locations:

- `.github/workflows/activation.yml:1`
- `.github/workflows/build-tests-mac.yml:1`
- `.github/workflows/build-tests-ubuntu.yml:1`
- `.github/workflows/build-tests-windows.yml:1`
- `.github/workflows/cats.yml:1`
- `.github/workflows/cleanup.yml:1`
- `.github/workflows/integrity-check.yml:1`
- `.github/workflows/versioning.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** hardcoded-credentials, unsafe-shell, script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 6 finding types across 8 workflow files:

1. hardcoded-credentials: Replaced hardcoded CODECOV_TOKEN UUID in integrity-check.yml and full Unity license XML in build-tests-ubuntu.yml with GitHub Actions secret references (${{ secrets.CODECOV_TOKEN }} and ${{ secrets.UNITY_LICENSE }}).

2. unsafe-shell: Replaced `bash <(curl -s https://codecov.io/bash)` in integrity-check.yml with a safe pattern: download to a temp file with mktemp, then execute the file separately.

3. script-injection: In cloud-runner-ci-pipeline.yml, moved `${{ matrix.test }}` out of all three run: shell strings (tests, k8sTests, awsTests jobs) into env: blocks as MATRIX_TEST, referenced as "$MATRIX_TEST" in the shell commands.

4. unpinned-uses: Pinned all 12 action references to full 40-character commit SHAs: game-ci/unity-request-activation-file@v2.0-alpha-1, actions/upload-artifact@v4, actions/checkout@v4, actions/checkout@v2, actions/cache@v4, actions/setup-node@v4, jlumbroso/free-disk-space@v1.3.1, kolpav/purge-artifacts-action@v1, aws-actions/configure-aws-credentials@v1, debianmaster/actions-k3s@v1.0.5, ruairidhwm/action-cats@1.0.2, Actions-R-Us/actions-tagger@v2.

5. missing-permissions: Added permissions blocks to all 8 workflow files. Used contents: read for most; versioning.yml uses contents: write (needed to update tags); cleanup.yml uses contents: read + actions: write (needed to delete artifacts); cloud-runner-ci-pipeline.yml already had permissions.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/cloud-runner-ci-pipeline.yml at the buildTargetTests job. The cp command that directly interpolated ${{ steps.unity-build.outputs.CACHE_KEY }} and ${{ steps.unity-build.outputs.BUILD_ARTIFACT }} into the shell script was refactored to use an env: block. The step output values are now assigned to CACHE_KEY and BUILD_ARTIFACT environment variables, and the shell script references them as "$CACHE_KEY" and "$BUILD_ARTIFACT" with proper double-quoting.

