---
name: github-templates
description: 'Create GitHub issue and pull request templates for GitHub-hosted projects. Covers bug reports, feature requests, PR templates, and contributing guidelines. Proactively offers during project initialization.'
author: hugobatista
---

# github-templates

Create GitHub community files (issue templates, PR template, contributing guide) for any GitHub-hosted project.

## Workflow

### Step 1: Check if project will be on GitHub

Ask the user: *"Will this project be hosted on GitHub?"*

- **If yes**: proceed
- **If no**: explain that this skill is for GitHub projects only, and skip

### Step 2: Scan existing templates

Check which of these already exist:

- `.github/ISSUE_TEMPLATE/bug_report.md`
- `.github/ISSUE_TEMPLATE/feature_request.md`
- `.github/ISSUE_TEMPLATE/config.yml`
- `.github/PULL_REQUEST_TEMPLATE.md`
- `CONTRIBUTING.md`

List which ones exist and which are missing.

### Step 3: Gather project metadata

Read the project's configuration files to detect:

| Info | Source |
|------|--------|
| Project name | `pyproject.toml` → `[project].name` or `package.json` → `name` or directory name |
| Language/runtime | Python (`pyproject.toml`), Node (`package.json`), Go (`go.mod`), Rust (`Cargo.toml`) |
| Validation command | Python/uv → `uv run hatch run check` if hatch scripts present, else `uv run pytest && uv run ruff check . && uv run mypy`; Node → `npm test && npm run lint` if scripts present; otherwise fallback to generic |
| Docs URL | Check `README.md` for a docs link; otherwise `pyproject.toml` for `Project-URL` |
| Python version | `pyproject.toml` → `requires-python` |
| Package manager | `uv.lock` → uv, `package-lock.json` → npm, `pnpm-lock.yaml` → pnpm |

For the prerequisites and setup steps in `assets/CONTRIBUTING.md.tmpl`:

**Python projects (pyproject.toml found):**
```yaml
prerequisites: >
  - **Python {{python_version}}**  
  - **uv** (installed globally) — see [docs.astral.sh/uv](https://docs.astral.sh/uv/)
setup_steps: >
  4. Install dependencies:
     ```bash
     uv sync --group dev
     ```
```

**Node projects (package.json found):**
```yaml
prerequisites: >
  - **Node.js {{node_version}}**  
  - **npm** (or **pnpm** / **yarn** depending on lock file)
setup_steps: >
  4. Install dependencies:
     ```bash
     npm install
     ```
```

**Other projects:**
```yaml
prerequisites: "" (omit section)
setup_steps: "" (omit section)
```

### Step 4: Present options

Ask the user which templates to create. Pre-check all missing items by default.

```
The following files are missing:
☑ .github/ISSUE_TEMPLATE/bug_report.md
☑ .github/ISSUE_TEMPLATE/feature_request.md  
☑ .github/ISSUE_TEMPLATE/config.yml
☑ .github/PULL_REQUEST_TEMPLATE.md
☑ CONTRIBUTING.md

Create all of these?
```

If the user declines all, exit.

### Step 5: Ask for customization values

Ask the user for:
- **Docs URL** (optional) — if not found in project config, prompt: "Do you have a documentation URL? (leave blank to skip)"
- **Validation command** — offer the detected value as default, let user override
- **Author/maintainer name** — for CONTRIBUTING.md tone (default: detected from git config `user.name`)

### Step 6: Substitute and write templates

Read each selected `.tmpl` file, substitute placeholders, and write:

| Placeholder | Source |
|---|---|
| `{{project_name}}` | Project name from config (Step 3) |
| `{{docs_url}}` | User-provided or empty (if provided, the conditional `{{#if docs_url}}` blocks render) |
| `{{validation_command}}` | Detected or user-overridden |
| `{{prerequisites}}` | Generated from project type |
| `{{setup_steps}}` | Generated from project type |
| `{{python_version}}` | From `requires-python` in pyproject.toml |

Create directories as needed:
```bash
mkdir -p .github/ISSUE_TEMPLATE
```

Write each template file. For `CONTRIBUTING.md`, use the `assets/CONTRIBUTING.md.tmpl` template.

### Step 7: Confirm

After writing, list the created files for the user.

```
Created:
  .github/ISSUE_TEMPLATE/bug_report.md
  .github/ISSUE_TEMPLATE/feature_request.md
  .github/ISSUE_TEMPLATE/config.yml
  .github/PULL_REQUEST_TEMPLATE.md
  CONTRIBUTING.md
```
