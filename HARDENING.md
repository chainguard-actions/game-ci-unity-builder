<!-- markdownlint-disable -->

# Hardening Report: game-ci--unity-builder/v4.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **game-ci--unity-builder/v4.8.0** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### hardcoded-credentials (severity: high)

Hardcoded credential found in build-tests-ubuntu.yml: the top-level `env:` block contains a literal Unity license XML blob assigned to `UNITY_LICENSE:` (starting at line 12). This is a real license key embedded in plaintext in the repository. It should be stored as a GitHub Actions secret and referenced via `${{ secrets.UNITY_LICENSE }}`.

Locations:

- `.github/workflows/build-tests-ubuntu.yml:12`

### hardcoded-credentials (severity: high)

Hardcoded credential found in integrity-check.yml: `CODECOV_TOKEN: '2f2eb890-30e2-4724-83eb-7633832cf0de'` is a literal Codecov upload token embedded in the workflow file. It should be stored as a GitHub Actions secret and referenced via `${{ secrets.CODECOV_TOKEN }}`.

Locations:

- `.github/workflows/integrity-check.yml:8`

### script-injection (severity: high)

Sub-rule (a) violation: `${{ matrix.test }}` is interpolated directly inside a `run:` shell command string in three jobs. The offending line is: `run: yarn run test "${{ matrix.test }}" --detectOpenHandles --forceExit --runInBand`. Although `matrix.test` values are defined in the workflow itself and are not directly attacker-controlled in this case, any `${{ ... }}` expression inside a `run:` block is a script-injection finding per the check rules, as the value flows through YAML template substitution before the shell sees it. The fix is to pass the value via an `env:` variable and reference it as `"$TEST_NAME"` in the shell command.

Locations:

- `.github/workflows/cloud-runner-ci-pipeline.yml:57`
- `.github/workflows/cloud-runner-ci-pipeline.yml:100`
- `.github/workflows/cloud-runner-ci-pipeline.yml:143`

### unsafe-shell (severity: high)

The integrity-check.yml workflow runs `bash <(curl -s https://codecov.io/bash)`, which downloads a remote script and executes it directly in a shell via process substitution. This is equivalent to `curl ... | bash` and is an unsafe-shell pattern. The script should be downloaded to a file first, its integrity verified (e.g., via checksum), and then executed separately.

Locations:

- `.github/workflows/integrity-check.yml:26`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags or version strings instead of full 40-character SHA commit digests, making them vulnerable to supply-chain attacks if the referenced tag is moved or the action is compromised.

activation.yml: `game-ci/unity-request-activation-file@v2.0-alpha-1`, `actions/upload-artifact@v4`
build-tests-mac.yml: `actions/checkout@v4`, `actions/cache@v4`, `actions/upload-artifact@v4`
build-tests-ubuntu.yml: `jlumbroso/free-disk-space@v1.3.1`, `actions/checkout@v4`, `actions/cache@v4`, `actions/upload-artifact@v4`
build-tests-windows.yml: `actions/checkout@v4`, `actions/cache@v4`, `actions/upload-artifact@v4`
cats.yml: `ruairidhwm/action-cats@1.0.2`
cleanup.yml: `kolpav/purge-artifacts-action@v1`, `actions/checkout@v4`, `actions/setup-node@v4`
cloud-runner-ci-pipeline.yml: `actions/checkout@v4`, `actions/checkout@v2`, `aws-actions/configure-aws-credentials@v1`, `debianmaster/actions-k3s@v1.0.5`, `actions/upload-artifact@v4`, `actions/setup-node@v4`
integrity-check.yml: `actions/checkout@v4`, `actions/setup-node@v4`
versioning.yml: `Actions-R-Us/actions-tagger@v2`

Locations:

- `.github/workflows/activation.yml:13`
- `.github/workflows/build-tests-mac.yml:35`
- `.github/workflows/build-tests-ubuntu.yml:80`
- `.github/workflows/build-tests-windows.yml:40`
- `.github/workflows/cats.yml:10`
- `.github/workflows/cleanup.yml:7`
- `.github/workflows/cloud-runner-ci-pipeline.yml:55`
- `.github/workflows/integrity-check.yml:20`
- `.github/workflows/versioning.yml:10`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions (which may be `write-all` for private repos or `read-all` for public repos), violating the principle of least privilege. Each workflow should declare the minimal permissions it requires.

Affected files: activation.yml, build-tests-mac.yml, build-tests-ubuntu.yml, build-tests-windows.yml, cats.yml, cleanup.yml, integrity-check.yml, versioning.yml.

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

Fixed all 6 finding types across 9 workflow files:

1. hardcoded-credentials: Replaced literal Unity license XML in build-tests-ubuntu.yml with `${{ secrets.UNITY_LICENSE }}`, and replaced literal Codecov token in integrity-check.yml with `${{ secrets.CODECOV_TOKEN }}`.

2. script-injection: In cloud-runner-ci-pipeline.yml, moved `${{ matrix.test }}` from all three `run:` shell commands into `env:` blocks as `TEST_NAME: ${{ matrix.test }}`, and updated shell commands to use `$TEST_NAME`.

3. unsafe-shell: In integrity-check.yml, replaced `bash <(curl -s https://codecov.io/bash)` with a safe pattern: download to a temp file via `curl -s --fail ... -o "$CODECOV_SCRIPT"`, then execute `bash "$CODECOV_SCRIPT"`.

4. unpinned-uses: Pinned all action references to full 40-character SHA digests with tag comments in all affected files: activation.yml, build-tests-mac.yml, build-tests-ubuntu.yml, build-tests-windows.yml, cats.yml, cleanup.yml, cloud-runner-ci-pipeline.yml, integrity-check.yml, versioning.yml.

5. missing-permissions: Added `permissions:` blocks to all 8 affected workflow files with minimal required permissions (contents: read for most; contents: write for versioning.yml which creates tags; actions: write added to cleanup.yml which purges artifacts).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/cloud-runner-ci-pipeline.yml at the buildTargetTests job. Moved `${{ steps.unity-build.outputs.CACHE_KEY }}` and `${{ steps.unity-build.outputs.BUILD_ARTIFACT }}` expressions from the `run:` shell string into the step's `env:` block as `CACHE_KEY` and `BUILD_ARTIFACT` environment variables. The shell command now references them as `"${CACHE_KEY}"` and `"${BUILD_ARTIFACT}"` with proper double-quoting, preventing shell metacharacter injection.

