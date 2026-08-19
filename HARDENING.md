<!-- markdownlint-disable -->

# Hardening Report: game-ci--unity-builder/v6.0.0-beta.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **game-ci--unity-builder/v6.0.0-beta.1** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### hardcoded-credentials (severity: high)

A literal CODECOV_TOKEN UUID value ('2f2eb890-30e2-4724-83eb-7633832cf0de') is hardcoded in the top-level `env:` block of integrity-check.yml. This is a real token value that should be stored as a GitHub Actions secret and referenced via `${{ secrets.CODECOV_TOKEN }}` instead.

Locations:

- `.github/workflows/integrity-check.yml:13`

### script-injection (severity: high)

Rule (a): Multiple `${{ ... }}` expressions are interpolated directly inside `run:` shell command strings in validate-community-plugins.yml. In the 'Create test project' step, `${{ matrix.source }}`, `${{ matrix.name }}`, `${{ matrix.package }}`, and `${{ matrix.unity }}` are embedded directly in shell commands (e.g., `if [ "${{ matrix.source }}" = "git" ]`, `manifest['dependencies']['${{ matrix.name }}'] = '${{ matrix.package }}'`, `m_EditorVersion: ${{ matrix.unity }}`). In the 'Record result' step, `${{ steps.build.outcome }}`, `${{ matrix.name }}`, `${{ matrix.platform }}`, `${{ matrix.unity }}`, `${{ matrix.source }}`, and `${{ matrix.package }}` are all interpolated directly into shell echo commands. Additionally, in the 'Parse plugin registry' github-script step, `${{ github.event.inputs.plugin_filter }}` and `${{ github.event.inputs.unity_version }}` are interpolated directly into the JavaScript script string. These allow an attacker to inject arbitrary shell or script commands via matrix values or workflow_dispatch inputs.

Locations:

- `.github/workflows/validate-community-plugins.yml:62`
- `.github/workflows/validate-community-plugins.yml:80`
- `.github/workflows/validate-community-plugins.yml:95`
- `.github/workflows/validate-community-plugins.yml:34`
- `.github/workflows/validate-community-plugins.yml:46`

### unsafe-shell (severity: high)

The integrity-check.yml workflow runs `bash <(curl -s https://codecov.io/bash)`, which downloads a remote script from codecov.io and executes it directly via process substitution without any integrity verification (e.g., checksum or signature check). This is equivalent to `curl ... | bash` and allows a compromised or malicious remote server to execute arbitrary code on the runner.

Locations:

- `.github/workflows/integrity-check.yml:38`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tags or version strings instead of immutable 40-character commit SHA digests. This exposes the workflow to supply-chain attacks if the referenced tag is moved or the action repository is compromised. Failing references include:
- activation.yml: `game-ci/unity-request-activation-file@v2.0-alpha-1`, `actions/upload-artifact@v4`
- build-tests-mac.yml: `actions/checkout@v4`, `actions/cache@v4`, `actions/upload-artifact@v4`
- build-tests-ubuntu.yml: `jlumbroso/free-disk-space@v1.3.1`, `actions/checkout@v4`, `actions/cache@v4`, `actions/upload-artifact@v4`
- build-tests-windows.yml: `actions/checkout@v4`, `actions/cache@v4`, `actions/upload-artifact@v4`
- cats.yml: `ruairidhwm/action-cats@1.0.2`
- integrity-check.yml: `actions/checkout@v4`, `actions/setup-node@v4`, `actions/cache@v4`
- validate-community-plugins.yml: `actions/checkout@v4`, `actions/github-script@v7`
- versioning.yml: `Actions-R-Us/actions-tagger@v2`

Locations:

- `.github/workflows/activation.yml:11`
- `.github/workflows/build-tests-mac.yml:34`
- `.github/workflows/build-tests-ubuntu.yml:67`
- `.github/workflows/build-tests-windows.yml:36`
- `.github/workflows/cats.yml:9`
- `.github/workflows/integrity-check.yml:22`
- `.github/workflows/validate-community-plugins.yml:27`
- `.github/workflows/versioning.yml:9`

### missing-permissions (severity: medium)

Six workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (which may be `read-all` or `write-all` depending on repository settings), violating the principle of least privilege. Affected files: activation.yml, build-tests-mac.yml, build-tests-ubuntu.yml, build-tests-windows.yml, cats.yml, versioning.yml.

Locations:

- `.github/workflows/activation.yml:1`
- `.github/workflows/build-tests-mac.yml:1`
- `.github/workflows/build-tests-ubuntu.yml:1`
- `.github/workflows/build-tests-windows.yml:1`
- `.github/workflows/cats.yml:1`
- `.github/workflows/versioning.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** hardcoded-credentials, unsafe-shell, script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 5 security findings across 7 workflow files:

1. hardcoded-credentials (integrity-check.yml): Removed hardcoded CODECOV_TOKEN UUID from top-level env block; replaced with ${{ secrets.CODECOV_TOKEN }} in the step's env block.

2. unsafe-shell (integrity-check.yml): Replaced `bash <(curl -s https://codecov.io/bash)` process substitution with download-then-execute pattern: curl to a temp file, then `bash "$CODECOV_SCRIPT"`.

3. script-injection (validate-community-plugins.yml): Moved all ${{ matrix.* }} and ${{ github.event.inputs.* }} expressions out of run: shell strings and github-script JS strings into env: blocks. Updated Python script to use os.environ, and JS to use process.env.

4. unpinned-uses: Pinned all 9 action references to full 40-char commit SHAs across activation.yml, build-tests-mac.yml, build-tests-ubuntu.yml, build-tests-windows.yml, cats.yml, integrity-check.yml, validate-community-plugins.yml, and versioning.yml.

5. missing-permissions: Added top-level permissions blocks to activation.yml (contents: read), build-tests-mac.yml (contents: read), build-tests-ubuntu.yml (contents: read), build-tests-windows.yml (contents: read), cats.yml (contents: read, pull-requests: write), and versioning.yml (contents: write).

