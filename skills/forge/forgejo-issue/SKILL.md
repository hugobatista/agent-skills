---
name: forgejo-issue
description: Create, view, edit, comment, assign, close, search issues on a Forgejo repo via fj.
author: hugobatista
---

## Setup

`fj auth list` must show an authenticated session.
Pass `--repo owner/repo` or `-R/--remote origin` to target a repo.

## Create

```
fj issue templates --repo owner/repo
```

Fill body from template sections in `/tmp/issue-body.md`, then:

```
fj issue create --repo owner/repo --template <key> --title "<title>" --body-file /tmp/issue-body.md
```

Use `--no-template` if blank issues are allowed.

## View

```
fj issue view <ID>
fj issue view <ID> body
fj issue view <ID> comments
fj issue view <ID> comment <N>
```

## Edit

```
fj issue edit <ID> title "<title>"
fj issue edit <ID> body --body-file <file>
fj issue edit <ID> labels --add <label> --rm <label>
fj issue edit <ID> comment <N> --body-file <file>
```

## Comment

```
fj issue comment <ID> "<text>"
fj issue comment <ID> --body-file <file>
```

## Assign

```
fj issue assign <ID> <user> [user...]
fj issue unassign <ID> <user> [user...]
```

## Close

```
fj issue close <ID>
fj issue close -w "reason" <ID>
```

## Search

```
fj issue search [query] [--labels <l>] [--creator <u>] [--assignee <u>] [--state open|closed|all]
```

## Browse

```
fj issue browse <ID>
```
