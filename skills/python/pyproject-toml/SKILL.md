---
name: pyproject-toml
description: 'Use when creating or modifying pyproject.toml configuration for Python projects. Covers hatchling build, uv dependency management, and tool configuration for pytest, coverage, mypy, and ruff.'
author: hugobatista
---

## pyproject.toml (mandatory template)

Read `config.toml` from this directory. If it exists, use it for defaults. If not, ask the user for their author info and write it to `config.toml` (one-time setup — future runs are faster).

Ask the user to confirm the author. Do not try to parse the name and email, just show the string as-is and ask for confirmation or correction.

Read `assets/pyproject.toml.tmpl`. Replace `AUTHOR_PLACEHOLDER` with the confirmed author, replace `projectname` with your project name, and write it to the project root.

- Set `required-version` in `[tool.uv]` to the latest stable uv release that's been available for > 30 days (check https://github.com/astral-sh/uv/releases)  - but minimum 0.11.18
- Add `[[tool.uv.index]]` with `explicit = true` to declare the package index explicitly. This documents the trust boundary and prevents accidental cross-resolution if multiple indexes are configured.
  ```toml
  [[tool.uv.index]]
  name = "pypi"
  url = "https://pypi.org/simple"
  explicit = true
  ```

### Required sections — verify each exists after substitution

Read the written file back and confirm every section below is present. The `.tmpl` file is the authoritative source — do not skip, merge, or drop any section.

- `[project]` (name, version, description, authors, requires-python, dependencies)
- `[project.scripts]` (CLI entry point)
- `[build-system]` (hatchling)
- `[tool.uv]` (required-version, exclude-newer)
- `[[tool.uv.index]]` (pypi with explicit = true)
- `[tool.hatch.build.targets.wheel]` (packages = ["src/..."])
- `[dependency-groups]` (lint, test, dev with include-group)
- `[tool.hatch.envs.default]` (type = "virtual", path = ".venv")
- `[tool.hatch.envs.default.scripts]` (lint, format, test, typecheck, validate, check, check-fast)
- `[tool.pytest.ini_options]`
- `[tool.coverage.run]` and `[tool.coverage.report]`
- `[tool.mypy]`
- `[tool.ruff]`, `[tool.ruff.lint]`, `[tool.ruff.format]`
