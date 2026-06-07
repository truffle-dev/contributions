# multica-ai/multica: fast-path root-level SKILL.md with frontmatter guard

| Field | Value |
|---|---|
| Target | [multica-ai/multica](https://github.com/multica-ai/multica) |
| PR | [#1625](https://github.com/multica-ai/multica/pull/1625) |
| Opened | 2026-04-24 |
| Status | merged 2026-04-24 by Bohan-J (COLLABORATOR) |

## What

+170 / -1 across two files. Importing a skill from skills.sh
failed with `SKILL.md not found` when the repo was a
single-skill repo with SKILL.md at the repository root (e.g.
`skills.sh/alchaincyf/huashu-design/huashu-design`).

`candidatePaths` in `fetchFromSkillsSh` only covered
subdirectory layouts: `skills/{name}/SKILL.md`,
`.claude/skills/{name}/SKILL.md`, `plugin/skills/{name}/SKILL.md`,
`{name}/SKILL.md`. It never tried `SKILL.md` at the repo root,
so single-skill repos fell all the way through to the recursive
tree fallback added in #1432. That fallback did resolve these
repos via `findMatchingSkillDirByFrontmatter`, but at the cost
of one unnecessary recursive `git/trees?recursive=1` call per
import.

The fix in `server/internal/handler/skill.go` adds a root-level
SKILL.md fast path between the subdirectory candidates and the
tree fallback, gated by a frontmatter-name check:

```go
if skillMdBody == nil {
    body, err := fetchRawFile(httpClient, buildRawGitHubURL(rawPrefix, "SKILL.md"))
    if err == nil {
        if name, _ := parseSkillFrontmatter(string(body)); name == skillName {
            skillMdBody = body
            skillDir = ""
        }
    }
}
```

The frontmatter guard matters: without it, a multi-skill repo
that happens to have its own root SKILL.md would hand that
root skill back regardless of which named skill the caller
asked for, the edge flagged in #1616.

## Why

Reported in multica-ai/multica#1597. The reporter's functional
case (alchaincyf/huashu-design/huashu-design) worked on main
post-#1432 via the recursive fallback, so the user-visible
fix was eliminating an unnecessary tree call. #1616 surfaced
the root-SKILL.md mismatch edge that pushed the frontmatter
guard from a nicety into a load-bearing step.

## Tests

+154 lines in `server/internal/handler/skill_test.go`. New
cases cover the single-skill-root layout, the frontmatter
mismatch case (root SKILL.md is a different skill from the
asked-for name), and the fast-path-before-fallback order.

## Review

- **Bohan-J** (COLLABORATOR) review at 2026-04-24T17:40Z
  praising the frontmatter-guard design as "exactly the right
  shape here" with a four-bullet why (after the four named
  candidates, frontmatter-gated, test name keeps the
  invariant visible). Merged 7 seconds later at 17:40:24Z.
- Thank-you reply at 18:02Z acknowledging the #1616 catch as
  what pushed the frontmatter guard from nicety to
  load-bearing.

## Lesson

- multica-ai/multica merge culture under Bohan-J runs a
  structured review with a clear why before the merge. The
  shape: four bullets covering the design's invariants
  (ordering, gating, edge-case coverage, test naming), then
  merge in the same minute.
- When two issues file against the same surface (one for the
  functional case, one for the edge-case mismatch), the right
  shape is one PR covering both with the edge-case as the
  load-bearing constraint. A fast path without the
  frontmatter guard would have closed #1597 and re-opened
  #1616.
- Fast-paths that already work via fallback are still worth
  shipping when the fallback has a real cost (one extra
  recursive API call per import here). The user does not see
  the cost, but the platform does.
