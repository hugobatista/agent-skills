---
name: python-project
description: >-
  Use when writing, reviewing, or modifying Python code (*.py files).
  Covers Typer CLI, FastAPI, pytest, uv + hatch, 100% coverage.
  When creating a new Python project, also load the python-ci skill for CI setup and the python-devcontainer skill for devcontainer setup.
author: hugobatista
---

## Frameworks & Libraries
- **CLI**: use [Typer](https://typer.tiangolo.com/) — always fetch latest docs
- **Web API**: use [FastAPI](https://fastapi.tiangolo.com/) — always fetch latest docs
- **Dependencies**: `pyproject.toml` + `uv`. Pin the tool version with `required-version` in `[tool.uv]`, exclude fresh deps with `exclude-newer = "30 days"`, and declare the package index explicitly with `[[tool.uv.index]]` (`explicit = true`). For production Docker images, export locked deps with `uv export --no-dev --no-emit-project -o requirements.txt` and install with `pip install --require-hashes`.
- **Build**: hatchling; use hatch when needed
- **Structure**: packages over monolithic files; no module > 500 lines; no name-prefix modules

## Testing & Coverage
- 100% coverage required; no `# pragma: no cover` or `# type: ignore` — fix the
  root cause instead (cast, explicit annotation, restructure). The only exception
  is inherently untypeable code (e.g. SQLAlchemy dynamic attribute access), and
  even then prefer `cast()` or a typed helper.
- Tests for all new features, bug fixes, critical paths, and edge cases (empty inputs, invalid types, large datasets)

## Code Standards
- PEP 8; 4-space indent; 79-char line limit; English only regardless of prompt language
- Type hints on all functions; `typing` module for annotations
- PEP 257 docstrings on public API only — private functions (`_name`) only if logic is non-obvious
- No docstrings on tests — test function names must be descriptive enough to stand alone
- No inline comments for self-explanatory code — names and types are the documentation; comment *why*, not *what*
- Descriptive names; break complex functions into smaller ones
- Handle edge cases explicitly; document design decisions in comments

## Project scaffolding

When creating a new Python project, write a `.gitignore` to the project root:

```gitignore
# Python
__pycache__/
*.pyc
*.pyo

# Virtual environment
.venv/

# Tools
.mypy_cache/
.pytest_cache/
.ruff_cache/

# Coverage
.coverage
coverage/
coverage.xml
htmlcov/

# Build
dist/
build/
*.egg-info/

# Environment
.env
```

## Documentation example
```python

def calculate_area(radius: float) -> float:
    """Return the area of a circle with the given radius."""
    import math
    return math.pi * radius ** 2
```
