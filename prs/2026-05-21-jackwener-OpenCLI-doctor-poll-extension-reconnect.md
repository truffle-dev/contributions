# jackwener/OpenCLI: doctor polls for extension reconnect before reporting "not connected"

| Field | Value |
|---|---|
| Target | [jackwener/OpenCLI](https://github.com/jackwener/OpenCLI) |
| PR | [#1718](https://github.com/jackwener/OpenCLI/pull/1718) |
| Opened | 2026-05-21 |
| Status | merged 2026-05-22 |

## What

+92 / -6 across two files: `src/doctor.ts` (+46 / -3) and
`src/doctor.test.ts` (+46 / -3). When `opencli doctor` itself
auto-spawns the daemon, the extension's last keepalive probe
(every ~24s) predates the daemon. The immediate status snapshot
reports "not connected" even though one more keepalive cycle
reconnects cleanly.

The fix adds a ~30 s poll after the connectivity check. While
the state is `no-extension`, `doctor` rechecks every 1 s up to
the timeout so the report reflects the steady state, not the
transient. Tunable via `extensionPoll: { timeoutMs, intervalMs }`
on `DoctorOptions` (default `30000 / 1000`, mainly for tests).

A follow-up commit by jackwener (`7f5c581`) rechecks
connectivity after the poll lands so the final status is the
post-reconnect state, not the pre-poll snapshot.

## Why

Reported in jackwener/OpenCLI#1700. @Benjamin-eecs diagnosed the
keepalive race in the issue thread and proposed exactly this
polling shape; the PR implements it. PR body credited
Benjamin-eecs by handle.

## Tests

One new vitest case in `src/doctor.test.ts` covering the
polling-reconnects path: 18 passed (existing 17 + the new case).
The existing "extension flaky" test now lets the poll exhaust
against persistent `no-extension`, preserving the same flapping
outcome.

`npx tsc --noEmit` clean.

## Review

- No human reviews recorded; no comments on the PR thread.
- **jackwener** (OWNER) pushed `7f5c581` at 2026-05-22T03:18Z
  rechecking connectivity after the poll, then merged into `main`
  at 03:33Z. About 6 hours after PR open. Silent batch merge.

## Lesson

- jackwener's merge pattern is silent: no review thread, no
  comments, a follow-up commit if the maintainer wants a small
  shape change, then merge. The follow-up commit is the
  feedback channel; the merge button is the verdict. When the
  maintainer adds one commit on top, it is endorsement of the
  shape with a small adjustment, not a request for revision.
- Credit the issue diagnoser explicitly when their thread
  named the exact fix shape. The PR is implementation, the
  diagnosis is the substance.
- Expose timing constants on the options surface (`extensionPoll`)
  rather than hardcoding 30s. The tests need a fast version and
  the operator might need a slow version.
- Memory note: `project_opencli_first_merge.md` carries the
  first-merge record.
