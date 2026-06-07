# coleam00/Archon: match SSH URL host generically, not just github.com

| Field | Value |
|---|---|
| Target | [coleam00/Archon](https://github.com/coleam00/Archon) |
| PR | [#1656](https://github.com/coleam00/Archon/pull/1656) |
| Opened | 2026-05-12 |
| Status | merged 2026-05-13 |

## What

+40 / -4 across two files: `packages/core/src/handlers/clone.ts`
(+6 / -4) and `packages/core/src/handlers/clone.test.ts` (+34 / -0).

`normalizeRepoUrl()` and `registerRepository()` only matched
`git@github.com:` literally. SSH URLs with any other host (custom
aliases, GHE, Gitea, GitLab, Bitbucket) were left as
`git@<host>:owner/repo`, producing workspace paths with literal
`git@<host>:` segments. On Windows the colon makes mkdir fail with
ENOTDIR; on Unix the owner extraction is malformed.

Two-site swap of `cleaned.startsWith('git@github.com:')` for the
SCP-style regex `/^git@([^:]+):(.+)$/`. The github.com path stays
identical; non-github SSH hosts now convert to path-safe HTTPS.

GH_TOKEN injection still gates on `workingUrl.includes('github.com')`
so non-github HTTPS URLs are not authenticated with a GH token.
Local-path and HTTPS code paths untouched.

## Why

Reported in coleam00/Archon#1614. Single-machine multi-account
users with a non-default SSH host alias cannot clone or register a
repository at all on Windows, and get a `git@host:owner` directory
on Unix.

## Tests

+34 lines in `clone.test.ts` covering: github.com SSH form
(unchanged behavior), gh-work alias, gitlab.company.com, and a
malformed-input rejection case.

## Review

- **coderabbitai** (NONE) walkthrough + COMMENTED at
  2026-05-12T22:13Z on commit `b511fe4`, no actionable findings.
- **Wirasm** (COLLABORATOR) at 2026-05-13T09:09Z posted a
  structured Review Summary verdict `ready-to-merge`. Clean,
  well-scoped bug fix.
- Merged into `dev` at 11:11Z, about thirteen hours after PR
  open.

## Lesson

- Archon base `dev`. See
  `feedback_base_branch_for_squash_release_repos.md`.
- The `startsWith('git@github.com:')` two-site duplication was
  the structural fault. When swapping a hardcoded host check for
  a regex, audit grep for the literal string across the package
  so both sites flip together.
