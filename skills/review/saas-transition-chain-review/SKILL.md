---
name: saas-transition-chain-review
description: >
  Review a chain of pull requests targeting the same feature branch, tracking
  non-blocker findings as forge issues across the chain. Runs the full
  saas-transition-review (4 lenses + tests) on each PR, creates issues for non-blocker
  findings, checks resolution in subsequent PRs, and produces a condensed
  cross-PR report with an end-to-end architecture brief.
  Triggered by: saas transition chain review #A-#B, saas transition chain
  review branch <name>, chain pr hybrid review #A-#B, chain pr hybrid review
  branch <name>.
  Mode: chain
author: hugobatista
---

# SaaS Transition Chain Review

This skill reviews a sequence of pull requests that form a refactoring chain
on the same feature branch. Non-blocker findings are tracked as forge
issues. Each subsequent PR is checked against open issues to see if the
pattern was resolved.

## Setup

- A forge CLI must be installed and authenticated. Load the `forge-detect` skill to set `$FORGE_CLI`.
- The repo must have a remote named `origin`.
- The project root is the current working directory (`pwd`).

## Required Skills

Loaded at the start:

- `forge-detect`
- `saas-transition-strategy`
- `saas-anti-pattern-review`
- `security-reviewer`
- `technical-debt-review`
- `saas-transition-review` (the per-PR review logic is delegated to this skill)

Used at the end:

- `architecture-brief` (generates the end-to-end architecture overview)

## Variables

```
MODE          = chain
FORGE_CLI     = set by forge-detect
STATE_DIR     = docs/code-review
STATE_FILE    = $STATE_DIR/chain-{branch}-state.json
REPORT        = $STATE_DIR/chain-{branch}-review.md
ARCH_BRIEF    = $STATE_DIR/chain-{branch}-arch-brief.md
CHAIN_REPO    = owner/repo  (from git remote url)
```

## Input

Two modes:

| Input | Description |
|---|---|
| `saas transition chain review #A-#B` | Range of PR numbers. If a target branch is not specified, it is inferred from the first PR. |
| `saas transition chain review branch <name>` | Discover all open PRs targeting the given branch, sorted by number ascending (oldest first). |

The legacy phrases `chain pr hybrid review #A-#B` and
`chain pr hybrid review branch <name>` are accepted as aliases.

## Workflow

### 1. Detect forge

Load the `forge-detect` skill. It detects the repository's forge and
sets `$FORGE_CLI`, `$FORGE_TYPE`, and `$FORGE_HOST`. Use the command
templates from `forge-detect` for all forge interactions throughout
this workflow.

Abort if the forge CLI is not installed or not authenticated.

### 2. Discover PR chain

Extract the PR list based on input mode.

**Range mode** (`#A-#B`):

```
for N in A..B:
    $FORGE_CLI pr view <N> (check it targets the expected branch)
```

If the user did not specify a target branch:
- Fetch the first PR and use its `base` branch.
- Show the inferred branch to the user and ask to confirm.

**Branch mode** (`branch <name>`):

```
$FORGE_CLI pr search --base <name> --state open
```

Parse the output, extract PR numbers, sort ascending.

**Validate chain** — show the user:

```
SaaS Transition Chain Review: <branch>

PRs to review (oldest to newest):
  #796  feat(jobs): durable job queue with outbox, retries, and dead-letter handling
  #805  refactor(backend): reorganize into modules/infra/core layers
  ...

Proceed? (Y/n)
```

If the user says no, exit.

### 3. Set variables

```
BRANCH      = <target_branch>
PRS         = [796, 805, ...]  (sorted ascending)
STATE_FILE  = docs/code-review/chain-{BRANCH}-state.json
REPORT      = docs/code-review/chain-{BRANCH}-review.md
```

### 4. Load or initialize state

If `$STATE_FILE` exists and the user wants to resume, load it.
Otherwise create a fresh state.

See `references/state-schema.md` for the state file schema.

### 5. For each PR (oldest to newest)

#### 5a. Fetch and create worktree

Fetch the PR, create the worktree, and set the per-PR variables.

See `references/worktree-and-base.md`.

#### 5b. Determine merge-base for diff

Compute the base commit and generate `$DIFF_FILE`.

See `references/worktree-and-base.md`.

#### 5c. Run SaaS transition review lenses

Use the four lenses from `saas-transition-review` skill (step 6):

- Lens A: SaaS Transition Strategy
- Lens B: SaaS Anti-Pattern
- Lens C: Security
- Lens D: Code Duplication
- Lens E: Technical Debt
- Quick hygiene checks

#### 5d. Run tests and coverage

Same as `saas-transition-review` step 7:

- Backend: `uv run hatch run test -- --cov`
- Frontend: `npm run test:unit` (if node_modules exists)
- Coverage check for new code

#### 5e. Generate per-PR report

Write to `$PR_REPORT` using the template from `saas-transition-review` step 8.

#### 5f. Check for blocking findings

Scan all findings for 🔴 Critical severity.

If any blocking finding exists:

```
STOP. Blocking finding detected in PR #<N>:

  <description>

Continue chain review or stop here?
```

Options:
- **Continue** — add finding to state as `blocking` but proceed to next PR.
- **Stop** — save state, exit with message to user.

#### 5g. Extract non-blocker findings

Collect all findings with severity 🟡 Moderate or ⚪ Low.

Skip findings that are already tracked in `state.issues` (matched by `id`).

#### 5h. Search for existing issues

For each non-blocker finding, search the repo for existing open issues
that match the finding. Use the issue search command from `forge-detect`.

Search by the `saas-transition-chain-review` marker (present in the body of
every issue this skill creates). Filter results locally: match if the
issue title contains the finding's file path and a keyword from the
description.

If a match is found, reuse the existing issue — do not create a duplicate.
Update the issue reference in state if needed.

#### 5i. Batch preview

Show the user all new non-blocker findings for this PR:

```
PR #<N> has <M> new non-blocker findings:

  1. ⚪ infra/jobs/router.py: datetime.now(UTC) instead of ClockProvider
     grep: datetime\.now\(UTC\)

  2. 🟡 worker.py: duplicate pre-flight checks with main.py
     grep: deprecated_env|required_env_vars|required_dirs

Create issues for these? (Y/n)
```

If the user declines, skip creating issues for this PR.

#### 5j. Create issues

For each confirmed finding, create the issue, add labels, and post a PR
comment.

See `references/issue-creation.md`.

#### 5k. Re-check open issues against current worktree

For each issue in state with `status == "open"`:

1. Extract `grep_pattern` and `grep_path` from the issue record.
2. Run grep in the worktree:

```bash
rg -q "<grep_pattern>" <WORKTREE>/<grep_path>
```

3. If the pattern is **not found** (exit code 1):
   - Mark `resolved_in_pr = <PR_ID>`
   - Set `status = "resolved"`
   - Comment on the issue:
     ```
     Resolved in PR #<PR_ID>.
     ```
   - Close the issue using the forge CLI:
     ```
     $FORGE_CLI issue close <ISSUE_ID>
     ```

4. If the pattern is **found** (exit code 0):
   - Keep `status = "open"`
   - No action needed.

#### 5l. Update state file

Save the updated `$STATE_FILE`.

See `references/state-schema.md` for the fields updated per PR.

### 6. Architecture brief (end of chain)

After all PRs are processed, generate an end-to-end architecture overview.

```bash
git diff <BRANCH>..pr-<LAST_PR_ID> --stat
```

Check if the total diff is significant enough (> 100 lines, introduces
new modules/subsystems). If not, skip.

Load the `architecture-brief` skill and pass the context:

```
Load: architecture-brief
Pass: MODE=local, DIFF=/tmp/chain-{BRANCH}-arch.diff
```

Save to `$ARCH_BRIEF`.

### 7. Generate condensed report

Write to `$REPORT` using the template in `references/report-template.md`.

### 8. Show report and ask user

Display the condensed report and the architecture brief to the user.

Ask:

```
SaaS transition chain review complete.

What would you like to do?

1. Save as doc — writes to $REPORT and $ARCH_BRIEF
2. Post as PR comments — post condensed report to each PR
3. None — discard

Keep worktrees for further work? (Y/n)
```

### 9. Cleanup

If the user does not want to keep worktrees:

```bash
for PR_ID in $PRS:
    git worktree remove ../pr-<PR_ID>-review
    git branch -D pr-<PR_ID>

rm -f /tmp/pr-*.diff /tmp/chain-*.md
```

Otherwise, leave worktrees in place and print a message:

```
Worktrees kept at:
  ../pr-<N>-review
  ...

Resume later with the same command.
```
