# Issue Creation

Details for step 5j of the SaaS transition chain review. Use this procedure
for each confirmed non-blocker finding.

## Discover repo templates

Repo conventions always win over this skill's defaults. Before creating
issues, discover the repo's templates (`$FORGE_CLI issue templates
--remote origin` for Forgejo, or list `.github/ISSUE_TEMPLATE/` /
`.gitlab/issue_templates/` / `.forgejo/ISSUE_TEMPLATE/`). Read each
template file and parse its YAML front-matter (`title:` prefix,
`labels:`) and its `##` body headings. Store in `$ISSUE_TEMPLATES`.
If the repo has no templates, use the built-in body format below.

## Step 0 — Select template and prefix

By finding type, not severity. Choose the template from `$ISSUE_TEMPLATES`
whose `about`/`name` description best matches the finding type:

| Finding type | Prefix | Label |
|---|---|---|
| Bug / wrong behavior | `[BUG]: ` | `bug` |
| Enhancement | `[FEATURE]: ` | `enhancement` |
| Refactor / code debt | `[REFACTOR]: ` | `refactor` |
| Security | `[SECURITY]: ` | `enhancement` or existing security label |

Use the repo's actual template and its front-matter labels where they
exist. If no template matches the type, reuse the nearest one and
override prefix and label — the front-matter `title:` is a default,
not enforced.

## Step 1 — Create the issue

Build the body dynamically from the chosen template's actual `##`
sections (read in "Discover repo templates"). Do not hardcode section
names — map the skill's fields into whichever headings the template
defines, by semantic fit:

- finding description → a heading like Summary, Description, Issue
- why it matters → a heading like Motivation, Context
- suggested fix → a heading like Proposed Solution, Expected Behavior
- alternative → a heading like Alternatives
- leftovers (Branch, PR, Severity, File, Grep pattern/path, and the
  `_Created automatically by the saas-transition-chain-review skill._` marker
  line used by step 5h) → the catch-all Context / Additional Context
  section, or a trailing list if the template has none

Follow the template's field prompts (e.g. checklists, step lists) as
far as the finding allows; skip sections that do not apply.

If the repo has no templates, fall back to the built-in body format:

```markdown
_This issue was created automatically by the `saas-transition-chain-review` skill._

Non-blocker finding from SaaS transition chain review.

**Branch**: `<BRANCH>`
**PR**: #<PR_ID>
**Severity**: <severity>
**File**: `<file_path>`

<detailed_description>

**Fix**:

<suggested_fix>

**Grep pattern**: `<grep_pattern>`
**Grep path**: `<grep_path>`
```

Title uses the mapped prefix from Step 0. For Forgejo pass the template
key and use a positional title (`fj issue create --remote origin
--template <key> "<title>" --body-file ...`); GitHub/GitLab use
`--title`.

Capture the issue number from the output.

## Step 2 — Add labels

Add labels with the `forge-detect` command template. Prefer the
chosen template's front-matter labels; override with the mapped label
from Step 0 when the prefix type differs. Do not create new labels;
reuse existing ones.

(Forgejo: labels with `--remote origin`, GitLab: skip)

## Step 3 — Post PR comment

Use the PR comment command from `forge-detect`:

```
$FORGE_CLI pr comment <PR_ID> --body-file /tmp/chain-pr-comment-<N>.md
```

Comment content:

```markdown
**Non-blocker finding** — created issue #<ISSUE_ID> to track this across the refactor chain.

> <short_description>

---
_This comment was created automatically by the `saas-transition-chain-review` skill._
```
