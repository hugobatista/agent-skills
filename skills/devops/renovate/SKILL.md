---
name: renovate
description: 'Use when creating or modifying Renovate configuration for automated dependency management. Covers renovate.json config, GitHub Actions workflow, and Forgejo Actions workflow templates for supply-chain hardened dependency updates.'
author: hugobatista
---

## When to use Renovate

Use Renovate for supply-chain hardened dependency management — action SHA pinning, Docker digest pinning, minimum release age, and OSV vulnerability alerts. Works on GitHub, Forgejo, Gitea, GitLab, and other platforms.

## Config template

Read `assets/renovate.json.tmpl`, replace `<timezone>` with your IANA timezone (e.g. `"Europe/Lisbon"`, `"America/New_York"`), and write it to the project root as `renovate.json`.

Read `config.toml` from this directory. If it exists, extract `renovate_git_author` for the default. If not, ask the user for their git author identity and write it to `config.toml` (one-time setup).

Before creating the config, ask the user for the **git author identity** for Renovate commits. Default: `<renovate_git_author from config>`. This email must be verified on the user's GitHub account to avoid "Unverified" commit badges. Replace `<git-author>` with the chosen value (e.g. `"Renovate Bot <bot@example.com>"`).

Key settings:
- `pinDigests: true` — converts `@v4` → `@<sha> # v4` for actions and `image:tag` → `image:tag@sha256:digest` for Docker. Keeps digests pinned and auto-updated.
- `minimumReleaseAge: "30 days"` — prevents brand-new releases from being proposed until a vetting window passes.
- `osvVulnerabilityAlerts: true` — monitors OSV.dev for known CVEs.
- `dependencyDashboard: true` — creates a visibility issue tracking all pending, blocked, and ignored updates.

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
  "fileMatch": [
    "(^|/)(?:compose|docker-compose)[^/]*\\.ya?ml(?:[^/]*\\.example)?$"
  ]
}
```

This captures both `docker-compose.yml` and example files like `docker-compose.yml.example` or `docker-compose.yml.secrets.example`.

### Custom managers

Custom regex managers can handle edge cases not covered by built-in managers. See https://docs.renovatebot.com/configuration-options/#custommanagers.

## Workflow templates

### GitHub (self-hosted)

Read `assets/renovate.yml.github.tmpl` and write it to `.github/workflows/renovate.yml` (no substitutions — pinned defaults for `actions/checkout`). Check for newer versions > 30 days old and update if needed.

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

A classic PAT with `repo` scope also works but grants broader access.

### Forgejo (self-hosted)

Read `assets/renovate.yml.forgejo.tmpl`. Replace `<forgejo-instance>` with your instance hostname (e.g. `codeberg.org`) and `<runner-label>` with your runner tag (e.g. `codeberg-small`). Write the result to `.forgejo/workflows/renovate.yml` (no substitutions for checkout or renovate — pinned defaults). Check for newer versions > 30 days old and update if needed.

Uses the `$FORGEJO_ENV` pattern so the env-var steps can be overridden via `forgejo-runner exec --env` for local testing.

Requires `RENOVATE_TOKEN` as a repository secret.

## README update

After creating the workflow file, add the following section to `README.md` so that contributors can run a Renovate dry-run locally.

**For GitHub projects:**

```bash
act workflow_dispatch -W .github/workflows/renovate.yml \
  --env RENOVATE_DRY_RUN=full \
  --env RENOVATE_REPOSITORIES=owner/repo \
  --env LOG_LEVEL=info \
  -s RENOVATE_TOKEN=<token>
```

Replace `owner/repo` with your repository path and `<token>` with a GitHub classic PAT with `repo` scope.

**For Forgejo projects:**

```bash
forgejo-runner exec \
  -W .forgejo/workflows/renovate.yml \
  --env RENOVATE_DRY_RUN=full \
  --env RENOVATE_REPOSITORIES=owner/repo \
  --env LOG_LEVEL=info \
  -s RENOVATE_TOKEN=<token>
```

Replace `owner/repo` with your repository path and `<token>` with a Forgejo PAT with `repo` scope.
