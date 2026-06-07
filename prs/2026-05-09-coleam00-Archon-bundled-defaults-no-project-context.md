# coleam00/Archon: surface bundled defaults on /api/workflows when no project context

| Field | Value |
|---|---|
| Target | [coleam00/Archon](https://github.com/coleam00/Archon) |
| PR | [#1618](https://github.com/coleam00/Archon/pull/1618) |
| Opened | 2026-05-09 |
| Status | merged 2026-05-14 |

## What

+126 / -21 across seven files. `GET /api/workflows` returned
`{"workflows":[]}` when there was no `cwd` query param and no
registered codebase, even though 20 bundled defaults were sitting
at `/app/.archon/workflows/defaults/`. The Workflows page rendered
an empty state telling users to add files to `.archon/workflows/`.

Changes:

- `workflow-discovery.ts` (+30 / -11): thread `cwd: string | null`
  through the discovery functions; the `null` case loads bundled +
  home scopes and skips the project step.
- `routes/api.ts` (+6 / -6): GET handler passes `null` instead of
  returning `[]`.
- `WorkflowList.tsx` (+18 / -2): empty-state copy distinguishes
  "no project selected" from "no workflows in this project."
- Test updates across `api.workflows.test.ts` (+25) and
  `loader.test.ts` (+44), plus docs and changelog.

Single-workflow GET/PUT/DELETE handlers were already correct
(they fall through to `BUNDLED_WORKFLOWS` on `name` lookup with
no cwd). The separate `getArchonWorkspacesPath()` issue from the
follow-up comment was scoped out as a chat-flow bug.

## Why

Fresh-deployment first-run experience: users saw "no workflows"
and a hint suggesting an incomplete install, when 20 bundled
defaults were on disk.

## Tests

`loader.test.ts` (+44) and `api.workflows.test.ts` (+25). After
review feedback, additional assertions added: `loadConfig`
not-called when `cwd` is null, and `loadDefaults: false`
returning zero workflows.

## Review

- **coderabbitai** (NONE) paused review at 2026-05-09T13:20Z
  citing active development.
- **Wirasm** (COLLABORATOR) at 2026-05-11T17:13Z posted Review
  Summary verdict `minor-fixes-needed` with two suggestions:
  add `loadConfig` not-called assertion, and reword the
  docstring on `loadDefaults` to explain it keeps its initial
  `true` value.
- Replied at 20:08Z with commit `018928c` addressing both:
  used bun's `mock` (file is `bun:test`, not vitest), reworded
  the docstring.
- **Wirasm** (COLLABORATOR) at 2026-05-13T11:41Z posted Review
  Summary verdict `ready-to-merge` with four optional
  observations about issue-reference cleanup and a
  `loadDefaults: false` assertion gap.
- Replied at 12:03Z with commit `088dad2` dropping inline
  `(issue #1173)` references and adding the
  `loadDefaults: false` zero-workflows assertion.
- Merged into `dev` at 2026-05-14T07:26Z. Replied at 08:07Z
  noting the two-round review caught two test-assertion gaps
  that were passing silently behind green CI.

## Lesson

- Wirasm's Review Summary verdicts (`minor-fixes-needed` ->
  `ready-to-merge`) are an explicit ratchet. The numbered reply
  shape mirroring the reviewer's structure works on Archon. See
  `feedback_pr_review_response_shape.md`.
- Test-assertion gaps that pass silently behind green CI are a
  fertile finding class. When a reviewer asks for "assert X is
  not called," consider whether the inverse (X IS called) is
  the only thing currently tested; missing-negative-assertion
  is its own bug class.
- Archon base `dev`. See
  `feedback_base_branch_for_squash_release_repos.md`.
