# mcp-use/mcp-use: keep mcp-use build non-fatal under bun runtime

| Field | Value |
|---|---|
| Target | [mcp-use/mcp-use](https://github.com/mcp-use/mcp-use) |
| PR | [#1382](https://github.com/mcp-use/mcp-use/pull/1382) |
| Opened | 2026-04-22 |
| Status | merged 2026-04-24 by khandrew1 (CONTRIBUTOR) |

## What

+82 / -28 across two files. `mcp-use build` crashed during the
tool-registry type-generation step when run under bun (e.g.
inside `oven/bun:alpine`). The step used `tsx/esm/api`'s
`tsImport`, which relies on Node.js custom loader hooks that
bun does not implement, and the exception killed the whole
build with `Cannot find module 'tsx://...'`.

The fix in `libraries/typescript/packages/cli/src/index.ts`:

- `generateToolRegistryTypesForServer` detects the bun runtime
  up front and skips the `tsx/esm/api` step with a clear
  warning instead of throwing.
- The `build` command wraps the call in try/catch so any other
  import-time error in the user's server file surfaces as a
  non-blocking warning instead of `process.exit(1)`.
- `tsc --noEmit` now runs via `process.execPath` rather than
  hardcoded `node`, since `oven/bun:alpine` ships no `node`
  binary. `--max-old-space-size=4096` is dropped under bun
  (unsupported flag).

Node.js behaviour is unchanged.

## Why

Reported in mcp-use/mcp-use#1327. Type generation is a dev
convenience (regenerates `.mcp-use/tool-registry.d.ts`) and
should not be able to fail a Docker production build.

## Tests

No new unit tests. Verified manually via `pnpm --filter
@mcp-use/cli build` (CJS + ESM build clean) and `pnpm lint /
pnpm format:check` clean. The maintainer added a tri-state
return value (`"ok" | "failed" | "skipped"`) post-merge to
fix the misleading "had errors" warning the boolean
return value produced after a clean skip.

## Review

- **pkg-pr-new** (NONE) posted snapshot install instructions
  at 2026-04-24T15:59Z.
- **khandrew1** (CONTRIBUTOR) APPROVED and merged at
  2026-04-24T15:58:50Z on commit `3889ab3`.
- Thank-you at 2026-04-25T08:03Z acknowledging the tri-state
  addition: the boolean shipped quietly produced a misleading
  "had errors" warning after the "Skipping under bun runtime"
  notice because the same return value covered both real
  type-gen failures and clean skips.

## Lesson

- bun runtime detection in Node.js CLIs follows the
  `globalThis.Bun || process.versions.bun` pattern. Trip
  wires include `tsx` loaders that rely on Node custom hooks
  and hardcoded `node` in `spawn` paths (the `process.execPath`
  swap is the durable fix).
- A boolean return value collapses two distinct outcomes
  (failure and intentional skip) into the same path. When
  the caller has to log differently per outcome, the right
  shape is tri-state. The maintainer's post-merge fix is the
  pattern to reach for first next time.
- Cross-link: `reference_bun_runtime_detection.md`.
