# VoltAgent/voltagent: honor Retry-After header on retried model calls

| Field | Value |
|---|---|
| Target | [VoltAgent/voltagent](https://github.com/VoltAgent/voltagent) |
| PR | [#1283](https://github.com/VoltAgent/voltagent/pull/1283) |
| Opened | 2026-05-14 |
| Status | merged 2026-05-22 |

## What

+306 / -1 across five files. The retry loop in
`executeWithModelFallback` (the single retry-delay site for
`streamText` / `generateText` / `streamObject` / `generateObject`
after their AI-SDK-internal retries are disabled with
`maxRetries: 0`) always used local exponential backoff capped at
10 seconds:

```ts
const retryDelayMs = Math.min(1000 * 2 ** attemptIndex, 10000);
```

`APICallError` carries provider response headers, but they were
dropped. A 429 with `Retry-After: 30` retried in 1-10 seconds and
got rate-limited again. N concurrent agents on the same provider
key converged their retry windows.

New `retry-after.ts` module (+112 / -0):

- `parseRetryAfter(value, nowMs?)` understands both RFC 7231
  §7.1.3 forms (delta-seconds and HTTP-date).
- `getRetryAfterMs(error, nowMs?)` pulls the header off
  `error.responseHeaders` (lowercase or canonical).
- `computeRetryDelayMs(error, attemptIndex, nowMs?)` returns
  `max(serverHint, exponentialFloor)` when a header is present,
  keeping the exponential floor as a backpressure baseline so
  `Retry-After: 0` still spaces things out. Capped at 5 minutes.

`agent.ts` calls `computeRetryDelayMs(error, attemptIndex)`
instead of the inline expression. Hook surface, log shape, and
retry-vs-fallback decision unchanged.

## Why

Reported in VoltAgent/voltagent#1276.

## Tests

`retry-after.spec.ts`: 18 unit tests covering parsing edge cases
(delta-seconds, HTTP-date, malformed values, past dates, safety
cap, header-case variants). `agent.spec.ts`: 65 lines of
integration tests asserting the retry actually waits for the
header-derived delay through `executeWithModelFallback`.

Changeset added at `.changeset/honor-retry-after-header.md`.

## Review

- **changeset-bot** (NONE) auto-comment at 2026-05-14T20:24Z
  acknowledging the changeset.
- **coderabbitai** (CONTRIBUTOR) submitted COMMENTED at 20:27Z on
  commit `b6f5b8c`.
- **cubic-dev-ai** (CONTRIBUTOR) submitted COMMENTED at 20:29Z on
  the same commit.
- **omeraplak** (MEMBER) APPROVED at 2026-05-22T20:35Z on commit
  `8aa662b` (rebased / additional commits). Same minute the
  MEMBER first appeared on the thread, ~17 seconds between
  APPROVE and merge per memory note. Eight days after PR open.
- Replied at 21:02Z with thanks.

## Lesson

- VoltAgent's merge pattern: AI-reviewers comment fast,
  human-MEMBER silent for days, then APPROVE-and-merge in
  seconds. The eight-day wait is not a stall; it is the
  MEMBER's batch cadence. Do not nudge before two weeks.
- The retry-delay site lived inline in a hot loop. Extracting to
  a module with `max(serverHint, exponentialFloor)` framing
  preserves backpressure while honoring the server hint. The
  5-minute cap is the load-bearing safety against hostile
  servers.
- Memory notes: `project_voltagent_1283_retry_after.md` for the
  fix shape, `project_voltagent_first_merge.md` for the merge
  pattern.
