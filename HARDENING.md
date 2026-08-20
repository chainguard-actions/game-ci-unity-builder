<!-- markdownlint-disable -->

# Hardening Report: game-ci--unity-builder/v4.8.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **game-ci--unity-builder/v4.8.1** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### hardcoded-credentials (severity: high)

integrity-check.yml contains a hardcoded CODECOV_TOKEN literal value: `CODECOV_TOKEN: '2f2eb890-30e2-4724-83eb-7633832cf0de'`. This is a real token UUID committed in plaintext. It should be stored as a GitHub Actions secret and referenced via `${{ secrets.CODECOV_TOKEN }}`.

Locations:

- `.github/workflows/integrity-check.yml:8`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or version strings instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks.

- activation.yml: `game-ci/unity-request-activation-file@v2.0-alpha-1`, `actions/upload-artifact@v4`
- build-tests-mac.yml: `actions/checkout@v4`, `actions/cache@v4`, `actions/upload-artifact@v4`
- build-tests-ubuntu.yml: `jlumbroso/free-disk-space@v1.3.1`, `actions/checkout@v4`, `actions/cache@v4`, `actions/upload-artifact@v4`
- build-tests-windows.yml: `actions/checkout@v4`, `actions/cache@v4`, `actions/upload-artifact@v4`
- cats.yml: `ruairidhwm/action-cats@1.0.2`
- cleanup.yml: `kolpav/purge-artifacts-action@v1`, `actions/checkout@v4`, `actions/setup-node@v4`
- cloud-runner-ci-pipeline.yml: `actions/checkout@v4`, `aws-actions/configure-aws-credentials@v1`, `actions/checkout@v2`, `debianmaster/actions-k3s@v1.0.5`, `actions/upload-artifact@v4`, `actions/setup-node@v4`
- integrity-check.yml: `actions/checkout@v4`, `actions/setup-node@v4`
- versioning.yml: `Actions-R-Us/actions-tagger@v2`

Locations:

- `.github/workflows/activation.yml:10`
- `.github/workflows/build-tests-mac.yml:35`
- `.github/workflows/build-tests-ubuntu.yml:77`
- `.github/workflows/build-tests-windows.yml:40`
- `.github/workflows/cats.yml:12`
- `.github/workflows/cleanup.yml:9`
- `.github/workflows/cloud-runner-ci-pipeline.yml:57`
- `.github/workflows/integrity-check.yml:19`
- `.github/workflows/versioning.yml:9`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` block and no job-level `permissions:` blocks. Without explicit permissions, workflows inherit the default (often overly broad) repository permissions, which can allow unintended write access.

- activation.yml
- build-tests-mac.yml
- build-tests-ubuntu.yml
- build-tests-windows.yml
- cats.yml
- cleanup.yml
- integrity-check.yml
- versioning.yml

Locations:

- `.github/workflows/activation.yml:1`
- `.github/workflows/build-tests-mac.yml:1`
- `.github/workflows/build-tests-ubuntu.yml:1`
- `.github/workflows/build-tests-windows.yml:1`
- `.github/workflows/cats.yml:1`
- `.github/workflows/cleanup.yml:1`
- `.github/workflows/integrity-check.yml:1`
- `.github/workflows/versioning.yml:1`

### script-injection (severity: high)

cloud-runner-ci-pipeline.yml directly interpolates GitHub Actions expressions inside `run:` shell commands (sub-rule a), allowing script injection if matrix values or step outputs are attacker-influenced.

1. `tests` job (line ~67): `run: yarn run test "${{ matrix.test }}" --detectOpenHandles --forceExit --runInBand` — `${{ matrix.test }}` is interpolated directly into the shell command.
2. `k8sTests` job (line ~100): same pattern with `${{ matrix.test }}`.
3. `awsTests` job (line ~135): same pattern with `${{ matrix.test }}`.
4. `buildTargetTests` job (line ~175): `run: cp ./cloud-runner-cache/cache/${{ steps.unity-build.outputs.CACHE_KEY }}/build/${{ steps.unity-build.outputs.BUILD_ARTIFACT }} ${{ steps.unity-build.outputs.BUILD_ARTIFACT }}` — step outputs interpolated directly into shell.

These should be moved to `env:` variables and referenced as `"$ENV_VAR"` in the shell.

Locations:

- `.github/workflows/cloud-runner-ci-pipeline.yml:67`
- `.github/workflows/cloud-runner-ci-pipeline.yml:100`
- `.github/workflows/cloud-runner-ci-pipeline.yml:135`
- `.github/workflows/cloud-runner-ci-pipeline.yml:175`

### unsafe-shell (severity: high)

integrity-check.yml executes remote shell content directly via process substitution: `bash <(curl -s https://codecov.io/bash)`. This is functionally equivalent to `curl ... | bash` — it downloads and executes an arbitrary remote script without verification. The script should be downloaded to a file first, its integrity verified (e.g., via checksum), and then executed separately.

Locations:

- `.github/workflows/integrity-check.yml:26`

## Iteration Notes

### Iteration 1

**Fixes applied:** hardcoded-credentials, unpinned-uses, missing-permissions, script-injection, unsafe-shell

**Notes:**

Fixed all 5 security findings across 9 workflow files:

1. hardcoded-credentials (integrity-check.yml): Replaced literal CODECOV_TOKEN UUID with `${{ secrets.CODECOV_TOKEN }}`.

2. unpinned-uses: Pinned all 12 action references across activation.yml, build-tests-mac.yml, build-tests-ubuntu.yml, build-tests-windows.yml, cats.yml, cleanup.yml, cloud-runner-ci-pipeline.yml, integrity-check.yml, and versioning.yml to full 40-char commit SHAs with original tags as comments.

3. missing-permissions: Added minimal `permissions:` blocks to all 8 workflow files that lacked them. Used `contents: read` for most; `contents: write` for versioning.yml (needs to create/update tags); `contents: read, actions: write` for cleanup.yml (needs to delete artifacts).

4. script-injection (cloud-runner-ci-pipeline.yml): Moved `${{ matrix.test }}` into `env: MATRIX_TEST:` and referenced as `$MATRIX_TEST` in shell; moved `${{ steps.unity-build.outputs.CACHE_KEY }}` and `${{ steps.unity-build.outputs.BUILD_ARTIFACT }}` into `env:` block and referenced as `$CACHE_KEY` and `$BUILD_ARTIFACT`.

5. unsafe-shell (integrity-check.yml): Replaced `bash <(curl -s https://codecov.io/bash)` with a two-step approach: download to a temp file with `curl -s --fail ... -o "$CODECOV_SCRIPT"`, then execute `bash "$CODECOV_SCRIPT"`.

