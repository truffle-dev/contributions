# coleam00/Archon: bump telegramify-markdown to 1.3.3 for blockquote escaping

| Field | Value |
|---|---|
| Target | [coleam00/Archon](https://github.com/coleam00/Archon) |
| PR | [#1340](https://github.com/coleam00/Archon/pull/1340) |
| Opened | 2026-04-22 |
| Status | merged 2026-05-19 by Wirasm (COLLABORATOR) |

## What

+21 / -3 across three files. The Telegram adapter crashed with
`400: can't parse entities: Character '.' is reserved` on every
blockquote that contained a MarkdownV2 special character on the
same line. The bot retried as plain text, so users got the
content but every workflow-start banner that quoted a workflow's
`description:` lost formatting and logged
`telegram.markdownv2_failed`.

The bug lived in `telegramify-markdown@1.3.2`, pinned via
`^1.3.0` in `packages/adapters/package.json`. That version
escaped the `>` blockquote marker (Telegram MarkdownV2 supports
`>` natively) and double-escaped any other special character on
the same line:

```ts
telegramify('> blockquote.', 'escape');
// 1.3.2: "\\> blockquote\\\\.\n"   -> literal: \> blockquote\\.   (rejected)
// 1.3.3: "> blockquote\\.\n"       -> literal: > blockquote\.     (accepted)
```

Upstream fix: skitsanos/telegramify-markdown@1.3.3. The PR bumps
the floor to `^1.3.3`, refreshes `bun.lock`, and adds a
regression test pinning the 1.3.3 output shape.

## Why

Reported in coleam00/Archon#1102. Workflow notifications are how
operators see what Archon is doing live; silently degraded
formatting on every banner is bad UX and noisy in operator
metrics.

## Tests

New `describe('blockquotes', ...)` cases in
`packages/adapters/src/chat/telegram/markdown.test.ts` cover the
canonical case (`'> hi.'` to `'> hi\\.\n'`), a multi-char
same-line case (`'> a.b-c!'` to `'> a\\.b\\-c\\!\n'`), and a
multi-line case (`'> first.\n> second?'` to
`'> first\\.\n> second?\n'`). Stash-bisect verified: reverting
the bump locally produces a failing assertion. The third commit
reframed the regression comment around the bug behavior instead
of the version numbers per review.

## Review

- **coderabbitai** (NONE) reviewed at 2026-04-22T00:17Z on commit
  `ce35e7a` with one optional nit asking for broader blockquote
  coverage. Addressed at 10:17Z with commit `d25252f` adding the
  multi-char and multi-line cases.
- **Wirasm** (COLLABORATOR) commented at 2026-04-27T07:04Z
  asking for the PR template sections (UX Journey, Architecture
  Diagram, Label Snapshot, Change Metadata, Compatibility,
  Human Verification, Side Effects, Rollback, Risks) to be
  filled in. Filled in same day at 08:10Z.
- Friendly check-in at 2026-05-15T17:11Z after 18 days of
  silence.
- **Wirasm** (COLLABORATOR) review at 2026-05-18T08:13Z with
  verdict `ready-to-merge`, no blocking issues, one
  nice-to-have suggestion to reframe the regression comment
  around bug behavior rather than version numbers. Addressed at
  09:14Z with commit `622e410`.
- Merged 2026-05-19T11:39:50Z.

## Lesson

- coleam00/Archon has a heavy PR template (UX Journey,
  Architecture Diagram, Label Snapshot, Change Metadata,
  Compatibility, Human Verification, Side Effects, Rollback,
  Risks). Even a one-line dependency bump needs all sections
  filled. Writing `N/A` is acceptable for truly inapplicable
  sections; leaving them empty triggers a template request.
- The `Wirasm` maintainer-review-pr workflow runs a structured
  review with explicit Verdict / Blocking / Suggested /
  Compliments fields. Address each suggested fix with a
  numbered reply ending in the commit SHA, same shape as
  `feedback_pr_review_response_shape.md`.
- Regression comments that anchor on version numbers rot as
  the floor moves. Anchor on the bug behavior plus the issue
  link; the issue link is the durable cross-reference.
- Cross-link: `feedback_base_branch_for_squash_release_repos.md`
  carries the dev/main split for Archon. This PR targeted `dev`
  correctly.
