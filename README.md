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

### python

| Skill | What it does |
|---|---|
| `python-project` | Python conventions. Typer, FastAPI, uv, 100% coverage. |
| `pyproject-toml` | `pyproject.toml` for uv + hatch projects. |
| `python-ci` | GitHub Actions CI: lint, test, coverage, PyPI publish. |
| `python-cli` | Packaging and publishing for Python CLI tools. |
| `python-devcontainer` | Dev Container for uv-based projects. |
| `docker-python` | Dockerfile best practices for Python. Non-root, pinned digests. |

### frontend

| Skill | What it does |
|---|---|
| `web-frontend` | Vue, Tailwind, Vite, dark mode, responsive design. |

### devops

| Skill | What it does |
|---|---|
| `renovate` | Renovate config for GitHub and Forgejo. |
| `github-templates` | Issue and PR templates that ask the right questions. |
| `node-ci` | GitHub Actions CI for Node.js and Bun. |

### review

| Skill | What it does |
|---|---|
| `architecture-brief` | Architecture overview with Mermaid from a PR or diff. |
| `saas-transition-review` | Review changes on self-hosted → SaaS projects. |
| `saas-transition-strategy` | Decision framework for the self-hosted → SaaS model. |
| `saas-anti-pattern-review` | Checklist for SaaS-unfriendly patterns. |
| `security-reviewer` | OWASP Top 10, Zero Trust, and LLM security. |
| `technical-debt-review` | Technical debt lens for code diffs. |
| `saas-transition-chain-review` | Review a chain of PRs, track findings as issues. |
| `design-advisor` | Architectural decisions, design reviews, risk assessment. |

### forge

| Skill | What it does |
|---|---|
| `forge-detect` | Detect GitHub, Forgejo, or GitLab. Provides CLI templates. |
| `forgejo-issue` | Forgejo issue operations via `fj`. |
| `forgejo-pr` | Forgejo pull request operations via `fj`. |

### docs

| Skill | What it does |
|---|---|
| `create-readme` | Generate a README from the project structure. |
| `markdown-style` | GFM formatting, structure, and front matter rules. |
| `mermaid-syntax` | Mermaid syntax that avoids parser errors. |
| `obsidian-vault` | Read and write structured findings to Obsidian vaults. |

### utilities

| Skill | What it does |
|---|---|
| `commit-message` | Propose conventional commits from a diff. |
| `discord-send` | Send Discord webhook messages. |

### coaching

| Skill | What it does |
|---|---|
| `journal-coach` | Socratic coaching grounded in the user's journal. |

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
