# coleam00/Archon: skip markdown code blocks in $nodeId.output validation

| Field | Value |
|---|---|
| Target | [coleam00/Archon](https://github.com/coleam00/Archon) |
| PR | [#1478](https://github.com/coleam00/Archon/pull/1478) |
| Opened | 2026-04-29 |
| Status | merged 2026-04-29 by Wirasm (COLLABORATOR) |

## What

+127 / -5 across four files. `validateDagStructure` in
`packages/workflows/src/loader.ts` scanned `node.when`,
`node.prompt`, and `loop.prompt` for `$nodeId.output` references
using a regex that matched anywhere in the string. Prompt
bodies in builder-style workflows (notably the bundled
`archon-workflow-builder`) embed fenced and inline code as
documentation for the LLM, and those literal `$<other-node>.output`
mentions got false-matched as real cross-node references.

The fix strips triple-backtick fenced blocks and single-backtick
inline code from prompt and `loop.prompt` strings before
scanning. A new `stripMarkdownCode` helper handles both: fenced
blocks come off first (the outer container), then inline code
from remaining prose. `when:` clauses still scan unchanged
because they are JS-like expressions and never carry markdown
code. The diff also wraps one bare `$nodeId.output` mention in
`archon-workflow-builder.yaml`'s Rules section in inline
backticks so it reads as documentation, matching the style of
the surrounding `$nodeId.output` mentions on the same lines.
`bundled-defaults.generated.ts` regenerated.

## Why

`archon-workflow-builder` failed to load on `dev` HEAD with
`dag_structure_invalid: Node 'generate-yaml' references unknown
node '$other-node.output'`. Any user workflow whose `prompt:`
body documented `$<name>.output` syntax inside fenced or inline
code hit the same false positive.

## Tests

+111 / -0 in `packages/workflows/src/loader.test.ts`. New cases
cover: fenced blocks ignored, inline code ignored, real
cross-node `$nodeId.output` references outside code still
validate, multi-fence and mixed-fence + inline cases. The
"still rejects unknown $nodeId.output refs outside code" case
guards against over-stripping prose.

## Review

- **coderabbitai** (NONE) reviewed at 2026-04-29T01:17Z on
  commit `b126ad8` with the standard walkthrough.
- **Wirasm** (COLLABORATOR) review at 2026-04-29T09:46Z with
  verdict `ready-to-merge`. Called out the outer-first
  stripping order (fenced before inline) as correct. Merged
  at 09:47:54Z, 90 seconds later.

## Lesson

- coleam00/Archon's `maintainer-review-pr` workflow runs a
  Verdict / Blocking / Suggested / Compliments structured
  review. `ready-to-merge` with no blocking issues and a
  same-minute merge is the fast-path shape on cleanly-scoped
  fixes.
- When a validator scans free-form text for tokens, it has to
  understand the text's escape conventions or it will false-
  match documentation against real references. Either gate the
  scan to non-markdown contexts (the `when:` case) or strip
  the markdown code envelopes before scanning.
- Outer-first envelope stripping. Fenced blocks contain
  inline-backtick spans inside their bodies; strip the outer
  container first or the inner-strip will eat backticks that
  were already inside a fenced block's payload.
