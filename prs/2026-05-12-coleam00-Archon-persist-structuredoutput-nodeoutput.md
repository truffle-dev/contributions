# coleam00/Archon: persist structuredOutput on NodeOutput for $node.output.field

| Field | Value |
|---|---|
| Target | [coleam00/Archon](https://github.com/coleam00/Archon) |
| PR | [#1654](https://github.com/coleam00/Archon/pull/1654) |
| Opened | 2026-05-12 |
| Status | merged 2026-05-13 |

## What

+295 / -9 across five files in `packages/workflows/`. When a
provider parses fence-wrapped or preamble-prefixed JSON onto the
result chunk (Pi/Minimax via `tryParseStructuredOutput`), the DAG
executor captured it into a local variable but never persisted it
onto `NodeOutput`. Downstream `$node.output.field` substitution
and `when:` conditions then called `JSON.parse(output)` on the
original prose-prefixed text, threw, and resolved to empty.

Changes:

- `schemas/workflow-run.ts` (+6 / -0): `structuredOutput?: unknown`
  added to the completed / running and failed `NodeOutput`
  branches.
- `dag-executor.ts` (+28 / -0): producer call sites persist the
  parsed structured output onto `NodeOutput`.
- `condition-evaluator.ts` (+25 / -3): `substituteNodeOutputRefs`
  and `resolveOutputRef` prefer `structuredOutput` over
  `JSON.parse(output)`; fall back to the original path when absent.

Bare `$node.output` (unfielded) still reads `output` text.
Cross-resume rehydration from `event_data` is out of scope.
Providers that already overwrite `output` with `JSON.stringify` of
structured output (Claude/Codex with `output_format`) are
unaffected because their `output` is already JSON.

This is a re-target of #1637 onto `dev` per Wirasm. No code change
from #1637; rebased onto upstream/dev. Branch tip `fcb33ed4`
(ED25519 signed).

## Why

Pi/Minimax workflows with a downstream node reading
`$classifier.output.type` (or any other field) silently routed to
the wrong branch or dropped the value. The DAG executor captured
the parsed structured output locally but never persisted it onto
the shared `NodeOutput` shape that downstream consumers read.

## Tests

+236 lines across `dag-executor.test.ts` (+117) and
`condition-evaluator.test.ts` (+119) covering every edge case:
producer persists, consumer prefers `structuredOutput`, fallback
when absent, fence-stripped + preamble cases, bare unfielded
reference still reads `output`.

## Review

- **coderabbitai** (NONE) walkthrough at 2026-05-12T11:03Z, no
  actionable findings. Submitted COMMENTED at 11:10Z on commit
  `fcb33ed`.
- **Wirasm** (COLLABORATOR) at 2026-05-13T09:17Z posted a
  structured Review Summary verdict `ready-to-merge`.
- Merged into `dev` at 11:12Z, about 24 hours after PR open.

## Lesson

- Archon: when a re-target is requested (#1637 → #1654), keep the
  branch tip identical; rebase, do not introduce new substance.
  The re-target body says "no code change" and lists the new
  SHA so the reviewer can verify by diff.
- Persistence asymmetry between producer and consumer is a
  fertile bug class. If the producer captures into a local and
  the consumer reads from the shared shape, the value never
  makes the trip; the test that catches this asserts on the
  consumer side, not on the producer side.
- Archon base `dev`. See
  `feedback_base_branch_for_squash_release_repos.md`.
