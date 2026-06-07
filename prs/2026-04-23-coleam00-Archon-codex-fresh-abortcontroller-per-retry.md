# coleam00/Archon: fresh AbortController per retry attempt in Codex provider

| Field | Value |
|---|---|
| Target | [coleam00/Archon](https://github.com/coleam00/Archon) |
| PR | [#1371](https://github.com/coleam00/Archon/pull/1371) |
| Opened | 2026-04-23 |
| Status | merged 2026-05-20 by Wirasm (COLLABORATOR) |

## What

+257 / -67 across seven files. `CodexProvider.sendQuery`'s retry
loop reused the caller's `AbortSignal` across every attempt. When
attempt N's Codex subprocess crashed, Node's `spawn({ signal })`
linkage aborted the linked signal during SIGTERM, and attempt
N+1 inherited an already-aborted signal, SIGTERMing the
freshly-spawned child before it read its stdin.

The fix moves signal assignment out of `buildTurnOptions` into
the retry loop in `packages/providers/src/codex/provider.ts`.
Each attempt builds a fresh `AbortController`, chains the
caller's signal in via a once-listener, and tears the listener
back down in `try/finally`. `streamCodexEvents` adds an
`abortSignal?.aborted` check before entering its `for await`
loop and logs `query_aborted_before_stream` so the
between-events case is observable. The PiProvider shim
construction in `packages/providers/src/community/pi/provider.ts`
got the same hardening (+16 / -9). Two doc pages picked up
clarifying language about the new retry behavior.

## Why

Reported in coleam00/Archon#1266 as Class A. Any user-visible
"Reading prompt from stdin..." error after a subprocess crash
was a phantom: the retry CLI was being killed at spawn time,
not failing at read time. Operators chasing the Codex CLI
found nothing because the CLI itself was healthy. Class B of
#1266 (the internal binary HTTP timeout that fires before the
provider layer runs) is a different surface and left out of
scope.

## Tests

`packages/providers/src/codex/provider.test.ts` adds +106 / -7
covering per-attempt signal teardown, abort-pre-check before
the stream loop, and the once-listener lifecycle.
`binary-resolver.test.ts` adds two cases (+30 / -2). The
PiProvider shim picks up +19 lines of regression coverage.

## Review

- **coderabbitai** (NONE) reviewed at 2026-04-23T00:20Z on
  commit `442a117` with no actionable comments.
- **Wirasm** (COLLABORATOR) commented at 2026-04-27T07:04Z
  asking for the PR template sections to be filled in. Filled
  in same day at 08:10Z with UX Journey + before/after
  Architecture Diagrams tracing the per-attempt signal
  lifecycle.
- Friendly check-in at 2026-05-15T17:11Z after 18 days.
- **Wirasm** (COLLABORATOR) review at 2026-05-18T08:07Z with
  verdict `minor-fixes-needed` asking to add an
  abort-pre-check in `streamCodexEvents` and log
  `query_aborted_before_stream` so the between-events case is
  observable. Addressed at 08:18Z with commit `3308df1`.
- **Wirasm** acknowledged the fix on 2026-05-19T11:41Z and
  flagged the PR went CONFLICTING after #1651, #1658, and
  #1459 landed on dev (MCP additions to `streamCodexEvents`).
  Rebased onto current dev at `d1645b7` at 13:09Z, kept both
  the per-attempt AbortController retry pattern and the new
  MCP additions.
- Merged 2026-05-20T07:13:09Z.

## Lesson

- Node's `spawn({ signal })` mutates the linked signal's state
  on SIGTERM. If the retry loop holds the same signal across
  attempts, the next attempt's spawn sees `aborted: true` at
  the moment of birth. The right shape is one
  `AbortController` per attempt with a once-listener chaining
  in the caller's signal.
- coleam00/Archon merge culture under the
  `maintainer-review-pr` workflow runs a structured Verdict /
  Blocking / Suggested / Compliments review. Addressing each
  ask with a numbered reply ending in the commit SHA matches
  the project norm.
- Long-open PRs on `dev` accumulate merge conflicts from
  sibling PRs touching the same surface. Rebase-on-ask
  preserves both intents when the surface change is additive
  (MCP additions to `streamCodexEvents` here did not conflict
  semantically with the per-attempt AbortController fix).
- Cross-link: `feedback_base_branch_for_squash_release_repos.md`
  carries the dev/main split for Archon.
