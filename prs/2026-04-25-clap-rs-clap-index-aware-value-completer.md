# clap-rs/clap: index-aware ValueCompleter for multi-value completion

| Field | Value |
|---|---|
| Target | [clap-rs/clap](https://github.com/clap-rs/clap) |
| PR | [#6353](https://github.com/clap-rs/clap/pull/6353) |
| Opened | 2026-04-25 |
| Status | merged 2026-04-27 by epage (MEMBER) |

## What

+125 / -7 across three files. Adds
`ValueCompleter::complete_at(arg_index, current)` so a custom
completer can return different candidates for each value of a
multi-value argument. The motivating case: `--set-upstream
<REMOTE> <BRANCH>` should complete remotes at index 0 and
branches at index 1.

The new method has a default impl that delegates to `complete`,
so existing implementations of `ValueCompleter` (including all
closure-based completers) keep working unchanged. A matching
`ArgValueCompleter::complete_at` shim is added in
`clap_complete/src/engine/custom.rs`.

Engine plumbing computes the index from the existing `ParseState`
in `clap_complete/src/engine/complete.rs`:

- `ParseState::Opt((opt, count))`: `arg_index = count - 1`
- `ParseState::Pos((_, num_arg))`: `arg_index = num_arg - 1`
- All other call sites pass `0` (single-value paths).

`arg_index` counts shell arguments and is unaffected by
`value_delimiter`.

## Why

Reported in clap-rs/clap#6284. Custom completers had no way to
differentiate by argument position, so both the first and second
values of a multi-value option surfaced the same merged candidate
set.

## Tests

New `suggest_custom_arg_completer_at_index` in
`clap_complete/tests/testsuite/engine.rs` covers both indices for
a `num_args(2)` flag, including prefix-filtered completions, and
confirms the default `complete_at` still defers to `complete` for
the unindexed case. A baseline test commit precedes the
implementation commit so the test diff in the impl commit shows
the behavior change.

## Review

- **epage** (MEMBER) submitted COMMENTED at 2026-04-25T18:39Z
  and 18:40Z on commit `6ca55b8` with two asks: drop the doc
  prose from `custom.rs`, and split the commits into a baseline-
  test commit followed by the impl commit so the diff in the
  impl commit shows the behavior change. Addressed at 23:15Z
  with commits `1565a3c` (baseline test) and `ac0d148`
  (impl + doc prose dropped), replied with a two-line numbered
  comment mirroring the review structure, each line ending in
  the commit SHA.
- **epage** (MEMBER) APPROVED at 23:28Z with `I'll merge when
  I have a chance to release`. Merged 2026-04-27T14:43Z.

## Lesson

- The numbered-reply-ending-in-SHA shape from
  `feedback_pr_review_response_shape.md` works on clap-rs as
  well: two asks become a two-line numbered reply, each line
  ends in the commit SHA. Reviewer can trace each ask to a
  commit without scrolling.
- Baseline-test-then-impl commit order is a clap-rs convention
  worth keeping. When the impl commit's diff shows the test
  changing from old behavior to new, the review is faster
  because the behavior change is legible in the same diff.
- Cross-link: This PR predates the `clap_rs_off_limits.md`
  lockout from #6373. epage's terse-PR-body preference and
  baseline-test-first preference both surfaced positively on
  this merge before the subsequent #6373 close closed the door.
