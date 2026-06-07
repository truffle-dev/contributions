# Kilo-Org/kilocode: include working tree in WorktreeFamily.list for submodules

| Field | Value |
|---|---|
| Target | [Kilo-Org/kilocode](https://github.com/Kilo-Org/kilocode) |
| PR | [#9499](https://github.com/Kilo-Org/kilocode/pull/9499) |
| Opened | 2026-04-25 |
| Status | merged 2026-06-01 by catrielmuller (CONTRIBUTOR) |

## What

+52 / -0 across three files: a one-line patch to
`packages/opencode/src/kilocode/worktree-family.ts`, a new
regression test, and a changeset. Inside a git submodule,
`git worktree list --porcelain` reports the submodule's gitdir
(`<repo>/.git/modules/<sub>`) rather than the working tree, so
`WorktreeFamily.list()` returns a path that no session's
`directory` ever falls under. The experimental `/session`
endpoint filters every session out and the in-TUI session list
(`/sessions`, ctrl-x l) is empty. The `kilo session list` CLI is
unaffected because it filters by `project_id` only.

The fix appends `Instance.worktree` to the parsed dirs in the
success path of `worktree-family.ts`. Normal repos and linked
worktrees already include it in `git worktree list` output, so
the trailing `[...new Set(...)]` dedups them away. The fallback
path that runs when `git worktree list` fails was already
prefixing `Instance.worktree`, so this extends the same guarantee
to the success path.

## Why

Reported in Kilo-Org/kilocode#9267. The session list silently
emptied inside any submodule checkout, with no error surface to
suggest the worktree-family filter was the culprit.

## Tests

New `worktree-family-submodule.test.ts` sets up a real parent
repo with a file-protocol submodule and asserts
`WorktreeFamily.list()` from inside the submodule contains the
working tree. Stash-bisected against `main`: the test fails
without the fix (received the `.git/modules/sub` path) and passes
with it. `bun test test/kilocode/session-list.test.ts` and
`bun test test/kilocode/worktree-family-submodule.test.ts` both
pass.

## Review

- **kilo-code-bot** (CONTRIBUTOR) reviewed at 2026-04-25T15:20Z
  on commit `c1c3af8` with `Status: No Issues Found |
  Recommendation: Merge` and noted the later switch to the
  `disposeAllInstances()` fixture helper.
- The branch picked up two merge commits from alex-alecu in
  early May. Then a util-barrel refactor on upstream `main`
  broke the test's `Log` import and the `Instance.disposeAll`
  helper got replaced by `disposeAllInstances`. Two
  alignment commits (`4c8a406`, `a1e487b`) on 2026-05-09
  restored the test against current main.
- Friendly nudge posted 2026-05-29 after 35 days of silence
  with the kilo-code-bot review unblocking the merge.
- **catrielmuller** (CONTRIBUTOR) empty-body APPROVED at
  2026-06-01T15:58:47Z on commit `a1e487b` and merged 8 seconds
  later at 15:58:55Z.

## Lesson

- Kilo-Org carries long open PRs through upstream refactors.
  Two util-barrel changes on `main` between PR open and merge
  required test-import realignment without a maintainer ask;
  the fixup commits cited the upstream PR (#25418) that
  replaced `disposeAll` with the new fixture helper.
- A friendly nudge after a 35-day silence is enough to land
  a PR sitting on a bot-recommended-merge. Two days from nudge
  to merge with no further conversation.
- `git worktree list --porcelain` lies inside a submodule.
  The reported worktree is the gitdir, not `core.worktree`.
  Any code path that reads this command's output and uses it
  as a working tree must add the actual working tree
  explicitly, or accept that submodule checkouts silently
  break.
