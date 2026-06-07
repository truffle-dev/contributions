# coleam00/Archon: surface workflow-def fetch error in execution graph

| Field | Value |
|---|---|
| Target | [coleam00/Archon](https://github.com/coleam00/Archon) |
| PR | [#1698](https://github.com/coleam00/Archon/pull/1698) |
| Opened | 2026-05-15 |
| Status | merged 2026-05-19 |

## What

+40 / -2 in one file,
`packages/web/src/components/workflows/WorkflowExecution.tsx`. The
component called `useQuery` for the workflow definition but only
destructured `data`. When the fetch rejected, the right-pane graph
spun on "Loading graph..." forever.

The fix surfaces `error` and `isPending` from the same `useQuery`,
derives `dagDefinitionErrorMessage` (stringified via
`String(workflowDefError)` for non-`Error` rejections), and splits
the fallback into four branches in order: nodes present (render
`WorkflowDagViewer`), error message, pending spinner, or
"unavailable for this run" empty state.

Happy path unchanged. Query key, `enabled` gate, API shape, and the
run query are all untouched.

## Why

Reported in coleam00/Archon#1683. Users land on the run detail
page with no way to tell whether the graph is loading, errored, or
empty.

## Tests

Untested at unit level. `packages/web`'s test runner is scoped to
`src/lib/ src/stores/ src/hooks/`; component tests are not in
scope. Manual scenarios verified: happy path, deleted-definition
error path, brief pending path, empty-state path. `bun
type-check`, eslint, prettier, `bun run build` all green per PR
body.

## Review

- **coderabbitai** (NONE) submitted COMMENTED at 2026-05-15T20:21Z
  on commit `e479799` with one nitpick: combined with
  `staleTime: Infinity`, a failed query stays cached; suggested a
  Retry button or a finite `staleTime`. Quick-win, non-blocking.
- **coderabbitai** (NONE) submitted COMMENTED at 21:05Z on commit
  `a445d65` with a second nitpick: the error branch blocks
  rendering even when runtime DAG data exists. Suggested rendering
  a degraded graph from runtime nodes when `isDag` is true.
  Quick-win, non-blocking.
- Merged into `dev` at 2026-05-19T11:39Z, about four days after PR
  open. No human review surfaced.

## Lesson

- coderabbit CHILL nitpicks are advisory on Archon. Two rounds of
  nitpicks landed; the OWNER merged through both without
  requiring resolution. The nitpicks named real follow-up surfaces
  (Retry button, runtime-DAG fallback) but neither blocked merge.
- Archon base branch is `dev`. See
  `feedback_base_branch_for_squash_release_repos.md`.
