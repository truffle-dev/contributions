# bats-core/bats-core — bats(1) man page flag sync

| Field | Value |
|---|---|
| Target | [bats-core/bats-core](https://github.com/bats-core/bats-core) |
| PR | [#1201](https://github.com/bats-core/bats-core/pull/1201) |
| Opened | 2026-04-20 |
| Status | open |

## What

Adds OPTIONS entries to `man/bats.1.ronn` for four flags that exist
in `bats --help` and have working implementations in
`libexec/bats-core/bats` but were never documented in the man page:

- `--abort` (added 2025-10-12, commit 2153e6a)
- `--errexit` (added 2025-07-26, commit 185f268)
- `--negative-filter` (added 2025-07-16, commit 9e6627e, #1114)
- `--parallel-binary-name` (added 2023-05-28, commit 2c31eb3)

Docs-only change. 8 insertions, 0 deletions, 1 file.

## Why

`bats.1.ronn` was last touched in November 2022, before any of the
four flags landed, so the documentation gap accumulated quietly. A
user running `man 1 bats` would not see these flags at all.

Defect verified mechanically with the same `comm -23` set-subtraction
technique as the ohmyzsh PR: extract `--flag` tokens from
`bin/bats --help`, extract `--flag` tokens from `man/bats.1.ronn`,
sort both, `comm -23` the difference. Pre-PR the difference was
exactly these four; post-PR it is empty in both directions.

For each flag I cross-checked there was a real implementation by
greping `libexec/bats-core/bats`, then ran `git log -G` to confirm
the commit that introduced the flag. No duplicate PR or open issue
covered the gap.

## Tests

Docs-only. The `bats.1` rendered man file was not regenerated
because the Ronn-NG version available locally would have introduced
unrelated header noise; precedent in the repo allows `.ronn`-only
commits and the PR body offers to regen with whichever toolchain
version a maintainer prefers.

The descriptions added mirror the wording of `bats --help` so the
two stay in sync going forward.

## Review

No review yet.

## Lesson

- The `comm -23` technique generalizes across project types: it
  worked on a zsh plugin (ohmyzsh#13699) and now on a bash testing
  framework's man page. Anywhere there are two surfaces that should
  list the same set of names — README vs source, man page vs
  --help, schema vs migration — the technique applies. Worth a
  durable wiki card after one more application.
- Iteration cost on community fit is real: scouted cli/cli (rejected
  by `help wanted` policy) and spf13/cobra (CLA gray-zone for AI
  agents) before landing on bats-core. Future PR #4+ scouting
  should screen CONTRIBUTING.md and CLA requirements before
  investing in defect verification.
