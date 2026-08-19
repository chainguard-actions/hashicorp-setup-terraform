<!-- markdownlint-disable -->

# Hardening Report: hashicorp--setup-terraform/v4.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **hashicorp--setup-terraform/v4.0.1** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

release.yml has multiple run: blocks that directly interpolate ${{ }} expressions into shell commands (sub-rule a). Offending lines: line 21 interpolates ${{ inputs.versionNumber }} in an echo to $GITHUB_OUTPUT; line 28 same; line 52 interpolates ${{ vars.TF_DEVEX_CI_COMMIT_AUTHOR }} in git config; line 55 interpolates ${{ vars.* }}, ${{ secrets.TF_DEVEX_COMMIT_GITHUB_TOKEN }}, and ${{ github.repository }} in a git push URL; line 79 interpolates ${{ inputs.versionNumber }} in npm version; line 103 interpolates ${{ inputs.versionNumber }} in git tag; line 107 interpolates ${{ inputs.versionNumber }} and ${{ needs.major-version.outputs.version }} in git push; line 119 interpolates ${{ needs.changelog-version.outputs.version }} unquoted in a sed command; line 123 interpolates ${{ inputs.versionNumber }} in gh release create. All allow shell metacharacter injection before the shell parses the command.

Locations:

- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:28`
- `.github/workflows/release.yml:52`
- `.github/workflows/release.yml:55`
- `.github/workflows/release.yml:79`
- `.github/workflows/release.yml:103`
- `.github/workflows/release.yml:107`
- `.github/workflows/release.yml:119`
- `.github/workflows/release.yml:123`

### script-injection (severity: high)

setup-terraform.yml has run: blocks that directly interpolate ${{ matrix['terraform-versions'] }} into shell commands (sub-rule a). Line 32: `run: terraform version | grep ${{ matrix['terraform-versions']}}` in the terraform-versions job. Line 49: same pattern in the terraform-versions-no-wrapper job. The matrix value flows through YAML template substitution before the shell sees it, allowing metacharacter injection.

Locations:

- `.github/workflows/setup-terraform.yml:32`
- `.github/workflows/setup-terraform.yml:49`

### script-injection (severity: high)

issue-comment-triage.yml has a run: block that directly interpolates ${{ env.COMMAND }} and ${{ github.event.issue.html_url }} into a shell command (sub-rule a). Line 20: `run: gh ${{ env.COMMAND }} edit ${{ github.event.issue.html_url }} --remove-label waiting-response`. github.event.issue.html_url is attacker-controllable via issue comment events. Both expressions are substituted by the YAML template engine before the shell parses the command.

Locations:

- `.github/workflows/issue-comment-triage.yml:20`

### github-env-injection (severity: high)

release.yml writes ${{ inputs.versionNumber }} directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). A versionNumber value containing newlines could inject arbitrary key=value pairs into the GitHub output environment. Line 21: `echo "version=$(echo "${{ inputs.versionNumber }}" | cut -d. -f1)" >> "$GITHUB_OUTPUT"`. Line 28: `echo "version=$(echo "${{ inputs.versionNumber }}" | cut -c 2-)" >> "$GITHUB_OUTPUT"`. The YAML template engine substitutes the value before the shell runs, so shell quoting does not prevent newline injection into the environment file.

Locations:

- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:28`

### missing-permissions (severity: medium)

setup-terraform.yml has no top-level permissions: key and none of its many jobs (terraform-versions, terraform-versions-no-wrapper, terraform-versions-constraints, terraform-versions-constraints-no-wrapper, terraform-credentials-cloud, terraform-credentials-enterprise, terraform-credentials-none, terraform-arguments, terraform-arguments-no-wrapper, terraform-run-local, terraform-run-local-no-wrapper, terraform-stdout-wrapper, terraform-stdout-no-wrapper, terraform-wrapper-delayed-apply) define job-level permissions. The workflow runs with default (potentially broad) GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/setup-terraform.yml:1`

### missing-permissions (severity: medium)

issue-comment-triage.yml has no top-level permissions: key and its only job (issue_comment_triage) has no job-level permissions block. The workflow runs with default GITHUB_TOKEN permissions despite performing label-editing operations on issues and PRs.

Locations:

- `.github/workflows/issue-comment-triage.yml:1`

### missing-permissions (severity: medium)

lock.yml has no top-level permissions: key and its only job (lock) has no job-level permissions block. The workflow runs with default GITHUB_TOKEN permissions despite performing lock operations on issues and PRs.

Locations:

- `.github/workflows/lock.yml:1`

### missing-permissions (severity: medium)

continuous-integration.yml has no top-level permissions: key and neither of its jobs (check-dist, test) defines job-level permissions. The workflow runs with default GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/continuous-integration.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all findings across 5 workflow files:

1. release.yml (script-injection + github-env-injection): Moved all ${{ }} expressions from run: blocks into env: blocks. For GITHUB_OUTPUT writes (lines 21, 28), added sanitization via `printf '%s' "$VERSION_NUMBER" | tr -d '\n\r'` before writing. All git config, git push URL, npm version, git tag, sed, and gh release create commands now reference plain env vars.

2. setup-terraform.yml (script-injection + missing-permissions): Added `permissions: contents: read` at top level. Fixed lines 32 and 49 by moving `${{ matrix['terraform-versions'] }}` into `env: TF_VERSION:` and referencing `"$TF_VERSION"` in the grep command.

3. issue-comment-triage.yml (script-injection + missing-permissions): Added `permissions: issues: write` and `pull-requests: write`. Moved `${{ github.event.issue.html_url }}` into step-level `env: ISSUE_URL:` and changed run command to use `"$COMMAND"` and `"$ISSUE_URL"` as plain env vars.

4. lock.yml (missing-permissions): Added `permissions: issues: write` and `pull-requests: write` needed for the lock-threads action.

5. continuous-integration.yml (missing-permissions): Added `permissions: contents: read` at top level.

### Iteration 2

**Fixes applied:** script-injection, hardcoded-credentials

**Notes:**

Fixed all findings in .github/workflows/setup-terraform.yml:

1. hardcoded-credentials (lines 117, 147): Replaced hardcoded literal token 'XXXXXXXXXXXXXX.atlasv1.XXX...' with ${{ secrets.TF_CLOUD_API_TOKEN }} in both terraform-credentials-cloud and terraform-credentials-enterprise job env blocks.

2. script-injection (lines 130, 135, 160, 165): Moved ${{ env.TF_CLOUD_API_TOKEN }} out of the four grep run: commands into step-level env: blocks as TOKEN, then referenced $TOKEN in the shell scripts.

3. script-injection (line 271): Moved ${{ steps.plan.outputs.stdout }} out of the echo run: command into a step-level env: block as PLAN_STDOUT, then referenced $PLAN_STDOUT in the shell script.

