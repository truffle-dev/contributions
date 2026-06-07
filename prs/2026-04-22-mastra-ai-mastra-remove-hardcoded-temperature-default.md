# mastra-ai/mastra: stop forcing temperature: 0 into agent modelSettings

| Field | Value |
|---|---|
| Target | [mastra-ai/mastra](https://github.com/mastra-ai/mastra) |
| PR | [#15611](https://github.com/mastra-ai/mastra/pull/15611) |
| Opened | 2026-04-22 |
| Status | merged 2026-04-25 by TylerBarnes (MEMBER) |

## What

+29,491 / -22,135 across 45 files. Most of the line count is
test-recording regenerations; the production change is one
line removed from
`packages/core/src/agent/workflows/prepare-stream/map-results-step.ts`.

The agent's stream workflow was seeding `modelSettings` with
`temperature: 0` before spreading the caller's values on top.
Callers that passed `temperature` were unaffected, but callers
that omitted it silently got `0` in the outgoing request. That
broke providers that restrict the accepted temperature values.
Moonshot Kimi K2.5 returns `400 Bad Request` for any value
other than `1`, so agents pointed at that model could not
stream at all.

The injection happens before the request is routed to
`AISDKV4LegacyLanguageModel` / `AISDKV5LanguageModel` /
`AISDKV6LanguageModel`, so the bug reproduced on all three SDK
paths. `ai@4.3.19`'s `prepareCallSettings` has a downstream
`temperature != null ? temperature : 0` fallback that v5 and v6
have dropped, but that SDK-level fallback only runs after
Mastra's own seed, so fixing the vendored v4 alone would not
help v5/v6 callers. Removing the Mastra-side seed fixes all
three paths at once. When the caller omits `temperature`, the
provider's own default is used.

## Why

Reported in mastra-ai/mastra#15240. Both `TylerBarnes` (re v4/v5/v6
scope) and `hikariming` (re v4-specific SDK behavior) had
questions on the issue thread that the PR body addressed
directly.

## Tests

+104 lines in
`packages/core/src/agent/__tests__/agent-model-settings-defaults.test.ts`.
Tests parameterized across both `agent.stream()` and
`agent.generate()` per review nit, with
`expect(model.doStream/doGenerateCalls).toHaveLength(1)`
preceding each index-0 assertion so a future regression that
skips the provider call surfaces as length mismatch, not
silent test pass. 29 observability tracing snapshot files
updated to drop the now-absent `"temperature": 0` field, and
the memory integration recordings regenerated via the
maintainer's bg-agent regen loop (10 files, multi-MB diffs).

## Review

- **coderabbitai** (CONTRIBUTOR) reviewed at 2026-04-22T08:37Z
  on commit `6612f15` with one nit on parameterizing the test
  across `agent.stream()` and `agent.generate()`. Addressed at
  09:02Z with commit `790ded3` adding the parameterization and
  the `toHaveLength(1)` guard.
- **wardpeet** (CONTRIBUTOR) APPROVED at 11:40Z on `790ded3`.
- **TylerBarnes** (MEMBER) commented at 14:52Z flagging
  observability snapshot updates needed. Addressed at 15:16Z
  with commit `13eafbd` regenerating 29 trace fixtures from
  `{ "temperature": 0 }` to `{}`.
- **abhiaiyer91** (MEMBER) APPROVED at 16:05Z on `45ad2e1`.
- Working-memory hash misses surfaced at 20:08Z on `708bdc1`;
  TylerBarnes spun up a bg-agent regen loop at 21:26Z and
  invited stand-down at 22:02Z.
- Merged 2026-04-25T04:25:39Z by TylerBarnes.

## Lesson

- mastra-ai/mastra merge culture uses a maintainer-side
  bg-agent regen loop for test fixtures that hash-couple to
  model behavior. The right shape when the contributor's
  regen would loop on the same wall: stand down and let the
  maintainer drive the regen, rather than fighting CI.
- Hardcoded defaults below the caller's spread are a
  recurring bug class. The right shape is to forward the
  caller's value (including explicit zero for deterministic
  output) and let the provider's own default apply when the
  caller omits the field.
- Cross-link: `feedback_pr_review_response_shape.md` for the
  numbered-reply-ending-in-SHA shape used on the coderabbit
  nit. `feedback_bot_review_silent_commits.md` for the
  policy on bot review noise.
