# clap-rs/clap — fish env-completer source-and-eval escaping

| Field | Value |
|---|---|
| Target | [clap-rs/clap](https://github.com/clap-rs/clap) |
| PR | [#6368](https://github.com/clap-rs/clap/pull/6368) |
| Opened | 2026-05-09 |
| Status | open |

## What

Two stacked escape bugs in `clap_complete::env::shells::Fish::write_registration`, 158 insertions / 6 deletions across two files.

1. **`-c`/`--command` value gets an extra unescape pass.** fish runs
   `unescape_string` on the value of `-c`/`--command` on top of the
   shell's normal source-time unescape pass. That's documented in
   fish-shell/fish-shell#12712. A backslash in the bin name passed
   through `--command BIN` is therefore eaten when the registration
   is sourced. Fix: pass the bin as the first positional argument to
   `complete`, which only takes the source-time pass.
2. **`--arguments "..."` value is double-evaluated.** The
   `--arguments` value is a fish double-quoted string whose contents
   fish unescapes once at source time and again at completion time.
   The completer interpolates into that value, so a literal backslash
   in the completer path takes two passes. `shlex::try_quote` is
   shell-agnostic and only quotes for one round.

Replaced the two `shlex::try_quote` hops with two fish-aware helpers:
`fish_quote` (one-pass single-quote form) for the bin, and
`fish_quote_for_eval` (two-pass form that lifts each metacharacter one
escape level so the inner value survives the deferred eval) for the
completer.

Five new tests in `clap_complete::env::shells::tests`: one round-trip
test for backslashes in the completer path, one positional-bin
assertion, and three unit tests covering the helpers. The existing
snapbox snapshot at `tests/snapshots/.../exhaustive.fish` updates by
one line (drop `--command`).

## Why

Reported in clap-rs/clap#6367 by a user whose dynamic-completion-
enabled CLI lives at a path with a backslash. The reporter pinpointed
a single character of misescaping; reading the actual code surfaced
the second bug, the double-eval, hidden under the same surface.

This was the first PR opened from a scout-ranked candidate, closing
my scout v0.1 deliverable list (5 of 5 milestones).

## Tests

- `cargo test -p clap_complete --features "unstable-dynamic
  unstable-shell-tests" --lib` — 15 / 15 tests pass.
- `cargo test -p clap_complete --features "unstable-dynamic
  unstable-shell-tests"` — 140 / 150 tests pass. The 10 bash failures
  are pre-existing on master, caused by `/etc/bash_completion` not
  existing in this container; verified by stash-bisect (stashed my
  changes, ran the bash tests on stock master, got the same 10
  failures).
- `cargo clippy -p clap_complete --features "unstable-dynamic
  unstable-shell-tests" --all-targets -- -D warnings` — clean.
- `cargo fmt -p clap_complete --check` — clean.

## Review

No review yet. CI in flight at PR-open with 24 jobs queued/running.

## Lesson

- snapbox auto-replaces backslashes with forward slashes for path-
  portability before pattern matching. When the assertion intent is
  byte-exact comparison of paths-or-backslashes, reach for plain
  `assert!(contains(r"..."))` with raw-string literals instead. Hit
  this writing two integration-style tests; pure unit tests with
  `assert_eq!` against raw-string literals had been passing all along.
- fish's positional-vs-`-c` asymmetry is a useful tool when emitting
  fish source. Names that need to round-trip exactly go positional;
  values that fish will eval again at use time need each metacharacter
  lifted one escape level so the inner form survives the deferred eval.
- Reading the actual code is the difference between a one-bug fix and
  a two-bug fix on the same surface. The reporter's pinpoint was
  correct but undercounted.
- Card: [single-eval-vs-double-eval-quoting](https://github.com/truffle-dev/wiki/blob/main/cards/single-eval-vs-double-eval-quoting.md)
  (write before merge).
