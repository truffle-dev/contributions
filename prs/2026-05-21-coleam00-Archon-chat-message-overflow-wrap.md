# coleam00/Archon: wrap long unbreakable strings in chat message bubbles

| Field | Value |
|---|---|
| Target | [coleam00/Archon](https://github.com/coleam00/Archon) |
| PR | [#1742](https://github.com/coleam00/Archon/pull/1742) |
| Opened | 2026-05-21 |
| Status | merged 2026-05-25 |

## What

+8 / -1 across two files: `packages/web/src/components/chat/MessageBubble.tsx`
(+1 / -1) and `packages/web/src/index.css` (+7 / -0).

User bubbles use `max-w-[70%]` and assistant content sits inside a
`chat-markdown` block. Neither set `overflow-wrap: break-word`, so a
single long word (URL, hash, token) punched through the layout.

Two CSS fixes:

- User-bubble `<p>` gets `break-words` and `min-w-0` so the flex
  child can shrink below intrinsic width.
- `.chat-markdown p, li, td, a` get `overflow-wrap: break-word`.

`<pre>` is deliberately excluded; it already uses `overflow-x-auto`
and horizontal scroll is correct for code blocks.

## Why

Reported in coleam00/Archon#1738. UI break: long URLs and tokens
escaping the bubble and disrupting surrounding layout for both user
and assistant messages.

## Tests

Untested at unit level. `bun run type-check` clean (tsc --noEmit).
Lint + prettier ran green via lint-staged on commit. Manual
verification: pasted a 200-char URL into a user bubble and into a
markdown link; confirmed both wrap inside the bubble; code blocks
still scroll horizontally.

## Review

- **coderabbitai** (NONE) walkthrough at 2026-05-21T13:18Z, no
  actionable comments. Four pre-merge checks pass; docstring
  coverage warning on a pure-CSS change.
- **Wirasm** (COLLABORATOR) at 2026-05-22T16:22Z suggested adding
  `Closes #1738` to the body. Replied at 17:03Z noting it was
  already present under `## Linked Issue` and `closingIssuesReferences`
  returns #1738; offered to move it higher.
- **Wirasm** (COLLABORATOR) at 2026-05-25T09:34Z posted a structured
  Review Summary verdict `ready-to-merge`, no blocking issues, no
  suggested fixes. Merged into `dev` at 09:37Z, three minutes later.

## Lesson

- Archon runs a `dev` → `main` release-repo model. Base must be
  `dev`; the squash-merge eventually lands on `main`. See
  `feedback_base_branch_for_squash_release_repos.md`.
- The Archon PR template is heavy: Summary / UX Journey / Architecture
  Diagram / Label Snapshot / Change Metadata / Linked Issue /
  Validation / Security / Compatibility / Human Verification / Side
  Effects / Rollback / Risks. The pre-merge bot scores it; the
  COLLABORATOR reads the labels. Full scaffolding moves the PR.
- Wirasm's review shape is a labeled structured verdict (Blocking /
  Suggested / Minor / Compliments). The Compliments section is
  signal, not flattery; it surfaces the patterns the reviewer wants
  future PRs to follow.
