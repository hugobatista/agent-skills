---
name: docker-python
description: 'Dockerfile best practices for Python projects. Covers non-root user (ARG UID/GID/APP_USER), COPY --chown, runtime-writable files, and entrypoint diagnostics.'
author: hugobatista
---

## Dockerfile

Read `assets/Dockerfile.tmpl`, replace `appname`, and write it to the project root.

Key patterns:
- uv image pinned to `tag@sha256:digest` — guarantees the exact uv version; update the digest when a new stable release has been available for > 30 days
- `ARG UID=1000 GID=1000 APP_USER=appuser` — configurable at build time via `--build-arg UID=568`
- Privileged RUN (package installs, user creation, app directory) happens **before** `USER`
- `COPY --chown=${UID}:${GID}` sets ownership at build time — no runtime `chown` needed
- `/app` is pre-created and chowned before any copy so the working directory is writable
- Runtime-writable config files (e.g. `config.js`) should be pre-created with `chmod 666` so `docker-compose user:` overrides still work
- `USER ${APP_USER}` is the last step before `ENTRYPOINT`

## Bind mounts

For host bind mounts in docker-compose, users must `chown` the host directory to the container UID:

```bash
sudo chown -R 1000:1000 /host/path
```

To use a custom UID, add `user: "<UID>:<GID>"` in docker-compose and ensure runtime-writable files have `chmod 666` set at build time.

## .dockerignore

```dockerignore
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

## Dependency pinning

- **Docker images**: pin all `FROM` lines to `tag@sha256:digest`. Tags are mutable; digests guarantee the exact image.
  ```dockerfile
  FROM python:3.13.12-slim@sha256:f1927c75e81efd1e091dbd64b6c0ecaa5630b38635a3d1c04034ac636e1f94c8 AS builder
  ```
- **NPM dependencies**: use `npm ci --ignore-scripts` in Dockerfile stages to block postinstall attacks.
  ```dockerfile
  RUN npm ci --prefer-offline --ignore-scripts
  ```
- **Python dependencies**: export from lockfile with `uv export` and install with `pip install --require-hashes` to verify every package hash.
  ```dockerfile
  RUN uv export --no-dev --no-emit-project -o requirements.txt
  RUN pip install --no-cache --require-hashes -r ./requirements.txt
  ```

## GitHub Actions workflow

When setting up Docker for a project, offer to create a GHCR publish workflow. Ask the user: *"Do you want to set up a GitHub Actions workflow to build and push this image to GHCR?"*

If yes:

1. Ask the user: *"Will this project be hosted on GitHub?"* If no, skip this section.
2. Read `config.toml` from this directory. If it exists, extract `github_owner` and `github_url_base` for defaults. If not, ask the user for these values and write them to `config.toml` (one-time setup).
3. Ask the user for the **GitHub owner** (default: `<github_owner from config>`)
4. Ask the user for the **URL base** for badge links:
   - `go.<owner>.com/gh` (default, e.g. `https://<github_url_base from config>`)
   - `github.com/<owner>` (e.g. `https://github.com/<github_owner from config>`)
   - Custom — user provides their own
4. Ask the user for the **repo name** (read from `pyproject.toml` `[project].name` if it's a Python project; otherwise ask directly)

Read `assets/ghcr.yml.tmpl`, then write it to `.github/workflows/ghcr.yml` (no substitutions needed).

Add the following badge to `README.md`:

```
[![GHCR Tag](https://img.shields.io/github/v/tag/<owner>/<repo>?logo=docker&logoColor=white&label=GHCR)](<base>/<repo>/packages)
```

Create the directory before writing:
```
mkdir -p .github/workflows
```
