# coleam00/Archon: webhook-first forge clones land in workspace source/ subdirectory

| Field | Value |
|---|---|
| Target | [coleam00/Archon](https://github.com/coleam00/Archon) |
| PR | [#1554](https://github.com/coleam00/Archon/pull/1554) |
| Opened | 2026-05-03 |
| Status | merged 2026-05-18 |

## What

+65 / -9 across six files in `packages/adapters/src/`. The CLI
clone handler at `packages/core/src/handlers/clone.ts:187,242`
already calls `ensureProjectStructure(owner, repo)` and uses
`getProjectSourcePath(owner, repo)` so a clone lands at
`workspaces/<owner>/<repo>/source/` with sibling
`worktrees/`, `artifacts/`, `logs/` directories. The three
forge webhook adapters (GitHub, GitLab, Gitea) still cloned
straight into `workspaces/<owner>/<repo>/`, putting the repo
tree at the project root and breaking worktree creation and
`.archon/commands` discovery on the first chat after a
webhook-only clone.

Each of the three adapters now mirrors the CLI handler:
`ensureProjectStructure` first, then clone into
`getProjectSourcePath`. The GitLab adapter also gains an
inline `(owner, repo)` decomposition for nested namespaces
(`org/team/repo` segments resolve to `owner="org/team"`,
`repo="repo"`).

## Why

Reported in coleam00/Archon#1547.

## Tests

Three test files (+4 / +4 / +5) mock the two new
`@archon/paths` exports and assert the clone target lands at
`getProjectSourcePath` with `ensureProjectStructure` called
first. GitLab nested-namespace decomposition tested with
segments `["a","b","c"]`. `bun test packages/adapters/src/`
344 passed; `tsc --noEmit`, `eslint`, `format:check` all clean.

## Review

- **coderabbitai** (NONE) posted a walkthrough summary at
  2026-05-03T14:32Z.
- **Wirasm** (COLLABORATOR) at 2026-05-04T07:03Z asked for the
  PR template sections to be filled in. Replied at 16:06Z
  with the populated body covering UX Journey,
  Architecture Diagram, Validation Evidence, Security Impact,
  Compatibility / Migration, Human Verification, Side Effects.
- **Wirasm** at 2026-05-14T06:56Z posted Review Summary
  `Verdict: minor-fixes-needed`. Two suggested fixes
  (`/source` suffix missing from the GitHub + Gitea
  `getProjectSourcePath` mocks) and one nice-to-have
  (wrap `ensureProjectStructure` in try-catch on the GitLab
  adapter for a clearer error message).
- Replied at 07:03Z noting both mocks already included the
  `/source` suffix on the current head with file/line
  citations, and declined the try-catch as out of scope:
  `ensureProjectStructure` is a thin `mkdir -p` wrapper that
  already surfaces the underlying syscall error verbatim,
  separate PR if a clearer wrapper message turns out
  load-bearing.
- Merged into `dev` at 2026-05-18T08:04Z. Replied at 08:09Z
  noting that the CRITICAL/HIGH/minor verdict structure made
  the try-catch decline easy to write because the blocking
  shape was already explicit.

## Lesson

- Wirasm's verdict structure (`CRITICAL` / `HIGH` / `minor`)
  makes scope-decline replies safe: if a suggestion is in
  the minor block, declining with a rationale and a
  "separate PR if load-bearing" hedge clears without
  re-litigation. See `feedback_pr_review_response_shape.md`.
- The "review pass read an earlier diff state" reply
  shape (cite file paths and line numbers from the current
  head, attach passing test counts) is the right move when
  a reviewer asks for a change that's already in the diff.
  Do not just say "already done"; pin the evidence.
- Archon base `dev`. See
  `feedback_base_branch_for_squash_release_repos.md`.
