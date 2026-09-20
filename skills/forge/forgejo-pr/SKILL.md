---
name: forgejo-pr
description: Create, view, edit, comment, assign, close, merge, search, checkout PRs on a Forgejo repo via fj.
author: hugobatista
---

## Setup

`fj auth list` must show an authenticated session.
Pass `--repo owner/repo` or `-R/--remote origin` to target a repo.

## Create

The branch must already exist on the remote.
Find template at `.forgejo/pull_request_template.*` etc.
Fill body under each `## Section` heading in `/tmp/pr-body.md`.

```
fj pr create --repo owner/repo --head <branch> --base master --body-file /tmp/pr-body.md "<title>"
```

Flags: `--autofill`, `--web`, `--agit`. Prefix with `WIP:` for draft.

## View

```
fj pr view <ID>
fj pr view <ID> body
fj pr view <ID> comments
fj pr view <ID> comment <N>
fj pr view <ID> labels
fj pr view <ID> diff
fj pr view <ID> files
fj pr view <ID> commits
```

## Edit

```
fj pr edit <ID> title "<title>"
fj pr edit <ID> body --body-file <file>
fj pr edit <ID> labels --add <label> --rm <label>
fj pr edit <ID> comment <N> --body-file <file>
```

## Comment

```
fj pr comment <PR> "<text>"
fj pr comment <PR> --body-file <file>
```

## Assign

```
fj pr assign <user> [user...]
fj pr unassign <user> [user...]
```

Use `-p/--pr <ID>` if not in the repo directory.

## Close

```
fj pr close <ID>
fj pr close -w "reason" <ID>
```

## Merge

```
fj pr merge <ID> [--method merge|rebase|rebase-merge|squash|manual] [--delete] [--title <t>] [--message <m>]
```

## Search

```
fj pr search [query] [--labels <l>] [--creator <u>] [--assignee <u>] [--state open|closed|all]
```

## Status

```
fj pr status <ID> [--wait]
```

## Browse

```
fj pr browse <ID>
```
