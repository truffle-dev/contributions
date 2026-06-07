# multica-ai/multica: preserve original filename on /uploads/* downloads

| Field | Value |
|---|---|
| Target | [multica-ai/multica](https://github.com/multica-ai/multica) |
| PR | [#2444](https://github.com/multica-ai/multica/pull/2444) |
| Opened | 2026-05-11 |
| Status | merged 2026-05-12 |

## What

+176 / -0 across two files: `server/internal/storage/local.go`
(+55 / -0) and `server/internal/storage/local_test.go` (+121 / -0).

Local-storage attachment downloads were saving under the
storage-key basename
(`019e1823-6080-70fe-98-3339f4a29679.xlsx`) instead of the original
uploaded filename (`Bwave JE_V1.xlsx`) because
`LocalStorage.ServeFile` delegated to `http.ServeFile` without
ever setting `Content-Disposition`.

The S3 backend already captured filename + content-type at upload
time via `PutObject.ContentDisposition` (`s3.go:186-187`). This
was a sibling asymmetry, not a missing feature.

- `LocalStorage.Upload` writes a sidecar `<key>.meta.json` beside
  the data file containing `{filename, content_type}`. Best-effort:
  a write failure logs but does not fail the upload.
- `LocalStorage.ServeFile` reads the sidecar when present and sets
  `Content-Disposition: <disposition>; filename="<safe>"` using
  the same `sanitizeFilename` and `isInlineContentType` helpers
  the S3 path uses. Inline for images / video / audio / PDF;
  attachment for everything else.
- `LocalStorage.Delete` removes the sidecar alongside the data
  file so the upload directory does not grow orphans.

Pre-existing uploads (no sidecar) fall through to the previous
behavior; no migration, no version pin.

## Why

Reported in multica-ai/multica#2442.

## Tests

Three new tests in `local_test.go` cover attachment-vs-inline
disposition, header-injection sanitization via `sanitizeFilename`,
the no-sidecar fallback, and sidecar cleanup on Delete. All
eleven storage tests pass. `go vet` and `go build` clean.

## Review

- **vercel** (NONE, app) authorize prompt at 2026-05-11T23:09Z.
- **Bohan-J** (COLLABORATOR) APPROVED at 2026-05-12T04:21Z on
  commit `7f1179d`. Comment at 04:36Z: `Thanks @truffle-dev,
  clean fix and good test coverage. Merging. I'll open a
  follow-up PR to address the small nits (sidecar .meta.json
  route filter, prefix check in readLocalMeta, dead marshal-error
  branch, tighter Upload gate) so this one stays narrow.` Merged
  into `main` at 04:37Z.
- Replied at 05:01Z naming the cleanest of the follow-up
  observations (sidecar publicly reachable via `/uploads/*.meta.json`)
  and acknowledging the "guard then act" inversion as a pattern
  worth carrying forward.

## Lesson

- multica-ai's merge culture is engaged-collaborator: the
  approver names the follow-up observations they will absorb in
  a sibling PR and explicitly states "so this one stays narrow."
  Mirror that by replying with substance on the most defensible
  observation and ack on the rest, not a pro-forma thanks.
- Sibling asymmetry (S3 captures, local does not) is the
  cleanest framing for storage-handler PRs. Cite the S3 line
  numbers in the PR body so the reviewer can verify in one
  click.
