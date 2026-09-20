---
name: saas-transition-review
description: >
  Review code changes on projects that start self-hosted and evolve into
  SaaS. Catches regressions, anti-patterns, and architecture violations
  before they reach production.  Four specialized lenses — saas transition strategy,
  SaaS anti-pattern, security, and technical debt — plus full test suite execution.
  Works on pull requests (creates isolated worktree) or local git diffs.
  Triggered by: review PR, PR #, saas transition review, hybrid review,
  review pull request, review, review diff.
author: hugobatista
---

# SaaS Transition Review

This skill reviews code changes on projects that follow a self-hosted → SaaS
model. It uses four specialized lenses to catch regressions, anti-patterns,
and safety issues that generic code review would miss. It supports two modes:

- **PR mode**: review a specific pull request (creates an isolated worktree)
- **Local diff mode**: review the current unstaged or staged git diff (uses the working tree directly)

## Setup

- A forge CLI must be installed and authenticated. Load the `forge-detect` skill to set `$FORGE_CLI`.
- The repo must have a remote named `origin`.
- The project root is the current working directory (`pwd`).

## Required Skills

Load these skills at the start (they provide the detailed review checklists):

- `forge-detect`
- `saas-transition-strategy`
- `saas-anti-pattern-review`
- `security-reviewer`
- `technical-debt-review`

## Workflow

### 1. Detect forge and mode

**Detect forge** — load the `forge-detect` skill. It detects the
repository's forge and sets `$FORGE_CLI`, `$FORGE_TYPE`, and
`$FORGE_HOST`. Use the command templates from `forge-detect` for
all forge interactions.

The forge CLI is only required if running in PR mode. If the CLI is not
installed or not authenticated, abort with clear instructions.

**Detect mode** — parse the user's input:

| Input | Mode | Description |
|---|---|---|
| `review PR #<N>`, `PR #<N>`, `review <N>` | **PR** | Review a specific PR |
| `review`, `review diff`, `review --staged` | **Local** | Review current diff |

- Unstaged diff: `review`
- Staged diff: `review --staged`

If both a PR number and no-PR indicators are present, PR mode wins.

### 2. Set mode variables

**PR mode:**

```
MODE        = pr
PR_ID       = <N>
WORKTREE    = ../pr-<N>-review
DIFF_FILE   = /tmp/pr-<N>.diff
REPORT      = docs/code-review/pr-<N>-review.md
```

Derive the PR web URL from `git remote get-url origin` (e.g., `https://github.com/owner/repo/pull/<N>`).

**Local diff mode:**

```
MODE        = local
BRANCH      = git branch --show-current
WORKTREE    = .  (current working tree)
DIFF_FILE   = /tmp/review.diff
REPORT      = docs/code-review/<branch>-review.md
```

No forge CLI needed. No PR web URL.

### 3. Gather input

#### PR mode

Use `$FORGE_CLI` to fetch the PR title, description, changed files, commits, and diff. Save the diff to `$DIFF_FILE`.

#### Local diff mode

Capture the diff:

- Unstaged: `git diff > $DIFF_FILE`
- Staged: `git diff --cached > $DIFF_FILE`

Also get the latest commit message with `git log -1 --format="%s"`.

### 4. Verify if this is a bug fix

Detect if the change is a bug fix by checking, in order:

1. **PR mode**: PR description mentions closing an issue (e.g., "Closes #N") — fetch it with `$FORGE_CLI`
2. **Branch name**: matches `fix/*`, `bugfix/*`, `hotfix/*`, `issue-*`, or contains `fix/`/`bug/`/`hotfix/`
3. **Latest commit**: starts with `fix:`, `fix(`, `bugfix:`, or contains `fix`/`bug`/`close`/`resolve`

If a fix is detected, read the issue context when available. Then examine the source code to confirm:

- **Root cause addressed** — does the change actually resolve the reported error?
- **Edge cases handled** — what happens with unusual input in the fix path?
- **No regression** — can any other code path be adversely affected?
- **Tests cover the fix scenario** — do existing or new tests reproduce the exact problem and confirm the fix? If the change adds code paths without test coverage for the reported scenario, flag it in the report under **Coverage gaps**.

Document findings in the report under a dedicated **Bug-fix verification** section.

### 5. Prepare source material

#### PR mode

Fetch the PR into a local branch and create an isolated worktree:

```bash
git fetch origin "pull/<PR_ID>/head:pr-<PR_ID>"
git worktree add $WORKTREE pr-<PR_ID>
```

The worktree lives at `$WORKTREE` (relative to the project root).

#### Local diff mode

No worktree needed — the current working tree is the source material.

### 6. Review — Five Lenses

For each lens, use the diff (`$DIFF_FILE`) and the source tree (worktree in PR mode, current tree in local mode) as source material.

#### Lens A: SaaS Transition Strategy (from saas-transition-strategy skill)

Load the `saas-transition-strategy` skill. It provides Ground Truths (checking `AGENTS.md` first, falling back to defaults), the five invariants, decision tree, and strategic anti-patterns.

- Check the five **invariants** against the diff
- Walk through the **decision tree** for each changed area
- Flag any **strategic anti-patterns** present in the changes
- Format findings as structured items

#### Lens B: SaaS Anti-Pattern (from saas-anti-pattern-review skill)

Run through the full checklist:

| # | Check |
|---|-------|
| 1 | State locality — in-memory state used for WS, MFA, rate limits, OAuth? |
| 2 | Blocking request patterns — `to_thread`, `BackgroundTasks`, long `future.result()`? |
| 3 | WebSocket dependency — is WS the only delivery mechanism? Fallback present? |
| 4 | Scheduler/job duplication — periodic tasks started in API process? Distributed lock? |
| 5 | Filesystem dependency — local path uploads? S3 abstraction layer? |
| 6 | Database pool pressure — pool sizes, COUNT vs len()? |
| 7 | Request cancellation — frontend timeouts, backend disconnect handling? |
| 8 | Credentials in URLs — tokens in WS URLs, API keys in query params? |
| 9 | Multiple WS consumers — `onmessage` assignment collisions? |

Annotate each finding with severity per the skill's legend: 🔴 Critical, 🟡 Moderate, ⚪ Low.

#### Lens C: Security (from security-reviewer skill)

- Determine code type → risk level → business constraints
- Select 3-5 relevant OWASP Top 10 checks
- Check Zero Trust principles (every request authenticated, all inputs validated)
- Check reliability (timeouts, retries, circuit breakers on external calls)
- If AI/LLM code is present, run OWASP LLM Top 10 checks

#### Lens D: Technical Debt (from technical-debt-review skill)

Load the `technical-debt-review` skill. Run through its full 8-point checklist
against `$DIFF_FILE` and the source tree.

Document in the report under a **Technical Debt** section.

#### Quick hygiene checks

Scan the diff for these common leftovers — grep the `+` lines so you don't get false positives from existing code:

1. **`console.log` / `debugger`** — easy to miss in a one-line debug change. Grep for `console\.(log|warn|error)` and `debugger` in added lines.
2. **Missing i18n keys** — hardcoded user-facing strings that should use the project's i18n system. First, detect the i18n library:
   - Check `package.json` dependencies/peerDependencies for `vue-i18n`, `react-i18next`, `react-intl`, `i18next`, `ngx-translate`, `svelte-i18n`, etc.
   - Check `pyproject.toml` dependencies for `django-modeltranslation`, `babel`, `fluent`, etc.
   Based on the detected library, know the expected translation function/method name (e.g. `t('...')`, `$t('...')`, `_('...')`, `| translate`) and look for hardcoded strings that should use it.
3. **`as any` / `@ts-ignore` / `@ts-expect-error`** — type escapes. Flag unless there's a clear comment saying why it's needed.
4. **Empty catch blocks** — `.catch(() => {})` or `catch { }` with no error handling or logging.
5. **Unused imports** — imported symbols that are never referenced in the diff's scope.

Don't block the PR over these — just mention them if spotted.

### 7. Run full test suite

Detect which test frameworks are available in the source tree (`$WORKTREE`):

**Backend** — find the first `pyproject.toml` in this order:
1. `$WORKTREE/backend/pyproject.toml`  (monorepo layout)
2. `$WORKTREE/pyproject.toml`  (flat layout)

If found, read `[tool.hatch.envs.default.scripts]` to discover available scripts:

```bash
cd <dir containing pyproject.toml>
```

Then run each script that exists in the hatch config:
- `uv run hatch run lint-check` (if script defined)
- `uv run hatch run format-check` (if script defined)
- `uv run hatch run typecheck` (if script defined)
- `uv run hatch run test` (if script defined)

If the hatch section doesn't exist or `uv`/`hatch` is unavailable, fall back to direct tool commands (`ruff check .`, `ruff format --check .`, `mypy`, `pytest`).

**Frontend** — find the first `package.json` in this order:
1. `$WORKTREE/frontend/package.json`
2. `$WORKTREE/frontend/app/package.json`
3. `$WORKTREE/app/package.json`
4. `$WORKTREE/package.json`

If found, read its `"scripts"` and run whichever exist:
```bash
cd <dir containing package.json>
npm run type-check    (if in scripts)
npm run lint:check    (if in scripts)
npm run format:check  (if in scripts)
npm run test:unit     (if in scripts)
```

If `node_modules` doesn't exist in the frontend dir, note it in the report and skip frontend checks.

If no known test framework is found, skip tests and note it in the report.

Track which commands were actually run (and their exit codes) — you'll use this in the report.

#### Coverage check for new code

After tests pass, verify that lines introduced by this diff are covered by tests:

1. Re-run pytest with `--cov-report=term-missing` for the relevant module(s)
2. Cross-reference uncovered lines from the coverage report against lines added in the diff (`$DIFF_FILE` uses `+` prefix)
3. List any new uncovered lines in the report under a dedicated **Coverage gap** subsection in the Test Suite Results section

Read the project's coverage threshold from the first `pyproject.toml` found (check `$WORKTREE/backend/pyproject.toml` first, then `$WORKTREE/pyproject.toml`). Look for `fail_under` under `[tool.coverage.report]` — use that value dynamically instead of hardcoding it. Beyond the global threshold, new code should ideally be covered — flag gaps so the author can address them.

### 8. Generate composite report

Save the report to `$REPORT`:

#### PR mode template

```markdown
# PR Review: #<PR_ID> — <title>

**PR**: <PR web URL>
**Branch**: <branch>
**Base**: <base>
**Files changed**: <count>
**Review date**: <date>
```

#### Local diff mode template

```markdown
# Code Review: <branch>

**Diff**: <files changed summary>
**Review date**: <date>
```

#### Common sections (both modes)

```markdown

---

## 1. SaaS Transition Strategy Review

### Invariants check
- ✅ / ❌ The local deploy works
- ✅ / ❌ No regression for self-hosted
- ✅ / ❌ Nothing requires SaaS-only infra
- ✅ / ❌ Same artifact, both modes
- ✅ / ❌ SaaS features are additive

### Decision tree walkthrough
{findings per changed area}

### Strategic anti-patterns flagged
{list or "None found"}
```

```markdown
## 2. SaaS Anti-Pattern Review

### Findings
{N}. **[severity] Title** — `file:line`
- Current behavior: ...
- Why it fails in SaaS: ...
- Migration path: ...

### Not affected
- {patterns checked and found clear}
```

```markdown
## 3. Security Review

### Risk assessment
{code type, risk level, business constraints}

### Findings
{N}. **[severity] Title** — `file:line`
- Vulnerability: ...
- Fix: ...

### Not affected
- {checks run and clear}
```

```markdown
## 4. Bug-fix verification

{only present if a bug fix was detected — via PR description, branch name, or commit message}

### Original problem
{what the reported issue was, or what the branch/commit suggests}

### Does the fix resolve it?
{analysis}

### Edge cases checked
{detailed analysis}

### Regression risk
{assessment}

```

```markdown
## 5. Technical Debt

### Findings
{N}. **[severity] Title** — `file:line`
- Observation: ...
- Why it is debt: ...
- Suggestion: ...

### Not affected
- {checks run and found nothing}

---

## 7. Test Suite Results

### Backend
{List each command that was run and its result:
- `<command>`: ✅ / ❌ / skipped — {details}
}

### Frontend
{List each command that was run and its result:
- `<command>`: ✅ / ❌ / skipped — {details}
}

### Coverage gaps
- {file:line} — {function/block} — {details}
- {or "None — all new code is covered"}

---

## Summary

### Cross-cutting concerns
{anything that appears in multiple lenses}

### Recommendation
- ✅ Approve
- 🔧 Changes requested (see above)
- ❌ Blocked
```

### 10. Architecture brief (optional)

If the diff introduces a **significant new subsystem, architectural pattern, or cross-cutting concern**, generate an architecture overview:

Load the `architecture-brief` skill and invoke it with the existing context:

```
Load: architecture-brief
Pass: PR_ID=<PR_ID>, WORKTREE=$WORKTREE, DIFF=$DIFF_FILE
```

The `architecture-brief` skill supports both PR mode and local diff mode — it will handle detection autonomously.

Skip this step if:
- The change is a bug fix, small feature (< 100 lines), pure refactor, or docs-only change
- The user explicitly says they don't need a brief

### 11. Show report and ask user

Display the full report to the user, then ask:

> What would you like to do with this review?
> 1. **Post as PR comment** — use the PR comment command from `forge-detect` with `$REPORT` as the body file (PR mode only)
> 2. **Save as doc** — writes to `$REPORT`
> 3. **None** — discard (no comment, no doc file)

If an architecture brief was generated in step 10, note to the user where it was saved.

In **local diff mode**, option 1 is not available — omit it from the prompt.

### 12. Cleanup

**PR mode only:**

```bash
git worktree remove $WORKTREE
git branch -D pr-<PR_ID>
```

**All modes:**

```bash
rm -f $DIFF_FILE
```

Skip worktree cleanup if the user asks to keep the worktree for further work.
