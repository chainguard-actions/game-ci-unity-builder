<!-- markdownlint-disable -->

# Hardening Report: game-ci--unity-builder/v4.7.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **game-ci--unity-builder/v4.7.0** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### hardcoded-credentials (severity: high)

A hardcoded Unity license XML is assigned to the `UNITY_LICENSE` environment variable at the top-level `env:` block. This is a literal credential value, not a `${{ secrets.* }}` reference. The license contains machine bindings, serial hash, and a cryptographic signature.

Locations:

- `.github/workflows/build-tests-ubuntu.yml:13`

### hardcoded-credentials (severity: high)

A hardcoded Codecov token UUID (`2f2eb890-30e2-4724-83eb-7633832cf0de`) is assigned to `CODECOV_TOKEN` in the top-level `env:` block. This is a literal credential value that should be stored in GitHub Secrets and referenced as `${{ secrets.CODECOV_TOKEN }}`.

Locations:

- `.github/workflows/integrity-check.yml:6`

### script-injection (severity: high)

Sub-rule (a): The expression `${{ matrix.test }}` is interpolated directly inside a `run:` shell command string: `run: yarn run test "${{ matrix.test }}" --detectOpenHandles --forceExit --runInBand`. The `matrix.test` value flows through YAML template substitution before the shell sees it, enabling script injection if the matrix value contains shell metacharacters. This pattern appears in three separate jobs (tests, k8sTests, awsTests).

Locations:

- `.github/workflows/cloud-runner-ci-pipeline.yml:62`
- `.github/workflows/cloud-runner-ci-pipeline.yml:103`
- `.github/workflows/cloud-runner-ci-pipeline.yml:143`

### script-injection (severity: high)

Sub-rule (a): The expressions `${{ steps.unity-build.outputs.CACHE_KEY }}` and `${{ steps.unity-build.outputs.BUILD_ARTIFACT }}` are interpolated directly inside a `run:` shell command: `cp ./cloud-runner-cache/cache/${{ steps.unity-build.outputs.CACHE_KEY }}/build/${{ steps.unity-build.outputs.BUILD_ARTIFACT }} ${{ steps.unity-build.outputs.BUILD_ARTIFACT }}`. Step outputs are workflow-controllable and must not be interpolated directly into shell commands.

Locations:

- `.github/workflows/cloud-runner-ci-pipeline.yml:175`

### unsafe-shell (severity: high)

The step `run: bash <(curl -s https://codecov.io/bash)` downloads a remote shell script via `curl` and executes it directly in bash using process substitution. This is equivalent to `curl ... | bash` and executes untrusted remote content without any integrity verification (no checksum or signature check).

Locations:

- `.github/workflows/integrity-check.yml:17`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags or version strings instead of pinned 40-character commit SHAs. Unpinned references are vulnerable to supply-chain attacks if the upstream tag is moved or the repository is compromised. Failing references include: `game-ci/unity-request-activation-file@v2.0-alpha-1`, `actions/upload-artifact@v4`, `actions/checkout@v4`, `actions/cache@v4`, `jlumbroso/free-disk-space@v1.3.1`, `ruairidhwm/action-cats@1.0.2`, `kolpav/purge-artifacts-action@v1`, `actions/setup-node@v4`, `aws-actions/configure-aws-credentials@v1`, `actions/checkout@v2`, `debianmaster/actions-k3s@v1.0.5`, `Actions-R-Us/actions-tagger@v2`.

Locations:

- `.github/workflows/activation.yml:12`
- `.github/workflows/activation.yml:15`
- `.github/workflows/build-tests-mac.yml:35`
- `.github/workflows/build-tests-mac.yml:43`
- `.github/workflows/build-tests-mac.yml:72`
- `.github/workflows/build-tests-ubuntu.yml:68`
- `.github/workflows/build-tests-ubuntu.yml:74`
- `.github/workflows/build-tests-ubuntu.yml:82`
- `.github/workflows/build-tests-ubuntu.yml:152`
- `.github/workflows/build-tests-windows.yml:42`
- `.github/workflows/build-tests-windows.yml:50`
- `.github/workflows/build-tests-windows.yml:101`
- `.github/workflows/cats.yml:11`
- `.github/workflows/cleanup.yml:9`
- `.github/workflows/cleanup.yml:14`
- `.github/workflows/cleanup.yml:16`
- `.github/workflows/cloud-runner-ci-pipeline.yml:50`
- `.github/workflows/cloud-runner-ci-pipeline.yml:52`
- `.github/workflows/cloud-runner-ci-pipeline.yml:91`
- `.github/workflows/cloud-runner-ci-pipeline.yml:93`
- `.github/workflows/cloud-runner-ci-pipeline.yml:131`
- `.github/workflows/cloud-runner-ci-pipeline.yml:133`
- `.github/workflows/cloud-runner-ci-pipeline.yml:163`
- `.github/workflows/cloud-runner-ci-pipeline.yml:172`
- `.github/workflows/cloud-runner-ci-pipeline.yml:183`
- `.github/workflows/cloud-runner-ci-pipeline.yml:184`
- `.github/workflows/integrity-check.yml:14`
- `.github/workflows/integrity-check.yml:15`
- `.github/workflows/versioning.yml:9`

### missing-permissions (severity: medium)

These workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, workflows inherit the default repository permissions (which may be `write-all` depending on repository settings), violating the principle of least privilege.

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

Fixed all 7 finding categories across 8 workflow files:

1. hardcoded-credentials: Removed hardcoded Unity license XML from build-tests-ubuntu.yml (replaced with ${{ secrets.UNITY_LICENSE }}) and hardcoded Codecov token UUID from integrity-check.yml (replaced with ${{ secrets.CODECOV_TOKEN }}).

2. script-injection: In cloud-runner-ci-pipeline.yml, moved ${{ matrix.test }} into env: blocks as MATRIX_TEST for all 3 jobs (tests, k8sTests, awsTests). Also moved ${{ steps.unity-build.outputs.CACHE_KEY }} and ${{ steps.unity-build.outputs.BUILD_ARTIFACT }} into env: blocks for the cp command in buildTargetTests.

3. unsafe-shell: Replaced `bash <(curl -s https://codecov.io/bash)` in integrity-check.yml with a safe pattern that downloads to a temp file first then executes it.

4. unpinned-uses: Pinned all 12 action references to full 40-character commit SHAs across activation.yml, build-tests-mac.yml, build-tests-ubuntu.yml, build-tests-windows.yml, cats.yml, cleanup.yml, cloud-runner-ci-pipeline.yml, integrity-check.yml, and versioning.yml.

5. missing-permissions: Added permissions blocks to all 8 workflow files listed in the findings. Used contents: read as the baseline, with contents: write for versioning.yml (actions-tagger needs to write tags) and actions: write for cleanup.yml (purge-artifacts needs to delete artifacts).

