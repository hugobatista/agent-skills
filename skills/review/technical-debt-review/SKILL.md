---
name: technical-debt-review
description: >
  Detect technical debt introduced or perpetuated in code diffs.
  Focuses on complexity, propagation of existing debt, hardcoded
  configuration, missing error handling, and inconsistent patterns.
  Designed to run against git diffs with access to the source tree.
  Triggered by: technical debt review, debt review.
author: hugobatista
---

# Technical Debt Review

This skill checks code diffs for technical debt. It covers debt
introduced by the change and debt that the change perpetuates by
touching already-problematic areas without cleanup.

It loads as a lens from a parent review skill (for example,
`saas-transition-review`) or runs standalone against a diff file.

## Input

The skill expects these variables:

- `DIFF_FILE` — path to a git diff file (scan only `+` lines for additions)
- `WORKTREE` — path to the source tree (used for propagation check)

## Checks

### 1. TODOs/FIXMEs/HACKs introduced

Scan added lines for `TODO`, `FIXME`, `HACK`, `XXX`, `BUG`,
`WORKAROUND`, `TEMP`.

```
rg -n "^\+.*(TODO|FIXME|HACK|XXX|BUG|WORKAROUND|TEMP)" "$DIFF_FILE"
```

- **Why**: Each marker defers a decision. The diff is the best time
  to resolve it.
- **Severity**: ⚪ Warning

### 2. Large functions

Check if the diff introduces functions with 40+ non-blank added lines.
Look for `def `, `function `, `fn `, or method definitions.

- **Why**: Large functions concentrate responsibility, resist testing,
  and attract more code.
- **Severity**: 🟡 Moderate

### 3. Deep nesting

Check for 4+ levels of indentation in new code (for example, 16+
spaces) from nested `if`/`for`/`while`/`try`/`match` blocks.

```
rg -n "^\+[ ]{16,}" "$DIFF_FILE"
```

Adjust the indent width to match the project's convention (2 or 4
spaces).

- **Why**: Deep nesting drives cyclomatic complexity. Each level
  doubles the test paths.
- **Severity**: 🟡 Moderate

### 4. Hardcoded configuration

Check added lines for literal URLs, IP addresses, ports, secrets,
API keys, tokens, or timeout values that should be environment
variables.

Exclude test files. Scan for:

- `https?://` with a literal domain (not `localhost`)
- `:<digit>{4,5}` (port numbers)
- `password=`, `secret=`, `api_key=`, `token=`, `apikey=`
- Timeout values larger than 5

- **Why**: Hardcoded config makes the code environment-specific.
  Secrets in code create security incidents.
- **Severity**: 🔴 Critical for secrets — 🟡 Moderate for URLs/ports

### 5. Error handling gaps

Check added code for calls that can fail without handling: `.unwrap()`,
`.expect()`, `assert`, empty `except`, `catch {}`, or results
assigned to `_`.

- **Why**: Unhandled errors crash production or corrupt data
  silently.
- **Severity**: 🔴 Critical

### 6. Propagation of existing debt

For each file modified in the diff, check if that file already
contains `TODO`, `FIXME`, `HACK`, or `XXX` markers. If the diff
touches a file with existing markers but does not address them,
flag it.

Extract modified files from the diff:

```
rg "^--- a/" "$DIFF_FILE" | sed 's|--- a/||'
```

Then check each:

```
rg -n "(TODO|FIXME|HACK|XXX)" "$WORKTREE/$file"
```

Exclude cases where:
- The change is only imports or whitespace
- The existing TODO is unrelated and in a distant part of the file

- **Why**: The broken-window effect — touching debt without cleanup
  normalizes it.
- **Severity**: 🟡 Moderate (🔴 if the change substantially rewrites
  the debt-ridden area)

### 7. Dead or commented-out code

Check added lines for commented-out code blocks (3+ consecutive
commented lines that look like executable code, not documentation).

```
# Python/Ruby: rg -n "^\+.*#[ ]{0,2}(def |if |for |class |import |return |print)" "$DIFF_FILE"
# JS/TS/C:    rg -n "^\+.*//( |)(function |if |for |const |let |var |import )" "$DIFF_FILE"
```

- **Why**: Commented code confuses intent. Readers cannot tell if
  it is deprecated, temporary, or significant.
- **Severity**: ⚪ Warning

### 8. Code duplication

Check for copy-pasted code between the diff's new code and the rest
of the codebase.

1. **Scan new `+` lines in the diff** for functions, helpers, or
   patterns that look like they might already exist elsewhere.
2. **Grep each suspicious function name/body** across the source tree.
   Check which directories exist (`src/`, `app/`, `backend/`,
   `frontend/`, etc.) and search all relevant ones — do not assume a
   specific project layout.
3. **Focus on utility helpers**, not legitimate duplication (for
   example, two different API endpoints following the same SQL pattern
   is fine; two views defining the same `queryParam()` helper is not).
4. **Do not be pedantic** — only flag clear copy-paste, not
   "this could be generalized."

- **Why**: Duplicated code doubles maintenance surface. Each copy
  drifts independently over time.
- **Severity**: 🟡 Moderate

### 9. Magic numbers and strings

Check added lines for numeric literals (excluding 0, 1, -1, empty
string, and 100) and unexplained string literals used as comparison
targets or keys.

Exclude test assertions and fixture data.

- **Why**: Magic values make code hard to understand and change.
  A reader cannot tell if two `86400` values are the same timeout
  or coincidentally equal.
- **Severity**: ⚪ Warning

## Output format

Each finding:

```
{N}. **[severity] Title** — `file:line`
- Observation: ...
- Why it is debt: ...
- Suggestion: ...
```

## Not affected section

List checks that ran but found nothing:

```
### Not affected
- Code duplication — no copy-pasted code found
- Large functions — no new function exceeds 40 lines
- TODOs/FIXMEs — no new markers introduced
- ...
```

## Severity legend

| Severity | Meaning |
|----------|---------|
| 🔴 Critical | Must fix before merge. Risk of production incident. |
| 🟡 Moderate | Should fix in this PR or a follow-up soon after. |
| ⚪ Warning | Note for the record. Can be addressed later. |
