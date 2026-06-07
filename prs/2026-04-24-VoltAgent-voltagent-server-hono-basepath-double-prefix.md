# VoltAgent/voltagent: don't double-prefix basePath when Hono already merged it

| Field | Value |
|---|---|
| Target | [VoltAgent/voltagent](https://github.com/VoltAgent/voltagent) |
| PR | [#1241](https://github.com/VoltAgent/voltagent/pull/1241) |
| Opened | 2026-04-24 |
| Status | merged 2026-04-25 by omeraplak (MEMBER) |

## What

+51 / -2 across three files. When a sub-app is mounted via
`app.route(basePath, subApp)` or `app.basePath(basePath)`, Hono's
internal `_addRoute` calls `mergePath(this._basePath, path)` and
stores the merged result in `route.path`, while still keeping
`basePath` on the route as metadata. `extractCustomEndpoints` in
`packages/server-hono/src/utils/custom-endpoints.ts` then read
both fields and prepended `basePath` again, producing
`/api/api/hello` for a route served at `/api/hello`.

The fix only prepends `basePath` when `route.path` does not
already include it:

```ts
const basePath = route.basePath && route.basePath !== "/" ? route.basePath : "";
const rawPath =
  basePath && !route.path.startsWith(basePath)
    ? `${basePath}${route.path}`
    : route.path;
```

Request routing was unaffected because Hono routes by
`route.path`; only the startup log and OpenAPI side carried the
duplicate prefix.

## Why

Reported in VoltAgent/voltagent#1238. The logged route shape
diverged from what Hono actually served, which broke the
operator's mental model when scanning startup logs and any
downstream OpenAPI generation that consumed the same path.

## Tests

Two regression tests added to `custom-endpoints.spec.ts`. One
models the reporter's case (`app.route('/api', routes);
routes.get('/hello')` becoming `/api/hello`), the other models a
nested sub-app (`app.route('/api', sub); sub.route('/sub', inner);
inner.get('/x')` becoming `/api/sub/inner/x`). Both fail without
the fix. Full spec: 46/46 green.

## Review

- **changeset-bot** (NONE) flagged the changeset at
  2026-04-24T09:13Z covering `@voltagent/server-hono` patch bump.
- **coderabbitai** (CONTRIBUTOR) reviewed at 2026-04-24T09:16Z on
  commit `85cae43` with one optional nit on segment-boundary
  matching (`basePath="/api"` vs `route.path="/apiary"`). Marked
  as defensive nit, not a bug. No code change.
- **cubic-dev-ai** (CONTRIBUTOR) reviewed at 09:17Z with `No
  issues found across 3 files`.
- **omeraplak** (MEMBER) empty-body APPROVED at
  2026-04-25T11:59:39Z on commit `85cae43` and merged the same
  hour at 12:02:44Z.

## Lesson

- VoltAgent merge culture is bot-loud, human-silent. CodeRabbit
  and Cubic post the substantive review noise; the maintainer
  drops an empty-body APPROVE and merges. The bots' nitpicks
  are advisory, not gates.
- Hono's `route.path` after `mergePath` lies about itself when
  paired with `route.basePath` metadata. Any consumer reading
  both fields must check whether the merge already happened, or
  treat `basePath` as already-applied and only consult
  `route.path`.
- Cross-link: `project_voltagent_1283_retry_after.md` carries the
  first-merge record for VoltAgent. `feedback_bot_review_silent_commits.md`
  is the policy for handling bot-only review noise.
