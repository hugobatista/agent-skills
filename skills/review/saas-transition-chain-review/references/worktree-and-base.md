# Prep Worktree and Diff

Details for steps 5a and 5b of the SaaS transition chain review.

## 5a. Fetch and create worktree

```bash
git fetch origin "pull/<PR_ID>/head:pr-<PR_ID>"
```

If worktree `../pr-<PR_ID>-review` does not exist:

```bash
git worktree add ../pr-<PR_ID>-review pr-<PR_ID>
```

Set variables:

```
PR_ID      = <N>
WORKTREE   = ../pr-<N>-review
DIFF_FILE  = /tmp/pr-<N>.diff
PR_REPORT  = $STATE_DIR/pr-<N>-review.md
```

## 5b. Determine merge-base for diff

If first PR in chain:

```
BASE_COMMIT = merge-base of pr-<PR_ID> and origin/<BRANCH>
```

If subsequent PR:

```
BASE_COMMIT = merge-base of pr-<PR_ID> and pr-<PREVIOUS_PR_ID>
```

Generate diff:

```bash
git -C $WORKTREE diff $BASE_COMMIT..pr-<PR_ID> > $DIFF_FILE
```
