<!-- markdownlint-disable -->

# Hardening Report: hashicorp--setup-terraform/v2.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **hashicorp--setup-terraform/v2.0.3** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct ${{ expression interpolation inside run: shell commands. In setup-terraform.yml, multiple steps interpolate ${{ matrix['terraform-versions'] }} and ${{ env.TF_CLOUD_API_TOKEN }} and ${{ steps.plan.outputs.stdout }} directly into run: blocks, allowing shell metacharacter injection. Offending lines include: `run: terraform version | grep ${{ matrix['terraform-versions']}}` (lines 32 and 58), `grep 'token = "${{ env.TF_CLOUD_API_TOKEN }}"'` (lines 124, 130, 154, 160), and `run: echo "${{ steps.plan.outputs.stdout }}"` (line 280).

Locations:

- `.github/workflows/setup-terraform.yml:32`
- `.github/workflows/setup-terraform.yml:58`
- `.github/workflows/setup-terraform.yml:124`
- `.github/workflows/setup-terraform.yml:130`
- `.github/workflows/setup-terraform.yml:154`
- `.github/workflows/setup-terraform.yml:160`
- `.github/workflows/setup-terraform.yml:280`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable version tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks. Failing references: add-content-to-project.yml uses `leonsteinhaeuser/project-beta-automations@v2.0.1` (lines 22, 31); continuous-integration.yml uses `actions/checkout@v3` (line 18) and `actions/setup-node@v3` (line 21); release.yml uses `actions/checkout@v3` (line 15); setup-terraform.yml uses `actions/checkout@v3` at multiple steps (e.g. line 23).

Locations:

- `.github/workflows/add-content-to-project.yml:22`
- `.github/workflows/add-content-to-project.yml:31`
- `.github/workflows/continuous-integration.yml:18`
- `.github/workflows/continuous-integration.yml:21`
- `.github/workflows/release.yml:15`
- `.github/workflows/setup-terraform.yml:23`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` keys, meaning they run with the default (potentially broad) token permissions. This is especially concerning for add-content-to-project.yml which uses the sensitive `pull_request_target` trigger. Files affected: continuous-integration.yml, setup-terraform.yml, add-content-to-project.yml.

Locations:

- `.github/workflows/continuous-integration.yml:1`
- `.github/workflows/setup-terraform.yml:1`
- `.github/workflows/add-content-to-project.yml:1`

### hardcoded-credentials (severity: high)

setup-terraform.yml hardcodes a literal token value in the `TF_CLOUD_API_TOKEN` env variable: `'XXXXXXXXXXXXXX.atlasv1.XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'`. Although this appears to be a placeholder/dummy value, it is a non-expression literal assigned to a token-named field and matches the hardcoded-credentials pattern. It should be replaced with a secrets expression such as `${{ secrets.TF_CLOUD_API_TOKEN }}`.

Locations:

- `.github/workflows/setup-terraform.yml:110`
- `.github/workflows/setup-terraform.yml:139`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions, hardcoded-credentials

**Notes:**

Fixed all four findings across four workflow files:

1. script-injection (setup-terraform.yml): Moved all ${{ }} expressions out of run: blocks into step-level env: blocks. matrix['terraform-versions'] → TF_VERSION, env.TF_CLOUD_API_TOKEN → TF_TOKEN, steps.plan.outputs.stdout → PLAN_STDOUT.

2. unpinned-uses: Pinned all mutable tag references to full 40-char SHAs with tag comments: actions/checkout@v3 → f43a0e5ff2bd294095638e18286ca9a3d1956744, actions/setup-node@v3 → 3235b876344d2a9aa001b8d1453c930bba69e610, leonsteinhaeuser/project-beta-automations@v2.0.1 → 7f947733020ee03daa363d16ea1223717b132f11.

3. missing-permissions: Added top-level 'permissions: contents: read' to setup-terraform.yml, continuous-integration.yml, and add-content-to-project.yml.

4. hardcoded-credentials: Replaced the literal placeholder token string in TF_CLOUD_API_TOKEN env vars (lines 110 and 139) with ${{ secrets.TF_CLOUD_API_TOKEN }} in both terraform-credentials-cloud and terraform-credentials-enterprise jobs.

