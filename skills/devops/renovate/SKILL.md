---
name: renovate
description: 'Use when creating or modifying Renovate configuration for automated dependency management. Covers renovate.json config, GitHub Actions workflow, and Forgejo Actions workflow templates for supply-chain hardened dependency updates.'
author: hugobatista
---

## When to use Renovate

Use Renovate for supply-chain hardened dependency management — action SHA pinning, Docker digest pinning, minimum release age, and OSV vulnerability alerts. Works on GitHub, Forgejo, Gitea, GitLab, and other platforms.

## Config template

Read `assets/renovate.json.tmpl`, replace `<timezone>` with your IANA timezone (e.g. `"Europe/Lisbon"`), and write it to the project root as `renovate.json`.

Read `config.toml` from the skill's config directory (`~/.config/opencode/skills-config/renovate/` if it exists, otherwise the skill directory). If it exists, extract `renovate_git_author` and `renovate_timezone` for defaults. If not, ask the user for their git author identity and timezone, and write both values to `config.toml` (one-time setup).

Before creating the config, ask the user for the **git author identity** for Renovate commits. Default: `<renovate_git_author from config>`. This email must be verified on the user's GitHub account to avoid "Unverified" commit badges. Replace `<git-author>` with the chosen value (e.g. `"Renovate Bot <bot@example.com>"`).

Key settings:

- `pinDigests: true` — converts `@v4` → `@<sha> # v4` for actions and `image:tag` → `image:tag@sha256:digest` for Docker. Keeps digests pinned and auto-updated.
- `minimumReleaseAge: "30 days"` — prevents brand-new releases from being proposed until a vetting window passes.
- `osvVulnerabilityAlerts: true` — monitors OSV.dev for known CVEs.
- `dependencyDashboard: true` — creates a visibility issue tracking all pending, blocked, and ignored updates.
- `lockFileMaintenance.enabled: true` — refreshes transitive dependencies. Do **not** add a narrow schedule unless you have verified that the workflow actually runs inside that window.
- `packageRules` — major updates get a `major-update` label, security updates get priority, and digest/pinDigest updates keep the 30-day gate.

### Labels

The template assumes these labels exist in the repository:

- `dependencies`
- `major-update`
- `security`

Create them before the first Renovate run, or remove the corresponding `labels` entries.

### Platform-specific additions to renovate.json

**For Forgejo** (with GitHub action mirroring at data.forgejo.org):

```json
"hostRules": [
  {
    "matchHost": "data.forgejo.org",
    "hostType": "github"
  }
]
```

**For GitHub**: no special hostRules needed.

### Docker compose support

Add the following if the project uses docker-compose:

```json
"docker-compose": {
  "enabled": true,
  "managerFilePatterns": [
    "/(^|/)(?:compose|docker-compose)[^/]*\\.ya?ml(?:[^/]*\\.example)?$/"
  ]
}
```

This captures both `docker-compose.yml` and example files like `docker-compose.yml.example` or `docker-compose.yml.secrets.example`.

The old `fileMatch` key is deprecated; use `managerFilePatterns`.

### Custom managers

Custom regex managers can handle edge cases not covered by built-in managers. See https://docs.renovatebot.com/configuration-options/#custommanagers.

#### Forgejo mirrored actions

Actions mirrored from `data.forgejo.org` may not be resolvable through the `forgejo-tags` datasource. Route them through `git-refs` against the upstream GitHub repository instead:

```json
{
  "customType": "regex",
  "description": "Track actions/checkout via git-refs",
  "managerFilePatterns": ["/^\\.forgejo/workflows/.+\\.ya?ml$/"],
  "matchStrings": [
    "uses:\\s+https://data\\.forgejo\\.org/actions/checkout@(?<currentDigest>[a-f0-9]{40})\\s+#\\s+(?<currentValue>v\\d+\\.\\d+\\.\\d+)"
  ],
  "depNameTemplate": "https://github.com/actions/checkout.git",
  "datasourceTemplate": "git-refs",
  "versioningTemplate": "semver"
}
```

When using these custom managers, disable the duplicate `github-actions` manager entries and the `pep621` Python lookup:

```json
{
  "matchManagers": ["github-actions"],
  "matchPackageNames": ["actions/checkout", "astral-sh/setup-uv"],
  "enabled": false
},
{
  "matchManagers": ["pep621"],
  "matchDepNames": ["python"],
  "enabled": false
}
```

`git-refs` uses `git ls-remote`, so it needs no GitHub token and avoids GitHub API rate limits.

#### Own Docker image

If the project publishes its own image and references it in compose examples with a floating tag, exempt it from digest pinning:

```json
{
  "matchPackageNames": ["ghcr.io/owner/project"],
  "pinDigests": false
}
```

Otherwise Renovate replaces `:latest` in the examples with an immutable digest, which defeats the purpose of the example.

## Workflow templates

### GitHub (self-hosted)

Read `assets/renovate.yml.github.tmpl` and write it to `.github/workflows/renovate.yml` (no substitutions — pinned defaults for `actions/checkout` and `renovatebot/renovate`). Check for newer versions > 30 days old and update if needed.

The template pins `renovatebot/renovate` to `44.151.0` and `actions/checkout` to `v7.0.1`. Renovate will update its own image and the checkout action once it runs, subject to `minimumReleaseAge`.

The workflow separates env-var setup into dedicated steps so that `act --env VAR=value` overrides pass through for local testing.

Requires `RENOVATE_TOKEN` as a repository secret. Create a **fine-grained PAT** at:

```
https://github.com/settings/personal-access-tokens/new
```

| Permission | Level |
|---|---|
| Commit statuses | Read and write |
| Contents | Read and write |
| Dependabot alerts | Read-only |
| Issues | Read and write |
| Metadata | Read-only |
| Pull requests | Read and write |
| Workflows | Read and write |

`Workflows` is required if Renovate should update files under `.github/workflows/`. Without it, action updates fail.

For organisation-owned repositories, also grant `Members: Read-only` at the organisation level.

A classic PAT with `repo` and `workflow` scopes also works but grants broader access.

### Forgejo (self-hosted)

Read `assets/renovate.yml.forgejo.tmpl`. Replace `<forgejo-instance>` with your instance hostname (e.g. `codeberg.org`) and `<runner-label>` with your runner tag (e.g. `codeberg-small`). Write the result to `.forgejo/workflows/renovate.yml` (no substitutions for checkout or renovate — pinned defaults). Check for newer versions > 30 days old and update if needed.

Forgejo ignores the `permissions` key; the template documents this and relies on the token scopes instead.

Uses the `$FORGEJO_ENV` pattern so the env-var steps can be overridden via `forgejo-runner exec --env` for local testing.

Requires `RENOVATE_TOKEN` as a repository secret.

## Docker digests and release timestamps

`minimumReleaseAge` only applies to digest and pinDigest updates when the datasource exposes a release timestamp.

- Docker Hub exposes `tag_last_pushed`, so digest updates for `python`, `node`, etc. are aged correctly.
- GHCR, Quay, and ECR do not expose release timestamps. With the default `minimumReleaseAgeBehaviour=timestamp-required`, digest updates for those images are held indefinitely.

For images without timestamps, either raise them without an age check:

```json
{
  "matchDatasources": ["docker"],
  "matchPackageNames": ["ghcr.io/your-org/your-image"],
  "matchUpdateTypes": ["digest"],
  "minimumReleaseAge": null
}
```

Or require manual approval through the Dependency Dashboard:

```json
{
  "matchDatasources": ["docker"],
  "matchPackageNames": ["ghcr.io/your-org/your-image"],
  "matchUpdateTypes": ["major", "minor", "patch"],
  "minimumReleaseAgeBehaviour": "timestamp-optional",
  "dependencyDashboardApproval": true
}
```

The second form detects the update but creates a PR only after you approve it in the dashboard. Use it for build tools where you want a human gate but no brittle date-based schedule.

## Lockfile maintenance scheduling

`lockFileMaintenance.schedule` is evaluated in the Renovate `timezone` when the workflow runs. GitHub Actions scheduled workflows are often delayed by minutes or hours, so a narrow window can never match.

Bad combination:

```yaml
# workflow
schedule:
  - cron: '0 4 * * *'
```

```json
"lockFileMaintenance": {
  "enabled": true,
  "schedule": ["before 4am on monday"]
}
```

The job starts at 04:00 UTC or later, while the Renovate window ends at 04:00. The dashboard shows "Awaiting Schedule" forever.

Prefer:

```json
"lockFileMaintenance": {
  "enabled": true
}
```

The workflow cadence now controls frequency: daily workflow means daily lockfile check, weekly workflow means weekly. Renovate only opens a PR when the lockfile actually changes. If you need a weekly cadence, change the workflow cron (e.g. `0 4 * * 1`) instead of adding a narrow Renovate schedule.

## uv image and `required-version`

If the project pins `[tool.uv] required-version` to an exact version and the Dockerfile uses `ghcr.io/astral-sh/uv:<same>@sha256:...`, Renovate must not bump the image tag without also updating `pyproject.toml`. A bare version is treated as exact (`0.11.18` means `==0.11.18`), so the build fails with:

```text
error: Required uv version `==0.11.18` does not match the running version `0.13.7`.
```

Options:

1. **Range (recommended when automation is wanted):**
   ```toml
   required-version = ">=0.11.18"
   ```
   The Docker image can move to a newer uv while the old version still satisfies the range. Verify the lockfile is compatible with the new uv before merging (`uv lock --check`, `uv sync --frozen`, `uv export`).
2. **Exact pin + manual bumps:** keep the exact version, disable Renovate version updates for the uv image, and bump the Dockerfile, `required-version`, and `uv.lock` together.
3. **Exact pin + custom manager + post-upgrade task:** automate both files and run `uv lock`, but this requires `postUpgradeTasks` and is more fragile.

When using a range, `setup-uv` resolves it. Choose a resolution strategy:

```yaml
- name: Install uv
  uses: astral-sh/setup-uv@<sha> # vX.Y.Z
  with:
    resolution-strategy: lowest
```

- `lowest` keeps CI on the lower bound (the vetted version).
- `highest` (default) installs the latest uv and acts as a canary.

If CI uses `lowest`, bump the lower bound in the same PR as the Docker image so CI and the image test the same version.

## Validation

Validate the repository config locally:

```bash
npx --yes --package renovate -- renovate-config-validator
```

Run a local dry-run with `act` (GitHub):

```bash
act workflow_dispatch -W .github/workflows/renovate.yml \
  --env RENOVATE_DRY_RUN=full \
  --env RENOVATE_REPOSITORIES=owner/repo \
  --env LOG_LEVEL=info \
  -s RENOVATE_TOKEN=<token>
```

Replace `owner/repo` with your repository path and `<token>` with a GitHub classic PAT with `repo` and `workflow` scopes.

## README update

After creating the workflow file, add the Renovate badge near the other badges:

```markdown
[![Renovate](https://img.shields.io/badge/renovate-enabled-brightgreen?logo=renovatebot)](https://docs.renovatebot.com)
```

Then add a short "Dependency updates" section so contributors can run Renovate locally:

**For GitHub projects:**

```bash
act workflow_dispatch -W .github/workflows/renovate.yml \
  --env RENOVATE_DRY_RUN=full \
  --env RENOVATE_REPOSITORIES=owner/repo \
  --env LOG_LEVEL=info \
  -s RENOVATE_TOKEN=<token>
```

**For Forgejo projects:**

```bash
forgejo-runner exec \
  -W .forgejo/workflows/renovate.yml \
  --env RENOVATE_DRY_RUN=full \
  --env RENOVATE_REPOSITORIES=owner/repo \
  --env LOG_LEVEL=info \
  -s RENOVATE_TOKEN=<token>
```

Replace `owner/repo` with your repository path and `<token>` with a GitHub/Forgejo PAT with `repo` and `workflow` scopes.

## Troubleshooting

- **Dependency Dashboard shows "Awaiting Schedule" forever:** the Renovate schedule does not overlap with the workflow run time. Remove the narrow schedule or widen it.
- **No PR for a GHCR digest update:** GHCR has no release timestamp. Use `minimumReleaseAgeBehaviour: "timestamp-optional"` or `minimumReleaseAge: null` for that package.
- **Renovate cannot update workflow files:** the PAT is missing `Workflows: Read and write`.
- **Renovate fails to add labels:** create the labels in the repository first.
- **Docker build fails after a uv image bump:** `required-version` is still exact. Use a range or update both files together.
