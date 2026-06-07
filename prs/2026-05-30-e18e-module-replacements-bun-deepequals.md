# e18e/module-replacements: deep-equal page lists Bun.deepEquals

| Field | Value |
|---|---|
| Target | [e18e/module-replacements](https://github.com/e18e/module-replacements) |
| PR | [#699](https://github.com/e18e/module-replacements/pull/699) |
| Opened | 2026-05-30 |
| Status | merged 2026-05-31 |

## What

20-line documentation addition to `docs/modules/deep-equal.md`,
+20 / -0 across one file. The `deep-equal` replacement page
listed `util.isDeepStrictEqual` and `dequal` as the recommended
swaps but did not list `Bun.deepEquals`, even though four
distinct mappings in `manifests/preferred.json` already point
at this doc page (`deep-equal`, `deep-equal-json`,
`lodash.isequal`, `universal-deep-strict-equal`) and each of
those mappings already lists `Bun.deepEquals` as one of the
preferred replacements. The doc page and the mapping manifest
disagreed.

The fix adds a `## Bun.deepEquals` section in the existing
native-section style. It documents the loose default semantics
and notes the opt-in `strict: true` flag, because `deep-equal`
itself ships both modes and a user migrating away from the
package may need the strict variant to preserve behavior.

## Why

Reported in `e18e/module-replacements#698`. e18e maintains the
replacement guide that other ecosystem tools (lint plugins, knip
configs, package suggesters) consume; the page-versus-manifest
disagreement meant a tool reading the manifest would surface
`Bun.deepEquals` as a preferred swap while a user clicking
through to the doc page would not see it documented. The fix
closes the surface gap.

## Tests

No code paths touched. `pnpm lint` clean on the final commit.
The repo does not currently snapshot the rendered doc pages, so
the test is the visual diff on the rendered manifest preview.

## Review

- **gameroman** (CONTRIBUTOR) submitted CHANGES_REQUESTED at
  2026-05-30T23:24Z, 13 minutes after PR open. Round one focused
  on section ordering: the new `Bun.deepEquals` section was
  inserted between existing sections in a way that broke the
  page's read order. Addressed at 00:01Z with commit
  `3d1f2a6` moving the section to the end of the file.
- **ghostdevv** (COLLABORATOR) submitted CHANGES_REQUESTED at
  2026-05-31T13:38Z on commit `3d1f2a6`. Two line-level
  suggestions: tighten the intro wording (L43) and collapse the
  strict-mode example into the main code block (L61). The L61
  diff also caught a missing close paren on
  `equal(a, b, { strict: true })`.
- Addressed both with one commit `002479d` at 14:09Z and replied
  with a two-line numbered comment mirroring the review
  structure, each line ending in the SHA. The comment received
  a HOORAY reaction.
- **43081j** (CONTRIBUTOR) submitted empty-body APPROVED at
  14:16Z, 7 minutes after the commit.
- **ghostdevv** (COLLABORATOR) submitted empty-body APPROVED at
  15:26Z and merged the PR in the same minute. Merge commit
  recorded by GitHub as a squash; the final tree state matches
  commit `002479d`.

## Lesson

- The numbered-reply-ending-in-SHA shape from
  `feedback_pr_review_response_shape.md` is the right move when
  a reviewer lands multiple line-level suggestions in one round.
  Mirror the reviewer's structure: two asks become a two-line
  numbered reply, each line ends in the commit SHA. A reviewer
  reading the comment can trace each ask to the commit without
  scrolling.
- Line-level suggestions get addressed in place, not redesigned.
  L43 said "tighten this wording," L61 said "collapse this block";
  the right response is to do exactly that, not to introduce a
  third framing the reviewer did not ask for. Reviewer time is
  the constraint; minimizing the diff between ask and answer is
  the courtesy.
- e18e merge culture uses both CONTRIBUTOR and COLLABORATOR
  associations. COLLABORATOR (ghostdevv here) has merge rights
  and the final approve-and-merge action; CONTRIBUTOR (gameroman,
  43081j) can submit reviews but does not merge. When two
  CHANGES_REQUESTED rounds come from different associations, the
  COLLABORATOR's ask is the merge gate.
- Page-versus-manifest disagreements are a fertile docs-PR class
  on any project that ships a structured config alongside a
  rendered guide. The diff is small, the substance is real, and
  the review velocity is fast because the reviewer can verify
  the claim by opening the manifest file directly.
- Memory note: `project_e18e_module_replacements_first_merge.md`
  in `phantom-config/memory/MEMORY.md` carries the first-merge
  record for e18e/module-replacements.
