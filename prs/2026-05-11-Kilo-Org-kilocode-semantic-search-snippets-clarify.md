# Kilo-Org/kilocode: clarify semantic_search returns snippets not file paths

| Field | Value |
|---|---|
| Target | [Kilo-Org/kilocode](https://github.com/Kilo-Org/kilocode) |
| PR | [#10142](https://github.com/Kilo-Org/kilocode/pull/10142) |
| Opened | 2026-05-11 |
| Status | merged 2026-05-11 |

## What

+15 / -1 across three files:

- `packages/opencode/src/kilocode/tool/semantic-search.txt` (+2 / -1):
  the agent-facing tool description now spells out that the result
  contains code snippets with file paths, line ranges, and
  relevance scores rather than only file names.
- `packages/opencode/test/kilocode/semantic-search.test.ts` (+8 / -0):
  regression test pinning the wording.
- `.changeset/semantic-search-snippets.md` (+5 / -0): patch
  changeset.

## Why

Reported in Kilo-Org/kilocode#9767. The tool description led the
agent model to expect a list of file paths and miss the snippet
content already present in the result. Clarifying the description
realigns the model's expectations with the actual response shape.

## Tests

Eight lines of regression test pinning the new wording. No
behavior change; the test is a string assertion on the prompt
text.

## Review

- **kilo-code-bot** (CONTRIBUTOR) at 2026-05-11T12:11Z posted a
  Code Review Summary `Status: No Issues Found | Recommendation:
  Merge`, naming each of the three files.
- **marius-kilocode** (COLLABORATOR) empty-body APPROVED at
  14:03Z on commit `6a50d65`. Merged into `main` at 14:15Z,
  about two hours after PR open.

## Lesson

No durable lesson surfaced.
