<!-- markdownlint-disable -->

# Hardening Report: hashicorp--setup-terraform/v4.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **hashicorp--setup-terraform/v4.0.0** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in release.yml directly interpolate ${{ }} expressions into shell commands, enabling script injection. Affected steps:
- Line 21: `run: echo "version=$(echo "${{ inputs.versionNumber }}" | cut -d. -f1)" >> "$GITHUB_OUTPUT"` — rule (a), inputs.versionNumber interpolated directly.
- Line 27: `run: echo "version=$(echo "${{ inputs.versionNumber }}" | cut -c 2-)" >> "$GITHUB_OUTPUT"` — rule (a), inputs.versionNumber interpolated directly.
- Line 56 (Git push changelog): `git config --global user.name "${{ vars.TF_DEVEX_CI_COMMIT_AUTHOR }}"` and `git push "https://${{ vars.TF_DEVEX_CI_COMMIT_AUTHOR }}:${{ secrets.TF_DEVEX_COMMIT_GITHUB_TOKEN }}@github.com/${{ github.repository }}.git"` — rule (a), vars.* and github.* interpolated directly.
- Line 68: `run: npm version "${{ inputs.versionNumber }}" --git-tag-version false` — rule (a), inputs.versionNumber interpolated directly.
- Lines 75–76 (Git push): same vars.* and secrets.* pattern as above.
- Lines 84–90 (Git push release tag): `git tag "${{ inputs.versionNumber }}"`, `git tag -f "${{ needs.major-version.outputs.version }}"`, and git push with inputs.versionNumber and needs.*.outputs.* — rule (a).
- Line 100 (Generate Release Notes): `sed ... ${{ needs.changelog-version.outputs.version }}.md` — rule (a), needs.*.outputs.* interpolated directly and unquoted.
- Lines 103–104 (GH Release): `gh release create "${{ inputs.versionNumber }}" ... --title "${{ inputs.versionNumber }}"` — rule (a).

Locations:

- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:27`
- `.github/workflows/release.yml:56`
- `.github/workflows/release.yml:68`
- `.github/workflows/release.yml:75`
- `.github/workflows/release.yml:84`
- `.github/workflows/release.yml:100`
- `.github/workflows/release.yml:103`

### script-injection (severity: high)

In setup-terraform.yml, a run: block interpolates a matrix expression directly and unquoted into a shell command: `run: terraform version | grep ${{ matrix['terraform-versions']}}`. The matrix value is unquoted, allowing shell metacharacter injection — rule (a). The same pattern appears in two jobs (terraform-versions and terraform-versions-no-wrapper).

Locations:

- `.github/workflows/setup-terraform.yml:35`
- `.github/workflows/setup-terraform.yml:71`

### script-injection (severity: high)

In issue-comment-triage.yml, a run: block directly interpolates ${{ env.COMMAND }} and ${{ github.event.issue.html_url }} into a shell command: `run: gh ${{ env.COMMAND }} edit ${{ github.event.issue.html_url }} --remove-label waiting-response`. Both expressions are interpolated without quoting or env-var indirection — rule (a). github.event.issue.html_url is attacker-controlled via issue comments.

Locations:

- `.github/workflows/issue-comment-triage.yml:22`

### github-env-injection (severity: high)

In release.yml, two run: steps write ${{ inputs.versionNumber }} directly to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker-supplied version string containing newlines could inject arbitrary environment variables or outputs:
- Line 21: `echo "version=$(echo "${{ inputs.versionNumber }}" | cut -d. -f1)" >> "$GITHUB_OUTPUT"`
- Line 27: `echo "version=$(echo "${{ inputs.versionNumber }}" | cut -c 2-)" >> "$GITHUB_OUTPUT"`

Locations:

- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:27`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` on any of their jobs, meaning they run with the default (potentially broad) token permissions:
- continuous-integration.yml: no permissions block at all.
- setup-terraform.yml: no permissions block at all.
- lock.yml: no permissions block at all.
- issue-comment-triage.yml: no permissions block at all.

Locations:

- `.github/workflows/continuous-integration.yml:1`
- `.github/workflows/setup-terraform.yml:1`
- `.github/workflows/lock.yml:1`
- `.github/workflows/issue-comment-triage.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all findings across 5 workflow files:

1. release.yml (script-injection + github-env-injection): Moved all ${{ inputs.versionNumber }}, ${{ vars.* }}, ${{ secrets.* }}, ${{ github.* }}, and ${{ needs.*.outputs.* }} expressions from run: blocks into step-level env: blocks. For the two GITHUB_OUTPUT writes (major-version and changelog-version jobs), added sanitization via `printf '%s' "$VERSION_NUMBER" | tr -d '\n\r'` before writing to prevent newline injection.

2. setup-terraform.yml (script-injection + missing-permissions): Fixed two `terraform version | grep ${{ matrix['terraform-versions']}}` occurrences by adding env: blocks with TF_VERSION and using `grep "$TF_VERSION"`. Added top-level `permissions: contents: read`.

3. issue-comment-triage.yml (script-injection + missing-permissions): Moved `github.event.issue.html_url` into a step-level env: block (ISSUE_URL) and used `gh "$COMMAND" edit "$ISSUE_URL"`. Added top-level `permissions: issues: write, pull-requests: write`.

4. continuous-integration.yml (missing-permissions): Added top-level `permissions: contents: read`.

5. lock.yml (missing-permissions): Added top-level `permissions: issues: write, pull-requests: write` (required for the lock-threads action to lock issues and PRs).

### Iteration 2

**Fixes applied:** script-injection, hardcoded-credentials

**Notes:**

Fixed in .github/workflows/setup-terraform.yml: (1) hardcoded-credentials: Replaced the literal placeholder token 'XXXXXXXXXXXXXX.atlasv1.XXX...' with ${{ secrets.TF_CLOUD_API_TOKEN }} in both terraform-credentials-cloud and terraform-credentials-enterprise jobs. (2) script-injection: In the four 'Validate Credentials' steps (Windows and Linux/macOS variants in both jobs), moved ${{ env.TF_CLOUD_API_TOKEN }} into step-level env blocks as TOKEN and referenced $TOKEN in the grep shell commands. Also fixed the 'Print Terraform Plan' step by moving ${{ steps.plan.outputs.stdout }} into a step-level env block as PLAN_STDOUT and referencing $PLAN_STDOUT in the echo command.

