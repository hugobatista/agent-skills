---
name: commit-message
description: >-
  Propose a git commit message using conventional commits format (feat:, fix:,
  refactor:, etc.) based on staged/unstaged changes. Always in English. Never
  commit without explicit user approval.
author: hugobatista
---

When the user asks you to suggest or propose a commit message, follow this workflow.

## 1. Detect changes

```bash
git diff --cached
```

If empty, fall back to:

```bash
git diff
```

If both are empty, inform the user and stop.

## 2. Analyze the diff

Determine these four parts:

| Part | Rules |
|------|-------|
| **Type** | One of `feat`, `fix`, `refactor`, `docs`, `style`, `test`, `chore`, `perf`, `ci`, `build`, `revert`. Prefer `feat`/`fix` unless the change is clearly neither. |
| **Scope** (optional) | The module or area affected. Infer from file paths (e.g. `auth`, `expenses`, `ui`, `backend`, `frontend`). |
| **Short description** | Imperative present tense, lowercase, no trailing period, max 72 characters. |
| **Body** (optional) | Blank line followed by bullet points or short paragraphs explaining motivation and context. Wrap at 72 characters. |

## 3. Propose

Present the message in a code block. Ask the user:
- Use as-is?
- Edit it (adjust type / scope / description)?
- Reject and start over?

## 4. Commit only on explicit approval

```bash
git commit -m "type(scope): description" -m "body line 1\nbody line 2"
```

**Never commit automatically.** Only run `git commit` if the user explicitly says to (e.g. "yes, commit it", "go ahead").

## 5. Iterate

If the user asks for adjustments, incorporate the feedback and re-propose.

## Examples

```
feat(auth): add force password change on first login
```

```
fix(expenses): prevent division by zero in split calculation

The split amount formula divided by participant count without
checking for zero, causing a 500 error when removing all
participants.
```

```
refactor(api): extract validation logic into reusable helpers
```

```
docs: update API endpoint list in architecture.md
```
