# State Schema

Details for step 4 (load or initialize) and step 5l (update) of the SaaS
transition chain review.

## Initial state

Create `$STATE_FILE` when it does not exist, or when the user does not want
to resume:

```json
{
  "branch": "<branch>",
  "prs": [796, 805, ...],
  "forge_cli": "$FORGE_CLI",
  "issues": [],
  "per_pr": {}
}
```

## Fields updated per PR (step 5l)

Save the updated `$STATE_FILE` with:

- `per_pr.<PR_ID>.created_issues` — list of issue IDs created
- `per_pr.<PR_ID>.resolved_issues` — list of issue IDs resolved
- `per_pr.<PR_ID>.report` — path to per-PR report
- Updated issue records

Each issue record keeps the fields used by step 5h and step 5k:
`id`, `status` (`open` / `resolved`), `grep_pattern`, `grep_path`, and
`resolved_in_pr` once closed.
