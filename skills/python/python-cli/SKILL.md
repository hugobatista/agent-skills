---
name: python-cli
description: 'Use when building or modifying Python CLI tools. Covers .dockerignore, PyPI publishing, and readme update conventions for Python CLI projects. For CLI tools in other languages, use the dedicated skill for that language. For the Dockerfile template, load the docker-python skill.'
author: hugobatista
---

## Docker

Load the `docker-python` skill for the Dockerfile template.

Create a `.dockerignore` to exclude unnecessary files. Example:

```
.git/
__pycache__/
*.pyc
.venv/
.env
dist/
*.egg-info/
.mypy_cache/
.pytest_cache/
.coverage
```

Ensure the README H1 title ends with a descriptive emoji icon (e.g. `# appname 🚀`), matching the convention used across all projects.

Update `readme.md` with a Docker section covering:

### Quick Start
```bash
# Option 1: Build from source
docker build -t appname .
docker run -it --rm appname [subcommand] [args]

# Option 2: Run pre-built image (auto-pulls from GHCR)
docker run -it --rm ghcr.io/OWNER/appname:latest [subcommand] [args]
```

## PyPI
Update `readme.md` with `pip install` instructions for the package.
