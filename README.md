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

| Update type | PRs | Merge |
| --- | --- | --- |
| Patch (production dependencies) | One PR per package. | Automerged once CI passes. |
| Minor (production dependencies) | One PR per package. | Human review. |
| Minor and patch (npm devDependencies) | Grouped under **dev dependencies (non-major)**. | Automerged once CI passes. |
| Minor and patch (GitHub Actions) | Grouped under **GitHub Actions (non-major)**. | Automerged once CI passes. |
| Major | Listed in the Dependency Dashboard issue. Check the approval box before Renovate creates the branch/PR. | Human review. |
| Security fix | Raised when a fix is available, bypassing the schedule, release-age delay, and dashboard approval. Labeled `security`. | Human review. |

Packages that release together (for example `@babel/*`) are grouped by `config:recommended`.

### 3-day release age

Renovate does not propose a version until it has been released for **3 days**, in every ecosystem (`minimumReleaseAge`). This gives malware scanners and maintainers time to catch and pull a compromised release. Security fixes skip the wait.

- **Missing release dates:** if a registry does not publish release timestamps, Renovate holds those updates and lists them as pending in the Dependency Dashboard. Add a per-package rule if one needs to go through.
- **Transitive dependencies:** Renovate only checks the versions it writes into manifests. In pnpm repositories, also set pnpm's `minimumReleaseAge` in `pnpm-workspace.yaml` so indirect dependencies resolved during install get the same wait.

Routine update branches are created **Monday between midnight and 6 a.m. IST** (`Asia/Kolkata`), unless a repository overrides the timezone or schedule. This is an allowed window, not a job trigger: Renovate must run during it. Existing branches can be updated outside the window.

Renovate's default limits (10 open PRs, 2 new PRs per hour) keep an outdated repository from opening dozens of PRs at once; the rest wait in the Dependency Dashboard and are created as PRs merge. A repository catching up on a large backlog can lower this temporarily with `"prConcurrentLimit": 3`.

### Automerge

- **CI is the reviewer.** Renovate merges only after the branch has passing status checks. A repository with no CI gets the PRs but never automerges them.
- **Merge window:** weekdays, 10 a.m. to 5 p.m. IST, so someone is around if `main` breaks.
- **One merge per run.** Renovate merges itself (`platformAutomerge: false`) so it can check CI and honor the window; it merges one PR per repository per run, so a backlog drains over several runs.
- **Failing PRs stay open** and point at a single package. Those are the only Renovate PRs that need a person, besides minor production updates, majors, and security fixes.
- **Weekly sweep:** search GitHub for `is:pr is:open author:app/renovate org:<org>` to see every open Renovate PR across repositories, and `is:issue is:open author:app/renovate "Dependency Dashboard" org:<org>` to list every repository's dashboard and check for majors awaiting approval.

Dashboard approval allows Renovate to create an update PR; it does not merge it.

To turn automerge off in one repository:

```json
{
  "extends": ["github>vidluther/renovate-config"],
  "packageRules": [{ "matchPackageNames": ["*"], "automerge": false }]
}
```

## GitHub Actions: pinned, but still updated

**Renovate proposes GitHub Actions updates even when Dependabot update PRs are disabled.** Pinning fixes the code your workflow runs; it does not stop Renovate from checking for newer versions.

The `helpers:pinGitHubActionDigestsToSemver` preset replaces action tags with exact commit SHAs and records the full version in a comment:

```yaml
# Before pinning
- uses: actions/checkout@v4

# After pinning
- uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4.2.2
```

A tag can move upstream; a pinned SHA keeps the action code unchanged until you merge an update. Because the comment holds a full version, action updates are classified as patch, minor, or major and follow the policy above. Major updates appear in the Dependency Dashboard first.

New PRs use **GitHub's normal notifications**. Dashboard changes do **not** notify: Renovate edits the issue body, and GitHub does not send notifications for edits. New majors awaiting approval are only found by checking the dashboard (see the weekly sweep above).

Reusable workflow calls, such as `owner/repo/.github/workflows/build.yml@v1`, are pinned the same way. Local workflows (`uses: ./.github/workflows/build.yml`) have no ref and are left alone.

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
- `security:minimumReleaseAgeNpm` exempts npm lock file maintenance and package replacements from the release-age wait, where it cannot be checked.
- `:configMigration` opens a PR when a repository's Renovate config uses deprecated options.

## Development

The `Validate` workflow runs `renovate-config-validator --strict` on `default.json` and this repository's own `renovate.json` for every push to `main` and every PR. This repository extends its own preset, so its workflow actions are pinned and updated like any other repository's.

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
