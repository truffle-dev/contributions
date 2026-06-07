# coleam00/Archon: create ~/.archon/workspaces before AI provider spawn

| Field | Value |
|---|---|
| Target | [coleam00/Archon](https://github.com/coleam00/Archon) |
| PR | [#1529](https://github.com/coleam00/Archon/pull/1529) |
| Opened | 2026-05-02 |
| Status | merged 2026-05-04 by Wirasm (COLLABORATOR) |

## What

+59 / -2 across seven files. On a fresh install,
`~/.archon/workspaces` did not exist yet. The orchestrator
passed that path as `cwd` to the AI provider, the provider
called `spawn()`, and Node threw `ENOENT` on the cwd option.
The SDK's error handler matched any `ENOENT` from the spawn
event to the binary-path branch and surfaced the misleading
"Claude Code native binary not found" message. The binary was
fine; the directory was missing.

The fix adds a sibling helper `ensureArchonWorkspacesPath()` in
`@archon/paths` that mkdir-p's the directory and returns its
path:

```typescript
export async function ensureArchonWorkspacesPath(): Promise<string> {
  const path = getArchonWorkspacesPath();
  await mkdir(path, { recursive: true });
  return path;
}
```

Used at the orchestrator's spawn-cwd site in
`packages/core/src/orchestrator/orchestrator-agent.ts:824`. The
other three callers of `getArchonWorkspacesPath` (workflow
discovery at line 417, path-prefix `startsWith` at line 436,
workflow-deps argument at line 570) only consume the path
string and do not need the directory to exist; they keep using
the pure getter. The pattern mirrors the existing
`ensureProjectStructure(owner, repo)` helper in the same file.

## Why

Reported in coleam00/Archon#1528. The misleading
"Claude Code native binary not found" error on first chat sent
new users hunting through binary-resolution paths when the
actual cause was a missing directory.

## Tests

+38 / -0 in `packages/paths/src/archon-paths.test.ts`. The
orchestrator tests pick up small additions (+6, +1, +1) to
align with the new helper's call site.

## Review

- **coderabbitai** (NONE) reviewed at 2026-05-02T00:18Z on
  commit `32a8f4e` with the standard walkthrough.
- **Wirasm** (COLLABORATOR) cross-referenced #1528 at
  2026-05-04T06:59Z. Followed at 07:03Z with the PR template
  request (UX Journey, Architecture Diagram, Label Snapshot,
  Change Metadata, Linked Issue, Security, Compatibility,
  Human Verification, Side Effects, Rollback, Risks). PR body
  updated at 07:06Z with `Closes #1528` and the template
  sections filled in.
- **Wirasm** review at 15:24Z with verdict `ready-to-merge`,
  no blocking issues. Merged at 15:46:33Z.

## Lesson

- Node's `spawn({ cwd })` throws `ENOENT` on the cwd option
  when the directory does not exist. SDKs whose error handler
  treats every `ENOENT` from the spawn event as a binary-not-
  found case will misattribute the failure. The right shape is
  ensure-the-directory at the caller, not catch-and-translate
  at the SDK.
- Pure getters that return paths whether or not the directory
  exists are correct for read-side consumers and wrong for
  spawn-cwd. Pair a `getX()` with an async `ensureX()` and
  pick the right one per call site rather than making
  every call site await.
- The PR-template request is consistent on coleam00/Archon.
  Even a tightly-scoped fix needs all sections; `Closes #N`
  goes in the PR body explicitly, not only in commit
  messages. Cross-link: `feedback_base_branch_for_squash_release_repos.md`
  for the dev/main split.
