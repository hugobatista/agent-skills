---
name: python-ci
description: >-
  ALWAYS load alongside the python skill for any new or existing Python project.
  Creates GitHub Actions CI workflows for linting (ruff) and testing (pytest + coverage).
  Do not skip — CI is a standard part of Python project setup.
author: hugobatista
---

## GitHub Actions workflows

When scaffolding a new Python project, or when adding CI to an existing one:

### Setup

0. Ask the user: *"Will this project be hosted on GitHub?"* If no, skip all CI workflow creation entirely.
1. Read the repo name from `pyproject.toml` `[project].name` (the name with hyphens is the repo name)
2. Read `config.toml` from this directory. If it exists, extract `github_owner` and `github_url_base` for defaults. If not, ask the user for these values and write them to `config.toml` (one-time setup).
3. Ask the user for the **GitHub owner** (default: `<github_owner from config>`)
4. Ask the user for the **URL base** for badge links:
   - `go.<owner>.com/gh` (default, e.g. `https://<github_url_base from config>`)
   - `github.com/<owner>` (e.g. `https://github.com/<github_owner from config>`)
   - Custom — user provides their own
4. Ask the user: *"Do you want to set up Renovate for automated dependency updates?"*
   If yes, load the `renovate` skill and follow its instructions for GitHub projects.
   This creates `renovate.json` and optionally `.github/workflows/renovate.yml`.

Substitute `<owner>`, `<repo>`, `<package>`, and `<base>` in all workflow files and badges using the values gathered above.

### Workflow files

- `.github/workflows/lint.yml` — always
- `.github/workflows/test.yml` — always
- `.github/workflows/pypi.yml` — only if the user opts in (ask first)
- `.github/workflows/create_draft_release.yml` — only if the user opts in (ask first)
- `renovate.json` — only if the user opted in to Renovate

### Badges

After creating workflow files, add corresponding badges to the top of `README.md` in a single row. The GitHub Tag badge is always added.

| Badge | Condition | Markdown |
|-------|-----------|----------|
| GitHub Tag | always | `[![GitHub Tag](https://img.shields.io/github/v/tag/<owner>/<repo>?logo=github&label=latest)](<base>/<repo>/releases)` |
| Lint | always | `[![Lint](https://img.shields.io/github/actions/workflow/status/<owner>/<repo>/lint.yml?label=Lint)](<base>/<repo>/actions/workflows/lint.yml)` |
| Test | always | `[![Test](https://img.shields.io/github/actions/workflow/status/<owner>/<repo>/test.yml?label=Test)](<base>/<repo>/actions/workflows/test.yml)` |
| PyPI | if opted in | `[![PyPI - Version](https://img.shields.io/pypi/v/<package>.svg)](https://pypi.org/project/<package>)` |
| Renovate | if opted in | `[![Renovate](https://img.shields.io/badge/renovate-enabled-brightgreen?logo=renovatebot)](https://docs.renovatebot.com)` |

### Templates

Each template file lives in `assets/`. Read the `.yml.tmpl` with the Read tool, substitute placeholders, then write the result to `.github/workflows/<name>.yml`:

- **`assets/lint.yml.tmpl`** → `lint.yml` (no substitutions)
- **`assets/test.yml.tmpl`** → `test.yml` (no substitutions)
- **`assets/pypi.yml.tmpl`** → `pypi.yml` (only if user opted in)
- **`assets/create_draft_release.yml.tmpl`** → `create_draft_release.yml` (only if user opted in)

Create the directory before writing:
```
mkdir -p .github/workflows
```

### Action pinning policy

All workflow actions (`uses:` lines) are pinned to full commit SHAs with a version comment. The pinned version is the latest stable release available for > 30 days, ensuring a vetting window before automated updates are proposed. Renovate's `minimumReleaseAge: "30 days"` respects this window. To update a pin, find the desired tag's commit SHA via the GitHub API (`/repos/{owner}/{repo}/git/ref/tags/{tag}`) or the action's release page. Consider setting up Renovate (step 4) to automate these updates.

## README update

After creating workflow files, add the following section to `README.md` so contributors can run CI workflows locally with `act`.

    ### Local CI with act

    Requires [act](https://github.com/nektos/act) and a modern runner image:

    ```bash
    # One-time setup
    act -P ubuntu-latest=catthehacker/ubuntu:act-latest
    ```

    ```bash
    # Lint and tests (triggered on push)
    act -P ubuntu-latest=catthehacker/ubuntu:act-latest -W .github/workflows/lint.yml
    act -P ubuntu-latest=catthehacker/ubuntu:act-latest -W .github/workflows/test.yml

    # PyPI publish (workflow_dispatch — needs GITHUB_TOKEN)
    act workflow_dispatch -P ubuntu-latest=catthehacker/ubuntu:act-latest \
      -W .github/workflows/pypi.yml -s GITHUB_TOKEN=<token>

    # Create draft release (workflow_dispatch — needs GITHUB_TOKEN)
    act workflow_dispatch -W .github/workflows/create_draft_release.yml \
      -s GITHUB_TOKEN=<token>
    ```

    Replace `<token>` with a GitHub classic PAT with `repo` scope.

(The block above is 4-space indented. Strip the leading 4 spaces from each line and write the result to README.)
