<!-- markdownlint-disable -->

# Hardening Report: hashicorp--setup-terraform/v2.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **hashicorp--setup-terraform/v2.0.3** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in setup-terraform.yml directly interpolate GitHub Actions expressions inside shell commands (sub-rule a). (1) `run: terraform version | grep ${{ matrix['terraform-versions']}}` — the matrix value is interpolated directly into the shell command without quoting, allowing shell metacharacter injection. This appears in both the `terraform-versions` and `terraform-versions-no-wrapper` jobs. (2) `run: echo "${{ steps.plan.outputs.stdout }}"` — step output is interpolated directly into the shell command; terraform plan output is attacker-influenced and can contain arbitrary content. (3) `cat ... | grep 'token = "${{ env.TF_CLOUD_API_TOKEN }}"'` — env context expression interpolated directly inside run blocks in the credentials validation steps.

Locations:

- `.github/workflows/setup-terraform.yml:31`
- `.github/workflows/setup-terraform.yml:50`
- `.github/workflows/setup-terraform.yml:130`
- `.github/workflows/setup-terraform.yml:134`
- `.github/workflows/setup-terraform.yml:168`
- `.github/workflows/setup-terraform.yml:172`
- `.github/workflows/setup-terraform.yml:208`

### hardcoded-credentials (severity: high)

Two job-level `env:` blocks in setup-terraform.yml assign a literal token string to `TF_CLOUD_API_TOKEN`: `TF_CLOUD_API_TOKEN: 'XXXXXXXXXXXXXX.atlasv1.XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'`. This is a hardcoded credential matching the pattern `token: <literal-value>`. Even if the value is a placeholder/test token, it is a literal non-expression value assigned to a name containing 'token'. It should be replaced with a secrets expression such as `${{ secrets.TF_CLOUD_API_TOKEN }}`.

Locations:

- `.github/workflows/setup-terraform.yml:100`
- `.github/workflows/setup-terraform.yml:148`

### unpinned-uses (severity: high)

All `uses:` references across workflow files use mutable tag refs instead of full 40-character commit SHA pins, making the workflows vulnerable to supply-chain attacks if the referenced tags are moved or compromised. Failing references include: `actions/checkout@v3` (multiple files), `actions/setup-node@v3` (continuous-integration.yml), `leonsteinhaeuser/project-beta-automations@v2.0.1` (add-content-to-project.yml, two steps).

Locations:

- `.github/workflows/continuous-integration.yml:14`
- `.github/workflows/continuous-integration.yml:18`
- `.github/workflows/release.yml:12`
- `.github/workflows/setup-terraform.yml:22`
- `.github/workflows/setup-terraform.yml:41`
- `.github/workflows/setup-terraform.yml:63`
- `.github/workflows/setup-terraform.yml:83`
- `.github/workflows/add-content-to-project.yml:20`
- `.github/workflows/add-content-to-project.yml:30`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs, meaning the workflows run with the default (broad) GitHub token permissions. `add-content-to-project.yml` is particularly sensitive as it uses `pull_request_target` (which grants write access to the base repo) with no permissions restriction. `continuous-integration.yml` and `setup-terraform.yml` also lack any permissions declaration.

Locations:

- `.github/workflows/add-content-to-project.yml:1`
- `.github/workflows/continuous-integration.yml:1`
- `.github/workflows/setup-terraform.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, hardcoded-credentials, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings across four workflow files:

1. script-injection (setup-terraform.yml): Moved all ${{ }} expressions out of run: blocks into env: blocks. Specifically: matrix['terraform-versions'] → TF_VERSION env var (terraform-versions and terraform-versions-no-wrapper jobs); steps.plan.outputs.stdout → PLAN_STDOUT env var (terraform-run-local job); env.TF_CLOUD_API_TOKEN → EXPECTED_TOKEN env var (terraform-credentials-cloud and terraform-credentials-enterprise jobs).

2. hardcoded-credentials (setup-terraform.yml): Replaced literal token 'XXXXXXXXXXXXXX.atlasv1.XXX...' in both terraform-credentials-cloud and terraform-credentials-enterprise job-level env blocks with ${{ secrets.TF_CLOUD_API_TOKEN }}.

3. unpinned-uses: Pinned all mutable tag references to full commit SHAs — actions/checkout@v3 → @f43a0e5ff2bd294095638e18286ca9a3d1956744 (setup-terraform.yml, continuous-integration.yml, release.yml); actions/setup-node@v3 → @3235b876344d2a9aa001b8d1453c930bba69e610 (continuous-integration.yml); leonsteinhaeuser/project-beta-automations@v2.0.1 → @7f947733020ee03daa363d16ea1223717b132f11 (add-content-to-project.yml).

4. missing-permissions: Added permissions: {} top-level block to setup-terraform.yml, continuous-integration.yml, and add-content-to-project.yml. release.yml already had a permissions block.

