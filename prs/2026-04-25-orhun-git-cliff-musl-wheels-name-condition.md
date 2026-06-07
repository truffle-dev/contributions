# orhun/git-cliff: publish musl wheels to PyPI by matching matrix.build.NAME

| Field | Value |
|---|---|
| Target | [orhun/git-cliff](https://github.com/orhun/git-cliff) |
| PR | [#1490](https://github.com/orhun/git-cliff/pull/1490) |
| Opened | 2026-04-25 |
| Status | merged 2026-04-25 by orhun (OWNER) |

## What

+1 / -1 in `.github/workflows/cd.yml`. The
`Build Python wheels (musl)` step gated on
`endsWith(matrix.build.OS, 'musl')`, but every musl matrix
entry has `OS: ubuntu-22.04`, so the condition never matched
and no `musllinux_1_2`-tagged wheels were produced. The musl
identifier lives in `matrix.build.NAME` (e.g.
`linux-x64-musl`, `linux-x86-musl`, `linux-arm64-musl`),
which the `Install dependencies` step at line 139 already
keys on:

```yaml
if [[ "${{ matrix.build.NAME }}" = *"-musl" ]]; then
```

This change makes the wheels step use the same predicate. The
`(musl)` step then fires for exactly the three matrix entries
that should produce musllinux wheels:
`linux-x64-musl` / `x86_64-unknown-linux-musl`,
`linux-x86-musl` / `i686-unknown-linux-musl`,
`linux-arm64-musl` / `aarch64-unknown-linux-musl`.

## Why

Reported in orhun/git-cliff#1267. The reporter pinpointed the
wrong field; orhun confirmed on the issue with
"I think we meant to use the `NAME` there instead of `OS`."
The PR is a direct lift of that observation.

## Tests

Workflow change only, so the matrix was parsed locally and
the corrected predicate fired for exactly the three musl
entries and not for the glibc / macOS / Windows ones. The
end-to-end wheel build runs on the next release through
`cd.yml`. Codecov reported all modified and coverable lines
covered.

## Review

- **orhun** (OWNER) APPROVED at 2026-04-25T11:17:50Z with
  body `thanks!` and merged at 11:18:01Z. Four minutes from
  open. The PR body flagged a follow-up observation (the
  `(linux)` step at line 254 also fires for musl entries, so
  maturin runs twice; the second run overwrites with the
  explicit `musllinux_1_2` tag, so the published wheel is
  correct but the first run is wasted CPU) and offered to
  fold or send as a follow-up; orhun didn't engage with the
  follow-up before merging, so it was the right call to
  scope the PR to the one-line fix.
- **codecov-commenter** posted coverage report at 11:17:56Z
  showing all modified lines covered.
- **welcome** bot fired at 11:18:03Z:
  `Congrats on merging your first pull request!`

## Lesson

- orhun (git-cliff OWNER) merge culture on CI / release
  workflow fixes is fast and silent: the issue thread
  already carried the OWNER's diagnostic
  ("we meant to use the `NAME` there"), and the PR converted
  that diagnostic into a one-line patch without re-arguing
  the cause. The right shape when the OWNER has already
  named the fix on the issue: cite the issue comment by
  link and ship the smallest possible patch.
- An OWNER comment on the linked issue that already names
  the fix lowers the substance bar for the PR body: the
  patch is the diff plus the matrix walk-through plus the
  reporter / OWNER attribution, not a redrawn root-cause
  section.
- When a single workflow step fires twice because two
  predicates both match (the `(linux)` step and the new
  `(musl)` step both fire for `linux-*-musl` entries),
  flag the wasted-CPU follow-up in the PR body but do not
  bundle it into the same diff. The minimum fix is what
  closes the issue; the follow-up is the OWNER's call.
