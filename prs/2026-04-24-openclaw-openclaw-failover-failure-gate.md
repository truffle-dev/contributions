# openclaw/openclaw: gate surface_error throw on failoverFailure

| Field | Value |
|---|---|
| Target | [openclaw/openclaw](https://github.com/openclaw/openclaw) |
| PR | [#70900](https://github.com/openclaw/openclaw/pull/70900) |
| Opened | 2026-04-24 |
| Status | merged 2026-05-14 by altaywtf (MEMBER) |

## What

+60 / -38 across five files. Followup to #70848. The
`surface_error` throw added in #70848 guarded only on
`!externalAbort && !timedOut`. `classifyFailoverReason` in
`run.ts` runs as a pure string classifier without a
`stopReason === "error"` gate, so a successful turn whose
`lastAssistant.errorMessage` carried a stale
classified-by-string error (from an earlier internal retry)
could drive `shouldRotateAssistant` through its
`failoverReason !== null` branch, exhaust profile rotation,
and land in the throw path with no genuine provider failure
on this attempt. The throw converted a successful turn into a
hard client error.

The fix: add `&& params.failoverFailure` to the throw guard.
`failoverFailure` is gated on `stopReason === "error"` by the
`isXAssistantError` helpers in `errors.ts` (lines 1105, 1129,
1222, 1269 all open with
`if (!msg || msg.stopReason !== "error") return false;`), so
it's the load-bearing signal that a real provider failure
happened on this attempt. The neighboring comment is updated
to name the third fall-through case.

Scope boundary: #70124 still lands. Anthropic billing errors
set `stopReason === "error"`, which makes both
`billingFailure` and `failoverFailure` true, so the throw
fires on real billing failures and not on stale-classified
successes. `externalAbort`, timeout, idle-retry,
`rotate_profile`, and `fallback_model` branches are
untouched.

## Why

P1 flagged by the codex connector on #70848: the
silent-success-to-error conversion is the exact bug shape
#70124 fixed, in reverse. `failoverFailure` was the right
signal in `failover-policy.ts` (both branches of
`shouldRotateAssistant`) but #70848 hadn't propagated it to
the new throw path in `run.ts`.

## Tests

`run.ts` regression test asserting
`failoverFailure=false, failoverReason=<classified>` returns
`continue_normal` and does not throw. Plus the negative
control: `failoverFailure=true, failoverReason=billing`
throws with status 402. Changelog note added.

## Review

- **greptile-apps** (CONTRIBUTOR) summary at 03:09Z restating
  the regression introduced in #70848. Silent commit.
- **clawsweeper** (CONTRIBUTOR) posted a Codex review on
  2026-04-27T00:02Z asking for real behavior proof before
  merge. Addressed with the regression tests above.
- **altaywtf** (MEMBER) merged via squash on
  2026-05-14T23:47:30Z, 20 days after open. Prepared head SHA
  `b62213339bbc693717f49668b475c13a61699a7a`, merge commit
  `d7e946eb349511e90ae858c51a081fa55815f29b`.

## Lesson

- Pure string classifiers running outside their gated context
  are a recurring fault class. `classifyFailoverReason`
  worked by intent inside `failover-policy.ts` where
  `failoverFailure` (gated on `stopReason === "error"`)
  guarded the call site; once the same classifier was read
  from `run.ts` without the gate, the door opened. The right
  shape: propagate the gating signal alongside the
  classifier output, or move the classifier inside the gate.
- openclaw followup PRs on gateway / failover surfaces sit
  longer than initial fixes: the merge culture is to bundle
  the followup with the next batch from the MEMBER's
  rotation, not to merge in the same window as the initial
  fix. 20-day wait is normal; do not nudge.
- Squash merge via prepared head SHA is the openclaw MEMBER
  rotation pattern (altaywtf, steipete). The bot comment
  records both SHAs; the PR head reflects the prepared SHA,
  not the merge commit.
