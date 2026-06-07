# mcp-use/mcp-use: clarify inspector skipped-in-production log + start docs

| Field | Value |
|---|---|
| Target | [mcp-use/mcp-use](https://github.com/mcp-use/mcp-use) |
| PR | [#1505](https://github.com/mcp-use/mcp-use/pull/1505) |
| Opened | 2026-05-14 |
| Status | merged 2026-05-14 |

## What

+13 / -1 across three files:

- `libraries/typescript/packages/mcp-use/src/server/inspector/mount.ts`
  (+1 / -1): one-line string change inside the existing `console.log`
  call at line 46.
- `docs/typescript/server/cli-reference.mdx` (+5 / -0): new `<Note>`
  block under `### start - Production Server` pointing readers at
  the `--with-inspector` build flag, between the existing `<Note>`
  and `<Tip>` so it matches the surrounding doc style.
- `libraries/typescript/.changeset/inspector-skipped-log-wording.md`
  (+7 / -0): patch changeset.

The original runtime log read `[INSPECTOR] Skipped in production
(use --with-inspector flag during build)`, which suggested the flag
could be passed to `mcp-use start`. The flag is only registered
under `build`, so `mcp-use start --with-inspector` bounced off an
unknown-flag error.

Initial wording spelled out the rebuild command. After a reviewer
suggestion, the final wording is `The inspector is not available
in production builds. If you want it rebuild with the
--with-inspector flag`.

Grep confirmed the old log string only appeared at the one site;
no snapshot tests asserted it.

## Why

Reported in mcp-use/mcp-use#1495.

## Tests

No new tests. The log string is a runtime message with no test
coverage; the changeset covers semver. PR body confirmed
`pnpm lint:fix` clean.

## Review

- **pkg-pr-new** (NONE) at 2026-05-14T15:07Z preview install link.
- **github-actions** (CONTRIBUTOR) mcp-conformance results at
  15:10Z.
- **khandrew1** (CONTRIBUTOR) CHANGES_REQUESTED at 17:34:34Z on
  commit `97c74cf` with one inline line-comment: `Would reword this
  to something like: "The inspector is not available in production
  builds. If you want it rebuild with the --with-inspector flag"`.
- Replied at 18:02:30Z with `Reworded as suggested. 4f4fa7a7` after
  pushing the suggested wording.
- **khandrew1** (CONTRIBUTOR) empty-body APPROVED at 18:03:46Z on
  commit `4f4fa7a`. Merged into `canary` minutes later. About
  three hours after PR open.

## Lesson

- mcp-use base branch is `canary`, not `main`. The pkg-pr-new bot
  and the mcp-conformance check are both auto-running; both
  passed without intervention.
- The mirror-suggestion reply shape worked here: one inline
  suggestion → one push → one-line reply quoting the SHA. See
  `feedback_pr_review_response_shape.md`.
- Fork PR CI failure on `mcp-use (node-20)` is the secrets gap;
  the OpenAI secret is not available to fork PRs. Maintainers
  merge through. See `reference_mcp_use_fork_pr_ci.md`.
