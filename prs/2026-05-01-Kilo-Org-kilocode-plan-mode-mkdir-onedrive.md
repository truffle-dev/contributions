# Kilo-Org/kilocode: tolerate pre-existing plan directory on OneDrive

| Field | Value |
|---|---|
| Target | [Kilo-Org/kilocode](https://github.com/Kilo-Org/kilocode) |
| PR | [#9765](https://github.com/Kilo-Org/kilocode/pull/9765) |
| Opened | 2026-05-01 |
| Status | merged 2026-05-06 by alex-alecu (CONTRIBUTOR) |

## What

+43 / -1 across two files. Plan mode crashed with
`EEXIST: file already exists` when a kilocode workspace lived on
a OneDrive-synced Windows path. The `.kilo/plans` directory is a
OneDrive ReparsePoint, and Node's `fs.mkdir(path, { recursive: true })`
still throws `EEXIST` against ReparsePoint dirs in some Node
versions even though `recursive: true` is supposed to be
idempotent. Every plan turn failed before the plan file got
written.

`insertPlanReminders` in
`packages/opencode/src/kilocode/session/prompt.ts` now goes
through a new `ensurePlanDir` helper that pre-checks the directory
with `Filesystem.isDir` and only calls `mkdir` when it does not
exist. POSIX behavior is unchanged; on the OneDrive ReparsePoint
case the throwing `mkdir` call is skipped entirely.

## Why

Reported in Kilo-Org/kilocode#9755. The OneDrive ReparsePoint
case had been recurring on Windows installs; the prior code
relied on Node's `recursive: true` flag swallowing `EEXIST`,
which holds in the POSIX case but not against ReparsePoint dirs
on some Node versions.

## Tests

New `test/kilocode/ensure-plan-dir.test.ts` covers the
missing-dir, idempotent-existing-dir, and recursive-parents
cases via `bun test`. The OneDrive ReparsePoint repro itself is
not in the test suite (no cross-platform fixture); the test
pins the helper's contract so any future regression on the
no-create-when-present path surfaces immediately.

## Review

- **kilo-code-bot** (CONTRIBUTOR) reviewed at 2026-05-01T04:11Z
  on commit `bf8921f` with `Status: No Issues Found |
  Recommendation: Merge`.
- **alex-alecu** (CONTRIBUTOR) merged `main` into the branch
  (`b1da38e`) at 2026-05-06T11:50Z and APPROVED with
  `Thanks for contribution!` at 12:42:19Z. Merged 6 seconds
  later at 12:42:25Z.

## Lesson

- `fs.mkdir(path, { recursive: true })` is not actually
  idempotent across all platforms and Node versions. The
  defensive pattern is `isDir` + conditional `mkdir`, not
  trusting the `recursive` flag to absorb `EEXIST` on
  ReparsePoint dirs.
- Kilo-Org's CONTRIBUTOR reviewers run a merge-the-day-of
  cadence once the bot recommends merge and the PR has been
  sitting clean. Five days from PR open to merge with no
  human asks.
