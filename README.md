# Agent Skills 🛠️

Portable [Agent Skills](https://agentskills.io) for coding agents. Each skill
packages procedural knowledge that an agent loads on demand.

Works with OpenCode, Claude Code, Codex, Cursor and any other agent that
supports the open Agent Skills format.

## Install

Install every skill globally for OpenCode:

```bash
npx skills add hugobatista/agent-skills --skill '*' -a opencode -g
```

Install one skill:

```bash
npx skills add hugobatista/agent-skills --skill docker-python -a opencode -g
```

List the available skills without installing:

```bash
npx skills add hugobatista/agent-skills --list
```

The [`skills` CLI](https://github.com/vercel-labs/skills) detects your
installed agents. Drop `-a opencode -g` to install interactively.

## Skills

29 skills in 8 categories. The **When to use** column mirrors each skill's
trigger description, so you can see when the agent picks it up.

### python

| Skill | What it does | When to use |
|---|---|---|
| `python-project` | Python conventions. Typer, FastAPI, pytest, uv + hatch, 100% coverage. | Writing, reviewing, or modifying Python code. Also loads `python-ci` and `python-devcontainer` when scaffolding. |
| `pyproject-toml` | `pyproject.toml` for uv + hatch. Build, dependencies, and tool config for pytest, coverage, mypy, and ruff. | Creating or editing a `pyproject.toml`. |
| `python-ci` | GitHub Actions CI: lint (ruff), test (pytest + coverage), PyPI publish. | Setting up or changing CI for a Python project. Load alongside `python-project`. |
| `python-cli` | Packaging and publishing for Python CLI tools. `.dockerignore`, PyPI, README conventions. | Building or modifying a Python CLI. |
| `python-devcontainer` | Dev Container configuration for uv-based projects. | Adding a devcontainer to a Python project. Load alongside `python-project`. |
| `docker-python` | Dockerfile best practices for Python. Non-root user, pinned digests, runtime-writable files. | Writing or reviewing a Dockerfile for a Python app. |

### frontend

| Skill | What it does | When to use |
|---|---|---|
| `web-frontend` | Vue, Tailwind, Vite, dark/light mode, responsive design, npm hardening. | Building or modifying web frontend code. |

### devops

| Skill | What it does | When to use |
|---|---|---|
| `renovate` | Renovate configuration for GitHub and Forgejo. Pinned actions, 30-day minimum release age. | Adding or changing automated dependency updates. |
| `github-templates` | Issue and PR templates, plus `CONTRIBUTING.md`. | Initialising a GitHub project or adding templates. |
| `node-ci` | GitHub Actions CI for Node.js and Bun. Typecheck, tests, draft releases, npm publish via OIDC. | Setting up CI for a Node or Bun project. |

### review

| Skill | What it does | When to use |
|---|---|---|
| `architecture-brief` | Architecture overview with Mermaid, from a PR or a local diff. | `pr brief`, or explaining what a change built. |
| `saas-transition-review` | Review changes on self-hosted → SaaS projects. Four lenses plus tests. | Reviewing a PR or local diff on such a project. |
| `saas-transition-strategy` | Decision framework for the self-hosted → SaaS model. Invariants and decision tree. | Choosing architecture for such a project. |
| `saas-anti-pattern-review` | Checklist for SaaS-unfriendly patterns: state locality, blocking requests, schedulers, scaling. | Checking code for horizontal-scaling readiness. |
| `security-reviewer` | OWASP Top 10, Zero Trust, and LLM security. | Security-focused review or threat modelling. |
| `technical-debt-review` | Technical debt lens for code diffs: complexity, duplication, hardcoded config, missing error handling. | Reviewing a diff for debt and complexity. |
| `saas-transition-chain-review` | Review a chain of PRs. Tracks findings as forge issues and produces a cross-PR report. | Reviewing a sequence of PRs on one feature branch. |
| `design-advisor` | Architectural decisions, design reviews, risk assessment, implementation planning. | Non-trivial design or planning work. |

### forge

| Skill | What it does | When to use |
|---|---|---|
| `forge-detect` | Detect GitHub, Forgejo, or GitLab. Sets `$FORGE_CLI` and provides command templates. | Any workflow that touches PRs or issues. |
| `forgejo-issue` | Forgejo issue operations via `fj`. | Creating, editing, or closing Forgejo issues. |
| `forgejo-pr` | Forgejo pull request operations via `fj`. | Creating, viewing, or merging Forgejo PRs. |

### docs

| Skill | What it does | When to use |
|---|---|---|
| `create-readme` | Generate a README from the project structure. | Creating or updating a project README. |
| `markdown-style` | GFM formatting, structure, headings, and front matter rules. | Writing or editing Markdown documentation. |
| `ste100` | ASD-STE100 rules for technical writing. Sentence limits, active voice, approved vocabulary. | Writing or editing technical documentation. |
| `mermaid-syntax` | Mermaid syntax that avoids parser errors across renderers. | Writing or reviewing Mermaid diagrams. |
| `obsidian-vault` | Read and write structured findings to Obsidian vaults. | Saving findings to, or reading from, a vault. |

### utilities

| Skill | What it does | When to use |
|---|---|---|
| `commit-message` | Propose conventional commits from a diff. Never commits without approval. | Preparing a commit. |
| `discord-send` | Discord webhook messages: text, embeds, files, threads. | Posting to Discord. |

### coaching

| Skill | What it does | When to use |
|---|---|---|
| `journal-coach` | Socratic coaching grounded in the user's journal and clinical frameworks. | Journal-based coaching sessions. |

## Layout

```
skills/<category>/<skill>/SKILL.md
```

Skills keep supporting files beside `SKILL.md`:

- `assets/` — templates and example config.
- `references/` — detailed documentation loaded on demand.
- `scripts/` — executable helpers.

Live configuration (`config.toml`, `webhooks.json`, `vaults.json`) stays
local and is never published. Each skill ships an `.example` file instead.

## License

MIT. See [LICENSE](LICENSE).
