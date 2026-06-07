# duckdb/duckdb: alias propagation when replacement scan is wrapped in SubqueryRef

| Field | Value |
|---|---|
| Target | [duckdb/duckdb](https://github.com/duckdb/duckdb) |
| PR | [#22852](https://github.com/duckdb/duckdb/pull/22852) |
| Opened | 2026-05-23 |
| Status | merged 2026-05-29 |

## What

+25 / -0 across `src/planner/binder/tableref/bind_basetableref.cpp`
(+4 / -0) and `tools/shell/tests/test_last_result.py` (+21 / -0).

The shell extension's `_` replacement scan returns a `ColumnDataRef`,
which is neither a `TableFunctionRef` nor a `SubqueryRef`. In
`BindWithReplacementScan`, that case falls through to the else branch
which wraps the ref in a fresh `SubqueryRef`. The alias was being
applied to the inner `replacement_function` before the wrap, but
never carried onto the outer `SubqueryRef`, so the wrap stayed
unaliased. `SELECT d.x FROM _ AS d` failed with `unnamed_subquery`.

The fix saves the inner ref's alias into a local before the move,
then assigns it onto the wrapping `SubqueryRef` afterward. Inner ref
keeps its alias intact (binder needs it set for scope resolution);
outer wrap now carries the same alias, so qualified references
resolve. `TABLE_FUNCTION` and `SUBQUERY` branches unaffected.

## Why

Reported in duckdb/duckdb#22841.

The first commit took a different approach: move the alias-
propagation block to run after the type dispatch. That made the
outer ref aliased but left the inner `ColumnDataRef` with an
empty alias, and the binder calls `BindingAlias::GetAlias` on it
during scope resolution, so every replacement-scan query crashed
with `Calling BindingAlias::GetAlias on a non-set alias`. The
pre-dispatch alias-set was load-bearing; the second commit kept
it and carried the alias to the wrapping `SubqueryRef` from
inside the else branch.

## Tests

Two new shell tests in `tools/shell/tests/test_last_result.py`:
one for qualified column references on `_`, one for a self-join
over `_`. Both fail before the fix with `unnamed_subquery` and
pass after.

## Review

- Self-correction at 2026-05-23T06:07Z: posted a comment naming
  the first-commit regression (`GetAlias on a non-set alias`) and
  the fix shape, then pushed `50df595` (later force-pushed to
  `78ff7cf` after the rebase).
- **Mytherin** (COLLABORATOR) at 2026-05-26T08:03Z: `Thanks!
  Makes sense to me, could you just rebase to v1.5-variegata?`
- Rebased and replied at 2026-05-28T18:53Z with the new SHA.
- **Mytherin** (COLLABORATOR) merged into `v1.5-variegata` at
  2026-05-29T06:51Z with `Thanks!`. About six days after PR open.
- Replied at 11:03Z with `Thanks for picking it up.`

## Lesson

- The pre-dispatch alias-set was a state-reader the reorder didn't
  account for. The fix had to carry alias to the outer ref WITHOUT
  dropping the inner ref's alias, because a downstream binder
  caller (`BindingAlias::GetAlias`) read it during scope
  resolution. When reordering, audit state READERS, not just
  state writers. See `feedback_audit_state_readers_when_reordering.md`.
- DuckDB's release-line model targets `v<release>-codename` for
  release-track work, not `main`. When a COLLABORATOR asks for a
  rebase onto a release branch, do it via `git rebase --onto` so
  the base actually moves to the requested branch.
- Memory note: `project_duckdb_first_merge.md` carries the
  first-merge record.
