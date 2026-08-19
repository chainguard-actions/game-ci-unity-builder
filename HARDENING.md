<!-- markdownlint-disable -->

# Hardening Report: game-ci--unity-builder/v4.6.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **game-ci--unity-builder/v4.6.2** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### hardcoded-credentials (severity: high)

Two workflow files contain hardcoded literal credential values. In integrity-check.yml, the env var CODECOV_TOKEN is set to a literal UUID token value '2f2eb890-30e2-4724-83eb-7633832cf0de'. In build-tests-ubuntu.yml, the env var UNITY_LICENSE is set to a full hardcoded Unity XML license string (not a ${{ secrets.* }} reference). Both should use GitHub Actions secret expressions instead.

Locations:

- `.github/workflows/integrity-check.yml:8`
- `.github/workflows/build-tests-ubuntu.yml:13`

### script-injection (severity: high)

In cloud-runner-ci-pipeline.yml, the expression ${{ matrix.test }} is directly interpolated inside run: shell commands in three separate jobs (tests, k8sTests, awsTests). The matrix context is workflow-controllable and direct interpolation in a run: block allows script injection. The offending lines are: `run: yarn run test "${{ matrix.test }}" --detectOpenHandles --forceExit --runInBand`. This violates sub-rule (a): any ${{ ... }} expression directly inside a run: shell command string is a script-injection finding. The value should be passed via an env: variable and the env var should be double-quoted in the shell command.

Locations:

- `.github/workflows/cloud-runner-ci-pipeline.yml:57`
- `.github/workflows/cloud-runner-ci-pipeline.yml:88`
- `.github/workflows/cloud-runner-ci-pipeline.yml:116`

### unsafe-shell (severity: high)

In integrity-check.yml, the run: block executes `bash <(curl -s https://codecov.io/bash)`. This uses process substitution to download and immediately execute a remote script from codecov.io, which is functionally equivalent to `curl ... | bash`. The script is never saved to disk for inspection before execution, making this an unsafe-shell pattern.

Locations:

- `.github/workflows/integrity-check.yml:24`

### unpinned-uses (severity: high)

All 10 workflow files use action references pinned to mutable tags or version strings instead of immutable 40-character SHA commit hashes. Unpinned references are vulnerable to supply-chain attacks if the referenced tag is moved or the repository is compromised. Failing references include: activation.yml: game-ci/unity-request-activation-file@v2.0-alpha-1, actions/upload-artifact@v4; build-tests-mac.yml: actions/checkout@v4, actions/cache@v4, actions/upload-artifact@v4; build-tests-ubuntu.yml: jlumbroso/free-disk-space@v1.3.1, actions/checkout@v4, actions/cache@v4, actions/upload-artifact@v4; build-tests-windows.yml: actions/checkout@v4, actions/cache@v4, actions/upload-artifact@v4; cats.yml: ruairidhwm/action-cats@1.0.2; cleanup.yml: kolpav/purge-artifacts-action@v1, actions/checkout@v4, actions/setup-node@v4; cloud-runner-ci-pipeline.yml: actions/checkout@v4, aws-actions/configure-aws-credentials@v1, actions/checkout@v2, debianmaster/actions-k3s@v1.0.5, actions/upload-artifact@v4, actions/setup-node@v4; integrity-check.yml: actions/checkout@v4, actions/setup-node@v4; versioning.yml: Actions-R-Us/actions-tagger@v2.

Locations:

- `.github/workflows/activation.yml:11`
- `.github/workflows/build-tests-mac.yml:34`
- `.github/workflows/build-tests-ubuntu.yml:75`
- `.github/workflows/build-tests-windows.yml:38`
- `.github/workflows/cats.yml:13`
- `.github/workflows/cleanup.yml:8`
- `.github/workflows/cloud-runner-ci-pipeline.yml:55`
- `.github/workflows/integrity-check.yml:19`
- `.github/workflows/versioning.yml:9`

### missing-permissions (severity: medium)

Eight of the ten workflow files have no top-level permissions: block and no job-level permissions: blocks on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions (which may be read/write for contents), violating the principle of least privilege. The affected files are: activation.yml, build-tests-mac.yml, build-tests-ubuntu.yml, build-tests-windows.yml, cats.yml, cleanup.yml, integrity-check.yml, and versioning.yml. (cloud-runner-async-checks.yml and cloud-runner-ci-pipeline.yml do have permissions blocks.)

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

**Fixes applied:** hardcoded-credentials, script-injection, unsafe-shell, unpinned-uses, missing-permissions

**Notes:**

Fixed all 5 security findings across 10 workflow files:

1. hardcoded-credentials: Removed hardcoded CODECOV_TOKEN UUID from integrity-check.yml (replaced with ${{ secrets.CODECOV_TOKEN }}) and removed hardcoded UNITY_LICENSE XML string from build-tests-ubuntu.yml (replaced with ${{ secrets.UNITY_LICENSE }}).

2. script-injection: Fixed all 3 occurrences in cloud-runner-ci-pipeline.yml where ${{ matrix.test }} was directly interpolated in run: shell commands. Moved to env: block as MATRIX_TEST and referenced as "$MATRIX_TEST" in the shell.

3. unsafe-shell: Fixed integrity-check.yml's 'bash <(curl -s https://codecov.io/bash)' by downloading the script to a temp file first with 'curl -s ... -o "$INSTALL_SCRIPT"' then executing 'bash "$INSTALL_SCRIPT"'.

4. unpinned-uses: Pinned all 12 action references across all workflow files to full 40-character SHA hashes: actions/checkout@v4→11d5960a, actions/checkout@v2→0717577d, actions/cache@v4→0057852b, actions/upload-artifact@v4→ea165f8d, actions/setup-node@v4→49933ea5, game-ci/unity-request-activation-file@v2.0-alpha-1→de1240e1, jlumbroso/free-disk-space@v1.3.1→54081f13, ruairidhwm/action-cats@1.0.2→309530f7, kolpav/purge-artifacts-action@v1→04c636a5, aws-actions/configure-aws-credentials@v1→67fbcbb1, debianmaster/actions-k3s@v1.0.5→b9cf3f59, Actions-R-Us/actions-tagger@v2→330ddfac.

5. missing-permissions: Added permissions blocks to all 8 affected files. Used 'contents: read' as the minimal permission for most workflows; 'contents: write' for versioning.yml (needs to update tags); 'contents: read, actions: write' for cleanup.yml (needs to delete artifacts).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/cloud-runner-ci-pipeline.yml in the buildTargetTests job. The step that ran `cp ./cloud-runner-cache/cache/${{ steps.unity-build.outputs.CACHE_KEY }}/build/${{ steps.unity-build.outputs.BUILD_ARTIFACT }} ${{ steps.unity-build.outputs.BUILD_ARTIFACT }}` was refactored to move the step output expressions into an `env:` block (as CACHE_KEY and BUILD_ARTIFACT), and the shell command now uses properly double-quoted shell variable references (`"$CACHE_KEY"` and `"$BUILD_ARTIFACT"`) to prevent script injection.

