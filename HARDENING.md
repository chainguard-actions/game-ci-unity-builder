<!-- markdownlint-disable -->

# Hardening Report: game-ci--unity-builder/v5.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **game-ci--unity-builder/v5.0.1** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### hardcoded-credentials (severity: high)

A literal CODECOV_TOKEN UUID is hardcoded as a top-level env var: `CODECOV_TOKEN: '2f2eb890-30e2-4724-83eb-7633832cf0de'`. This exposes the token to anyone with read access to the repository and should be stored in GitHub Secrets instead.

Locations:

- `.github/workflows/integrity-check.yml:13`

### hardcoded-credentials (severity: high)

A full Unity license XML blob (containing a SerialHash, DeveloperData, and RSA SignatureValue) is hardcoded as the top-level env var `UNITY_LICENSE`. This embeds a real license credential directly in the workflow file. It should be stored in a GitHub Secret.

Locations:

- `.github/workflows/build-tests-ubuntu.yml:8`

### script-injection (severity: high)

Sub-rule (a): Multiple `${{ ... }}` expressions are interpolated directly inside `run:` shell command strings in the 'Create test project' and 'Record result' steps. Specifically: `${{ matrix.source }}` is used in an `if` condition, `${{ matrix.name }}` and `${{ matrix.package }}` are interpolated into a Python one-liner string argument, `${{ matrix.unity }}` is written into a heredoc, and `${{ steps.build.outcome }}` is assigned to a shell variable — all without any quoting or sanitization at the YAML template level. An attacker who can influence the matrix values (e.g. via a malicious community-plugins.yml entry or workflow_dispatch input) could inject arbitrary shell commands.

Locations:

- `.github/workflows/validate-community-plugins.yml:75`
- `.github/workflows/validate-community-plugins.yml:88`
- `.github/workflows/validate-community-plugins.yml:97`
- `.github/workflows/validate-community-plugins.yml:107`
- `.github/workflows/validate-community-plugins.yml:113`

### missing-permissions (severity: medium)

These workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any job. Without explicit permissions, workflows inherit the repository's default token permissions (which may be `write-all`), violating the principle of least privilege.

Locations:

- `.github/workflows/activation.yml:1`
- `.github/workflows/build-tests-mac.yml:1`
- `.github/workflows/build-tests-ubuntu.yml:1`
- `.github/workflows/build-tests-windows.yml:1`
- `.github/workflows/cats.yml:1`
- `.github/workflows/versioning.yml:1`

### unpinned-uses (severity: high)

All `uses:` references across workflow files are pinned to mutable tags or version strings rather than immutable 40-character commit SHAs. This exposes the workflows to supply-chain attacks if any referenced action's tag is moved or compromised. Failing references include: `game-ci/unity-request-activation-file@v2.0-alpha-1`, `actions/upload-artifact@v4`, `actions/checkout@v4`, `actions/cache@v4`, `actions/setup-node@v4`, `jlumbroso/free-disk-space@v1.3.1`, `ruairidhwm/action-cats@1.0.2`, `actions/github-script@v7`, `azure/setup-kubectl@v4`, `Actions-R-Us/actions-tagger@v2`.

Locations:

- `.github/workflows/activation.yml:11`
- `.github/workflows/activation.yml:14`
- `.github/workflows/build-tests-mac.yml:36`
- `.github/workflows/build-tests-mac.yml:43`
- `.github/workflows/build-tests-mac.yml:79`
- `.github/workflows/build-tests-ubuntu.yml:79`
- `.github/workflows/build-tests-ubuntu.yml:85`
- `.github/workflows/build-tests-ubuntu.yml:92`
- `.github/workflows/build-tests-ubuntu.yml:143`
- `.github/workflows/build-tests-windows.yml:43`
- `.github/workflows/build-tests-windows.yml:50`
- `.github/workflows/build-tests-windows.yml:107`
- `.github/workflows/cats.yml:9`
- `.github/workflows/integrity-check.yml:23`
- `.github/workflows/integrity-check.yml:27`
- `.github/workflows/integrity-check.yml:33`
- `.github/workflows/validate-community-plugins.yml:29`
- `.github/workflows/validate-community-plugins.yml:33`
- `.github/workflows/validate-community-plugins.yml:68`
- `.github/workflows/validate-community-plugins.yml:120`
- `.github/workflows/validate-community-plugins.yml:123`
- `.github/workflows/validate-orchestrator-integration.yml:50`
- `.github/workflows/validate-orchestrator-integration.yml:57`
- `.github/workflows/validate-orchestrator-integration.yml:68`
- `.github/workflows/validate-orchestrator-integration.yml:75`
- `.github/workflows/validate-orchestrator-integration.yml:81`
- `.github/workflows/validate-orchestrator-integration.yml:196`
- `.github/workflows/validate-orchestrator.yml:62`
- `.github/workflows/validate-orchestrator.yml:66`
- `.github/workflows/validate-orchestrator.yml:79`
- `.github/workflows/versioning.yml:9`

### unsafe-shell (severity: high)

Remote content is piped directly to a shell interpreter without first downloading and verifying it. In `integrity-check.yml`, `bash <(curl -s https://codecov.io/bash)` fetches and immediately executes a remote script. In `validate-orchestrator-integration.yml`, `curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash` does the same for the k3d installer. Both patterns allow a compromised or man-in-the-middle remote server to execute arbitrary code on the runner.

Locations:

- `.github/workflows/integrity-check.yml:38`
- `.github/workflows/validate-orchestrator-integration.yml:213`

## Iteration Notes

### Iteration 1

**Fixes applied:** hardcoded-credentials, script-injection, missing-permissions, unpinned-uses, unsafe-shell

**Notes:**

Fixed all 6 finding types across 8 workflow files:

1. hardcoded-credentials: Removed hardcoded CODECOV_TOKEN UUID from integrity-check.yml (now uses secrets.CODECOV_TOKEN) and removed hardcoded UNITY_LICENSE XML blob from build-tests-ubuntu.yml (now uses secrets.UNITY_LICENSE like the other build workflows).

2. script-injection: In validate-community-plugins.yml, moved all ${{ matrix.* }} and ${{ steps.build.outcome }} expressions from run: shell strings into step env: blocks. The Python manifest-editing script now reads MATRIX_NAME and MATRIX_PACKAGE via os.environ instead of inline interpolation. The ProjectVersion.txt heredoc was replaced with printf using the MATRIX_UNITY env var.

3. missing-permissions: Added permissions blocks to activation.yml (contents: read), build-tests-mac.yml (contents: read), build-tests-ubuntu.yml (contents: read), build-tests-windows.yml (contents: read), cats.yml (contents: read), and versioning.yml (contents: write for tag updates).

4. unpinned-uses: Pinned all 10 distinct action references to full 40-character commit SHAs across all 8 affected workflow files, preserving the original tag as a comment.

5. unsafe-shell: Fixed 3 pipe-to-shell patterns: (a) bash <(curl ...) in integrity-check.yml converted to download-then-execute, (b) curl ... | bash for k3d installer in validate-orchestrator-integration.yml converted to download-then-execute, (c) curl ... | sudo bash for rclone installer also converted to download-then-execute.

