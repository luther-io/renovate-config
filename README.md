# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) preset for my repositories. Renovate handles dependency and GitHub Actions update PRs instead of Dependabot updates; Dependabot alerts stay enabled for security detection.

## Usage

Install the Renovate GitHub App for the repository, then add a `renovate.json`:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>vidluther/renovate-config"]
}
```

Renovate resolves `github>vidluther/renovate-config` to `default.json` on the default branch.

The app needs **Workflows: read and write** permission to update GitHub Actions workflow files.

## Update policy

| Update type | Behavior |
| --- | --- |
| Minor and patch | Grouped under **all non-major dependencies**. The group can span package managers; it is not one PR per manager. |
| Major | Listed in the Dependency Dashboard issue. Check the approval box before Renovate creates the branch/PR. |
| Action digest (SHA) only | Can get separate PRs; the minor/patch grouping rule does not cover digest-only updates. |
| Security fix | Raised when a fix is available. Vulnerability-alert PRs bypass the normal schedule and dashboard approval. |

Routine update branches are created **Monday between midnight and 6 a.m. UTC**, unless a repository overrides the timezone or schedule. This is an allowed window, not a job trigger: Renovate must run during it. Existing branches can be updated outside the window.

This preset does **not** enable automerge. Dashboard approval allows Renovate to create an update PR; it does not merge it.

## GitHub Actions: pinned, but still updated

**Renovate proposes GitHub Actions updates even when Dependabot update PRs are disabled.** Pinning fixes the code your workflow runs; it does not stop Renovate from checking for newer versions.

The `helpers:pinGitHubActionDigests` preset replaces action tags with exact commit SHAs, preserving the version in a comment:

```yaml
# Before pinning
- uses: actions/checkout@v4

# After pinning
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4
```

A tag can move upstream; a pinned SHA keeps the action code unchanged until you merge an update. Renovate tracks the version comment and proposes SHA/version updates through PRs using the policy above. Major updates appear in the Dependency Dashboard first.

PRs and dashboard issue updates use **GitHub's normal notifications**, so notification delivery depends on your GitHub subscription settings.

Reusable workflow calls, such as `owner/repo/.github/workflows/build.yml@v1`, are excluded from automatic SHA pinning. Renovate can still propose version-tag updates for them.

## Security updates and Dependabot

Renovate uses two vulnerability sources:

| Source | Coverage |
| --- | --- |
| OSV (`osvVulnerabilityAlerts`) | Direct dependencies in supported ecosystems. |
| GitHub Dependabot alerts (`vulnerabilityAlerts`) | Can include transitive dependencies. These PRs receive the `security` label. |

In adopting repositories:

- **Disable Dependabot update PRs:** remove `.github/dependabot.yml` and turn off Dependabot security updates to avoid duplicate PRs.
- **Keep Dependency graph and Dependabot alerts enabled:** Renovate reads those alerts to propose fixes. The Renovate app needs read access to Dependabot alerts.

These are repository settings; extending this preset does not change them automatically.

## Other defaults

- `config:recommended` provides Renovate's recommended defaults and the Dependency Dashboard issue.
- `platformCommit: "enabled"` creates commits through GitHub's API so the GitHub App's commits are signed, supporting repositories that require signed commits.

## Per-repository overrides

Extend the preset and add rules. Later matching rules override earlier values; for example, allow one package's major-update PRs without dashboard approval:

```json
{
  "extends": ["github>vidluther/renovate-config"],
  "packageRules": [
    {
      "matchPackageNames": ["some-package"],
      "dependencyDashboardApproval": false
    }
  ]
}
```
