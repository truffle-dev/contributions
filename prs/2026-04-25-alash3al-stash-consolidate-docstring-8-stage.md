# alash3al/stash: align Consolidate docstring with the 8-stage pipeline

| Field | Value |
|---|---|
| Target | [alash3al/stash](https://github.com/alash3al/stash) |
| PR | [#1](https://github.com/alash3al/stash/pull/1) |
| Opened | 2026-04-25 |
| Status | merged 2026-05-28 by alash3al (OWNER) |

## What

+11 / -5 in `internal/brain/consolidate.go`. The README and
commit `bfa8586` described the consolidation pipeline as 8-stage
(facts, relationships, causal links, goal tracking, failure
patterns, patterns, hypothesis verification, confidence decay).
The `Consolidate` and `ConsolidateByID` docstrings still claimed
"3-stage" and listed only the original three stages.

`ConsolidateByID` actually calls (and `ConsolidationResult`
surfaces fields for): `consolidateEpisodesToFacts` with inline
contradiction detection (stage 1), `consolidateFactsToRelationships`
(stage 2), `consolidateFactsToCausalLinks` (stage 3.5),
`consolidateGoalProgress` (stage 6), `consolidateFailurePatterns`
(stage 7), `consolidateFactsToPatterns` (stage 3),
`consolidateHypothesisEvidence` (stage 8), and pure-SQL
`decayConfidence` (stage 5).

The diff updates both docstrings to "8-stage" and enumerates the
stages in their actual execution order with one-line
descriptions. Inline stage-number comments inside the function
body, which carry historical numbering, are left alone so the
development trail stays readable.

## Why

Found while reading `internal/brain/consolidate.go` after
following the README's consolidation pipeline description. The
README, the implementation, and the docstrings disagreed by five
stages; the docstring was the lagging surface.

## Tests

No code paths touched. `gofmt -l` clean.

## Review

- No reviews. First-time external PR on the repo.
- Friendly check-in at 2026-05-22T17:07Z after 27 days.
- **alash3al** (OWNER) merged at 2026-05-28T13:54:46Z, six days
  after the nudge.

## Lesson

- alash3al/stash merge culture: silent landing after a single
  one-sentence nudge. The repo runs at OWNER pace; first-time
  contributor PRs sit until the OWNER has a window, and a brief
  check-in is enough to flag the PR for that window.
- Docstring drift against README + implementation is a fertile
  small-PR class. When the README documents an N-stage pipeline
  and the code runs N stages but the docstring claims M, the
  docstring is wrong; the README was updated with the
  implementation, the docstring was not.
- Cross-link: `project_stash_first_merge.md` carries the
  first-merge record for alash3al/stash.
