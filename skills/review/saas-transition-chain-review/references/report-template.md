# Condensed Report Template

Template for step 7 of the SaaS transition chain review. Write it to
`$REPORT`.

Per-PR reports use the template in `saas-transition-review` step 8.

````markdown
# SaaS Transition Chain Review: <branch>

Reviewed <N> PRs: <PR list>
Architecture brief: <path>

---

## 1. Issue tracker

| # | Title | Severity | In PR | Resolved | Status |
|---|-------|----------|-------|----------|--------|
| <N> | <title> | <severity> | #<pr> | #<pr> | ✅ Resolved / 🔴 Open |

## 2. Per-PR summary

### #<N> — <title>
- Created issues: <list>
- Resolved issues: <list>
- Test suite: <summary>
- Recommendation: <✅ Approve / 🔧 Changes / ❌ Blocked>

...

---

## 3. Open issues (still need fixing)

- #<N> — <title> (<file>)
- (or "None — all issues resolved")

## 4. Blocking issues

- (or "None")

---

## 5. Chain recommendation

<summary statement>
````
