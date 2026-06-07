# coleam00/Archon: always_run node opt-out for DAG resume caching

| Field | Value |
|---|---|
| Target | [coleam00/Archon](https://github.com/coleam00/Archon) |
| PR | [#1730](https://github.com/coleam00/Archon/pull/1730) |
| Opened | 2026-05-20 |
| Status | merged 2026-05-21 |

## What

+278 / -3 across six files in `packages/workflows/` and the docs
package. New optional `always_run: boolean` field on every DAG node.
When `true`, the node re-executes on resume even when
`priorCompletedNodes` says it completed. Both the resume pre-populate
(filters out always_run node IDs) and the per-node skip-check (gated
by `!node.always_run`) honor the flag.

Two commits:

- `9b3a433` adds the schema field, the executor wiring, the loader
  pass-through, and a `dag.node_always_run_resume_forced` log event.
- `9a8f1c4` adds a symmetric `node_always_run_reset` workflow_event
  so resume forensics can reconstruct decisions from the
  `workflow_events` table; the prior-success skip path already wrote
  a DB event so this restores symmetry.

Default behavior unchanged. Normal cached nodes still skip on resume.
No schema migration, no DB change.

## Why

Reported in coleam00/Archon#1391. Producer nodes whose exit code
does not validate their output (bash that writes a file the
consumer parses, code generators, fetch scripts) cache forever on
success across every resume. A successful-but-garbage producer
stays cached; the only previous escape was renaming the node.

## Tests

193 new lines in `dag-executor.test.ts` plus 25 in `loader.test.ts`.
Covers the resume pre-populate filter and the per-node skip-check
both honoring `always_run: true`, plus loader pass-through of the
new field.

## Review

- **coderabbitai** (NONE) submitted COMMENTED at 2026-05-20T09:51Z
  with two nitpicks on commit `9b3a433`. First, the new test
  asserted behavior but did not assert the structured log emission
  for `dag.node_always_run_resume_forced` per the coding-guidelines
  event-naming convention. Second, a doc clarification on "same run"
  phrasing. Neither was blocking.
- Merged into `dev` at 2026-05-21T11:52Z, about 26 hours after open.
  No human review surfaced; the OWNER merged through coderabbit's
  comments.

## Lesson

- Archon's `dev` branch is the merge target; squash later moves it
  to `main`. See `feedback_base_branch_for_squash_release_repos.md`.
- Symmetric event emission matters on resume-forensics paths. If the
  skip path writes a DB workflow_event, the reset path should write
  one too; structured-log-only on one side and DB-event on the other
  makes the events table unreliable as a source of truth.
- coderabbit nitpicks on Archon are advisory, not blocking. The
  OWNER merged through them in this case. Future Archon PRs can
  ship without pre-addressing CHILL-profile nitpicks if the
  substance is intact.
