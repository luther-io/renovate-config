# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) preset for my repositories, used instead of Dependabot.

## Usage

Add a `renovate.json` to the repository:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["github>vidluther/renovate-config"]
}
```

Renovate resolves `github>vidluther/renovate-config` to `default.json` on the default branch.

## What `default.json` does

- `config:recommended` plus the dependency dashboard issue
- Weekly run: PRs are created early on Monday
- Minor and patch updates grouped into a single PR per manager
- Major updates require approval from the dependency dashboard before a PR is opened
- GitHub Actions pinned to commit digests via `helpers:pinGitHubActionDigests`; reusable workflow references stay on tags
- Vulnerability PRs from two sources: OSV (`osvVulnerabilityAlerts`, direct dependencies) and GitHub's Dependabot alerts (`vulnerabilityAlerts`, includes transitive), the latter labelled `security`

## Per-repository overrides

Extend the preset and add rules; later entries win:

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

## Notes

- Extends `helpers:pinGitHubActionDigests`, which needs the Renovate app's *Workflows: read and write* permission.
- Dependabot is disabled in the adopting repositories: no `.github/dependabot.yml`, and Dependabot security updates turned off. Dependabot alerts stay enabled as a second source for `vulnerabilityAlerts`.
