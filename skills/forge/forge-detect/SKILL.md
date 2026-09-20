---
name: forge-detect
description: >
  Detect the forge type from the current repo's git remote. Sets
  $FORGE_CLI, $FORGE_TYPE, and $FORGE_HOST. Provides per-forge
  command templates for PR and issue operations.
author: hugobatista
---

# Forge Detect

Load this skill at the start of any workflow that needs to interact
with pull requests or issues on GitHub, Forgejo, or GitLab.

## Setup

- The repo must have a remote named `origin`
- The forge CLI must be installed and authenticated
  (`gh` for GitHub, `fj` for Forgejo, `glab` for GitLab)

## Detection

Run `git remote get-url origin` and parse the host:

| Remote host | `$FORGE_TYPE` | `$FORGE_CLI` |
|---|---|---|
| `github.com` | `github` | `gh` |
| `codeberg.org` | `forgejo` | `fj` |
| `gitlab.com` or `*.gitlab.*` | `gitlab` | `glab` |
| Other | unsupported — abort | — |

Also set `$FORGE_HOST` to the detected hostname.

### Load forge-specific skills

If `$FORGE_TYPE == "forgejo"`, load `forgejo-pr` and `forgejo-issue`
now. They are the authoritative reference for all `fj` subcommands,
flags, and patterns.

If `$FORGE_TYPE == "github"` or `"gitlab"`, no extra skills needed.
Refer to `gh` or `glab` CLI documentation for detailed usage.

## Command templates

All operations below use `$ID` for the PR/issue number and `<path>`
for file arguments.

### PR view (title, body, metadata)

| `$FORGE_TYPE` | Command |
|---|---|
| `github` | `gh pr view $ID --json title,body,changedFiles,headRefName,baseRefName,additions,deletions` |
| `forgejo` | `fj pr view $ID` |
| `gitlab` | `glab mr view $ID` |

### PR diff

| `$FORGE_TYPE` | Command |
|---|---|
| `github` | `gh pr diff $ID` |
| `forgejo` | `fj pr view $ID diff` |
| `gitlab` | `glab mr diff $ID` |

### PR files

| `$FORGE_TYPE` | Command |
|---|---|
| `github` | `gh pr view $ID --json files --jq '.files[].path'` |
| `forgejo` | `fj pr view $ID files` |
| `gitlab` | `glab mr view $ID --json files --jq '.files[].path'` |

### PR comment

| `$FORGE_TYPE` | Command |
|---|---|
| `github` | `gh pr comment $ID --body-file <path>` |
| `forgejo` | `fj pr comment $ID --body-file <path>` |
| `gitlab` | `glab mr note $ID --message "$(cat <path>)"` |

### Issue create

| `$FORGE_TYPE` | Command |
|---|---|
| `github` | `gh issue create --title "<title>" --body-file <path>` |
| `forgejo` | `fj issue create --remote origin "<title>" --body-file <path>` |
| `gitlab` | `glab issue create --title "<title>" --description "$(cat <path>)"` |

For Forgejo, the title is a positional argument (not a `--title` flag).
If the repo requires a template, add `--template <key>` or `--no-template`
(run `fj issue templates --remote origin` first to list templates).

### Issue template conventions

Before creating an issue, read the repo's templates and use their
conventions. For each template, parse the YAML front-matter (`title:`
prefix, `labels:`, `about:`/`name:`) and the `##` body headings. Pick
the template whose description best matches the issue type, use its
`title:` prefix and labels, and fill its `##` sections. For types
without a matching template, reuse the nearest one and override the
prefix/label.

### Issue edit labels

| `$FORGE_TYPE` | Command |
|---|---|
| `github` | `gh issue edit $ID --add-label <label>` |
| `forgejo` | `fj issue edit $ID labels --add <label> --remote origin` |
| `gitlab` | Not supported — skip label management by `glab` |

### Issue search by labels

| `$FORGE_TYPE` | Command |
|---|---|
| `github` | `gh issue list --state open --label <label> --json number,title` |
| `forgejo` | `fj issue search --remote origin --labels <label> --state open` |
| `gitlab` | `glab issue list --state opened --label <label>` |

### Issue close

| `$FORGE_TYPE` | Command |
|---|---|
| `github` | `gh issue close $ID` |
| `forgejo` | `fj issue close $ID --remote origin` |
| `gitlab` | `glab issue close $ID` |

