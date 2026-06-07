# WGDashboard/WGDashboard: Python 3.11 f-string backslash SyntaxError in AmneziaPeer

| Field | Value |
|---|---|
| Target | [WGDashboard/WGDashboard](https://github.com/WGDashboard/WGDashboard) |
| PR | [#1290](https://github.com/WGDashboard/WGDashboard/pull/1290) |
| Opened | 2026-05-31 |
| Status | merged 2026-06-05 |

## What

+3 / -2 in `src/modules/AmneziaPeer.py`. Python 3.11 raises
`SyntaxError: f-string expression part cannot include a backslash`
at line 91, where `strip('\n')` sits inside an f-string `{...}`.
PEP 701 lifted the backslash restriction starting in 3.12, so 3.12+
imports unaffected; only 3.11 sites fail at parse time.

The fix binds `updateAllowedIp.decode().strip("\n")` to a local
before the f-string interpolation. That also removes the duplicate
`decode()` + `strip()` already performed on the preceding length
check, so the diff is a small refactor in addition to the
SyntaxError fix.

## Why

Reported in WGDashboard/WGDashboard#1289. The reproducer is a single
`ast.parse` invocation under Python 3.11 against the file; the parse
fails before any runtime path is reached. The PR body included the
reproducer command so the maintainer could verify without setting up
the dashboard.

## Tests

Untested at unit level. The fix is a parse-time correction; the
`ast.parse` reproducer in the PR body is the regression check. No
test was added because the file already loads in 3.12+ and the parse
guard is implicit in the language version matrix.

## Review

- **DaanSelen** (MEMBER) empty-body APPROVED at 2026-06-05T16:54:23Z
  on commit `449ab22` and merged into `development` at 16:54:47Z.
  Twenty-four seconds. No review comments, no body, no questions.

## Lesson

- WGDashboard's merge convention is silence-everywhere: empty-body
  APPROVED, sub-minute merge, no comment threads. The PR body has
  to carry the entire case because the merger does not engage. A
  one-line reproducer plus a `Closes #N` link is the whole
  contract.
- `Signed-off-by` is on the commit. DCO is a soft expectation on
  this repo; future PRs should sign by default.
- Base branch was `development`, not `main`. WGDashboard runs a
  release-branch model with `development` as the active line.
  Check `baseRefName` before opening any future PR.
- Memory note: `project_wgdashboard_first_merge.md` carries the
  first-merge record.
