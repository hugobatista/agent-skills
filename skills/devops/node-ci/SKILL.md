---
name: node-ci
description: >-
  Creates GitHub Actions CI workflows for Node.js/Bun projects:
  typecheck (tsc), tests (bun test), draft releases, and npm publish
  with OIDC trusted publishing.
author: hugobatista
---

## GitHub Actions workflows

When scaffolding a new Node.js/Bun project, or when adding CI to an existing one:

### Setup

0. Ask the user: *"Will this project be hosted on GitHub?"* If no, skip all CI workflow creation entirely.
1. Read the repo name from `package.json` `name` field (the name with hyphens is the repo name).
2. Read `config.toml` from this directory. If it exists, extract `github_owner` and `github_url_base` for defaults. If not, ask the user for these values and write them to `config.toml` (one-time setup).
3. Ask the user for the **GitHub owner** (default: `<github_owner from config>`)
4. Ask the user for the **URL base** for badge links:
   - `go.<owner>.com/gh` (default, e.g. `https://<github_url_base from config>`)
   - `github.com/<owner>` (e.g. `https://github.com/<github_owner from config>`)
   - Custom — user provides their own
5. Ask the user: *"Do you want to set up Renovate for automated dependency updates?"*
   If yes, load the `renovate` skill and follow its instructions for GitHub projects.
   This creates `renovate.json` and optionally `.github/workflows/renovate.yml`.

Substitute `<owner>`, `<repo>`, and `<base>` in all workflow files and badges using the values gathered above.

### Workflow files

- `.github/workflows/lint.yml` — always (typecheck via `tsc --noEmit`)
- `.github/workflows/test.yml` — always (`bun test`)
- `.github/workflows/create_draft_release.yml` — only if the user opts in (ask first)
- `.github/workflows/npm.yml` — only if the user opts in (ask first)
- `renovate.json` — only if the user opted in to Renovate

### Badges

After creating workflow files, add corresponding badges to the top of `README.md` in a single row. The GitHub Tag badge is always added.

| Badge | Condition | Markdown |
|-------|-----------|----------|
| GitHub Tag | always | `[![GitHub Tag](https://img.shields.io/github/v/tag/<owner>/<repo>?logo=github&label=latest)](<base>/<repo>/releases)` |
| Lint | always | `[![Lint](https://img.shields.io/github/actions/workflow/status/<owner>/<repo>/lint.yml?label=Lint)](<base>/<repo>/actions/workflows/lint.yml)` |
| Test | always | `[![Test](https://img.shields.io/github/actions/workflow/status/<owner>/<repo>/test.yml?label=Test)](<base>/<repo>/actions/workflows/test.yml)` |
| npm | if opted in | `[![npm](https://img.shields.io/npm/v/<repo>.svg)](https://www.npmjs.com/package/<repo>)` |
| Renovate | if opted in | `[![Renovate](https://img.shields.io/badge/renovate-enabled-brightgreen?logo=renovatebot)](https://docs.renovatebot.com)` |

### Templates

Each template file lives in `assets/`. Read the `.yml.tmpl` with the Read tool, substitute placeholders, then write the result to `.github/workflows/<name>.yml`:

- **`assets/lint.yml.tmpl`** → `lint.yml` (no substitutions)
- **`assets/test.yml.tmpl`** → `test.yml` (no substitutions)
- **`assets/create_draft_release.yml.tmpl`** → `create_draft_release.yml` (only if user opted in)
- **`assets/npm.yml.tmpl`** → `npm.yml` (only if user opted in)

Create the directory before writing:
```
mkdir -p .github/workflows
```

### Action pinning policy

All workflow actions (`uses:` lines) are pinned to full commit SHAs with a version comment. The pinned version is the latest stable release available for > 30 days, ensuring a vetting window before automated updates are proposed. Renovate's `minimumReleaseAge: "30 days"` respects this window. To update a pin, find the desired tag's commit SHA via the GitHub API (`/repos/{owner}/{repo}/git/ref/tags/{tag}`) or the action's release page. Consider setting up Renovate (step 5) to automate these updates.

Bun is installed via the official `curl -fsSL https://bun.sh/install | bash` script. No third-party action needed. To pin a specific version, pass it as an argument: `bash -s "bun-v1.2.0"`.

### npm Trusted Publishing

The `assets/npm.yml.tmpl` uses npm OIDC trusted publishing (GA since July 2025) instead of long-lived `NPM_TOKEN` secrets. Setup steps:

1. First publish must use a classic access token (create on npmjs.com → Account → Access Tokens → read and write).
2. After the first publish, go to the package on npmjs.com → Settings → Trusted Publisher.
3. Fill in GitHub owner, repo name, and the workflow filename (`npm.yml`).
4. Delete the classic access token from npm and the secret from GitHub.
5. Remove `always-auth` and `registry-url` from the workflow if present.

The workflow strips `_authToken` from `.npmrc` before publish to prevent `setup-node` from interfering with OIDC. It publishes with `--provenance` for supply chain attestation.

If the user prefers classic tokens over OIDC, they can set `NODE_AUTH_TOKEN` as a repository secret and the workflow falls back to token-based auth.

## README update

After creating workflow files, add the corresponding badges to `README.md` at the top, below the `# <title>` line. Badges go in a single row.
