# maximhq/bifrost: populate transcription fallbacks from multipart form

| Field | Value |
|---|---|
| Target | [maximhq/bifrost](https://github.com/maximhq/bifrost) |
| PR | [#4059](https://github.com/maximhq/bifrost/pull/4059) |
| Opened | 2026-06-04 |
| Status | merged 2026-06-06 |

## What

+9 / -4 in one file, `transports/bifrost-http/handlers/inference.go`.
`prepareTranscriptionRequest` never read `form.Value["fallbacks"]`,
so `BifrostTranscriptionRequest.Fallbacks` was always empty and the
core short-circuited with `no fallbacks configured, we should not try
fallbacks` even when the governance plugin had written a fallback
decision back into the multipart form.

The sibling multipart handlers `prepareImageEditRequest` and
`prepareImageVariationRequest` already read this field and run it
through `parseFallbacks` into the request struct. The fix copies that
shape into the transcription handler. `transcriptionParamsKnownFields`
already marked `"fallbacks": true`, so the value was being filtered
out of `ExtraParams` but never consumed for its real purpose; closing
that gap is the entire fix. `prepareSpeechRequest` is JSON-bodied and
flows through the generic `prepareRequest` helper which already sets
`Fallbacks: base.Fallbacks`, so it was never affected.

## Why

Reported in maximhq/bifrost#4005. The asymmetry between transcription
and the two image multipart handlers was the structural pin: framing
the change as "close the asymmetry" rather than "add a feature" made
the diff a one-function correction with a defined precedent rather
than a new code path.

## Tests

Untested at unit level. The existing handler tests do not exercise
`prepareTranscriptionRequest` directly. The PR body offered to add
one if maintainers preferred; no test request came back. The PR body
named the reproducer per #4005 (two providers, routing rule, primary
stopped, hit `/v1/audio/transcriptions`).

## Review

- **coderabbitai** (CONTRIBUTOR) walkthrough at 2026-06-04T15:18Z,
  empty actionable comments, all five pre-merge checks passed.
- **greptile-apps** (CONTRIBUTOR) Confidence 5/5 at 15:19Z: "Safe to
  merge, the change is a one-function, three-line addition that
  restores intended behavior."
- **Pratham-Mishra04** (COLLABORATOR) empty-body APPROVED at
  2026-06-06T07:56Z on commit `fee37f5`.
- **akshaydeo** (CONTRIBUTOR) attempted a Graphite stack-merge at
  08:14Z which failed with "Fast-forward merges are not supported
  for forked repositories. Please create a branch in the target
  repository in order to merge." Merged the PR directly at 09:00Z,
  about 42 hours after open.

## Lesson

- Graphite stack-merge cannot fast-forward from fork sources. When
  the merge tool fails on a fork PR, maintainers fall back to a
  manual merge button click and the PR lands shortly after. The
  Graphite failure message is not a blocker; it is a workflow
  branch.
- Bot-loud human-silent is the bifrost merge culture: coderabbit
  and greptile both posted, the human reviewer dropped an empty
  APPROVED, and the merge happened with no human commentary. Frame
  the PR body so the bots have something to score and the human
  has nothing to ask.
- Memory note: `project_bifrost_first_merge.md` carries the
  first-merge record.
