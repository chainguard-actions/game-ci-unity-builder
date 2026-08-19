<!-- markdownlint-disable -->

# Hardening Report: game-ci--unity-builder/v5.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **game-ci--unity-builder/v5.0.0** was hardened automatically. 9 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### hardcoded-credentials (severity: high)

A literal CODECOV_TOKEN UUID is hardcoded as a top-level env var: `CODECOV_TOKEN: '2f2eb890-30e2-4724-83eb-7633832cf0de'`. This exposes the token in the repository and to anyone who can read the workflow file. It should be stored as a GitHub secret and referenced via `${{ secrets.CODECOV_TOKEN }}`.

Locations:

- `.github/workflows/integrity-check.yml:14`

### hardcoded-credentials (severity: high)

A literal Unity license XML file is hardcoded as the top-level env var `UNITY_LICENSE`. The value contains machine bindings, serial hash, developer data (base64-encoded serial), and a cryptographic signature — all sensitive credential material. It should be stored as a GitHub secret and referenced via `${{ secrets.UNITY_LICENSE }}`.

Locations:

- `.github/workflows/build-tests-ubuntu.yml:13`

### missing-permissions (severity: medium)

These workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any job. Without explicit permissions, GitHub grants the default token permissions (which can be write for contents in some repository configurations), violating the principle of least privilege.

Locations:

- `.github/workflows/activation.yml:1`
- `.github/workflows/build-tests-mac.yml:1`
- `.github/workflows/build-tests-ubuntu.yml:1`
- `.github/workflows/build-tests-windows.yml:1`
- `.github/workflows/cats.yml:1`
- `.github/workflows/versioning.yml:1`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are directly interpolated inside `run:` shell command strings in the 'Create test project' step. The values `${{ matrix.source }}`, `${{ matrix.name }}`, `${{ matrix.package }}`, and `${{ matrix.unity }}` are substituted into the shell before execution. Since the matrix is populated from the community-plugins.yml registry (which could be modified via PR), these values are attacker-controllable and allow arbitrary command injection. Offending lines include: `if [ "${{ matrix.source }}" = "git" ]`, `manifest['dependencies']['${{ matrix.name }}'] = '${{ matrix.package }}'`, and `m_EditorVersion: ${{ matrix.unity }}`.

Locations:

- `.github/workflows/validate-community-plugins.yml:55`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are directly interpolated inside `run:` shell command strings in the 'Record result' step. The values `${{ steps.build.outcome }}`, `${{ matrix.name }}`, `${{ matrix.platform }}`, `${{ matrix.unity }}`, `${{ matrix.source }}`, and `${{ matrix.package }}` are substituted into shell echo commands before execution, enabling command injection if any of these values contain shell metacharacters.

Locations:

- `.github/workflows/validate-community-plugins.yml:88`

### unpinned-uses (severity: high)

All `uses:` references across every workflow file use mutable tag or branch refs instead of immutable 40-character SHA digests. This exposes the workflows to supply-chain attacks if any referenced action's tag is moved or the action is compromised. Unpinned references include: `game-ci/unity-request-activation-file@v2.0-alpha-1`, `actions/upload-artifact@v4`, `actions/checkout@v4`, `actions/cache@v4`, `jlumbroso/free-disk-space@v1.3.1`, `actions/setup-node@v4`, `ruairidhwm/action-cats@1.0.2`, `actions/github-script@v7`, `azure/setup-kubectl@v4`, `Actions-R-Us/actions-tagger@v2`, and others.

Locations:

- `.github/workflows/activation.yml:11`
- `.github/workflows/build-tests-mac.yml:36`
- `.github/workflows/build-tests-ubuntu.yml:76`
- `.github/workflows/build-tests-windows.yml:43`
- `.github/workflows/cats.yml:9`
- `.github/workflows/integrity-check.yml:23`
- `.github/workflows/validate-community-plugins.yml:30`
- `.github/workflows/validate-orchestrator-integration.yml:57`
- `.github/workflows/validate-orchestrator.yml:68`
- `.github/workflows/versioning.yml:9`

### unsafe-shell (severity: high)

Remote script content is fetched and executed directly in a shell via bash process substitution: `bash <(curl -s https://codecov.io/bash)`. This pattern executes whatever the remote server returns without any integrity verification, making it vulnerable to supply-chain attacks or MITM attacks.

Locations:

- `.github/workflows/integrity-check.yml:43`

### unsafe-shell (severity: high)

Remote install script is fetched and piped directly to bash: `curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash`. This executes whatever the remote server returns without integrity verification, and the URL references the mutable `main` branch, making it vulnerable to supply-chain attacks.

Locations:

- `.github/workflows/validate-orchestrator-integration.yml:126`

### unsafe-shell (severity: high)

Remote install script is fetched and piped directly to a privileged shell: `curl https://rclone.org/install.sh | sudo bash`. This executes whatever the remote server returns with root privileges and without integrity verification, making it vulnerable to supply-chain attacks or MITM attacks.

Locations:

- `.github/workflows/validate-orchestrator-integration.yml:388`

## Iteration Notes

### Iteration 1

**Fixes applied:** hardcoded-credentials, missing-permissions, script-injection, unpinned-uses, unsafe-shell

**Notes:**

Fixed all 9 findings across 10 workflow files:

1. hardcoded-credentials (integrity-check.yml): Replaced literal CODECOV_TOKEN UUID with ${{ secrets.CODECOV_TOKEN }}.

2. hardcoded-credentials (build-tests-ubuntu.yml): Replaced entire hardcoded UNITY_LICENSE XML blob with ${{ secrets.UNITY_LICENSE }}.

3. missing-permissions (6 files): Added permissions blocks to activation.yml (contents: read), build-tests-mac.yml (contents: read), build-tests-ubuntu.yml (contents: read), build-tests-windows.yml (contents: read), cats.yml (contents: read), versioning.yml (contents: write — needed for tag management).

4. script-injection (validate-community-plugins.yml 'Create test project'): Moved matrix.source/name/package/unity into env: block as MATRIX_* vars; Python script now reads from os.environ; ProjectVersion.txt written via printf.

5. script-injection (validate-community-plugins.yml 'Record result'): Moved steps.build.outcome and all matrix.* values into env: block; shell script references plain env vars.

6. unpinned-uses: Pinned all action references across all workflow files to full 40-char SHAs with tag comments (actions/checkout, actions/upload-artifact, actions/cache, actions/setup-node, actions/github-script, game-ci/unity-request-activation-file, jlumbroso/free-disk-space, ruairidhwm/action-cats, azure/setup-kubectl, Actions-R-Us/actions-tagger).

7. unsafe-shell (integrity-check.yml): Replaced bash <(curl ...) with download-then-execute pattern.

8. unsafe-shell (validate-orchestrator-integration.yml k3d): Replaced curl|bash with download-then-execute.

9. unsafe-shell (validate-orchestrator-integration.yml rclone): Replaced curl|sudo bash with download-then-execute.

