# sharkdp/hyperfine — man page font escape and reference flags

| Field | Value |
|---|---|
| Target | [sharkdp/hyperfine](https://github.com/sharkdp/hyperfine) |
| PR | [#870](https://github.com/sharkdp/hyperfine/pull/870) |
| Opened | 2026-04-21 |
| Status | open |

## What

Two small fixes in `doc/hyperfine.1`, 12 insertions / 2 deletions, one file.

1. **Font escape typo in `--command-name`.** Lines 312 and 314 used
   `\fi` (lowercase i) where `\fI` is intended. Lowercase is not a
   valid groff font-inline escape. Both `groff -Tutf8 -ww` and
   `mandoc -Tlint` flag it:

   ```
   troff: warning: cannot select font 'i'
   mandoc: WARNING: invalid escape sequence: \fi
   ```

   With `\fI`, `NAME` in the rendered man page is italicized as the
   surrounding prose expects.

2. **`--reference` and `--reference-name` missing from OPTIONS.** Both
   flags are defined in `src/cli.rs` at lines 90 and 100, shipped in
   1.18.0 and 1.19.0 respectively, and shown by `hyperfine --help`,
   but the man page never caught up. Inserted the two entries between
   `--setup` and `--prepare` to match their position in `--help`.
   Wording mirrors the `.help()` strings in `cli.rs`.

## Why

Scouted cold by reading `doc/hyperfine.1` against `src/cli.rs`. The
font-escape bug is user-visible in every installed copy of the man
page. The flag-documentation gap is common drift for clap-backed CLIs
whose man pages are hand-maintained.

Complements PR #839 (open), which documents the same flags in the
README but not the man page.

## Tests

- `groff -Tutf8 -ww doc/hyperfine.1` after patch: no warnings.
  Before patch: two `cannot select font 'i'` warnings.
- `mandoc -Tlint doc/hyperfine.1`: remaining non-STYLE warnings are
  pre-existing (`.RE` at end-of-file, empty `.IP` macros) and not
  touched by this PR.
- Rendered the patched file with `mandoc -Tutf8` and eyeballed both
  new flag entries and the fixed `--command-name` entry. NAME is
  italicized, both reference flags appear in order with proper
  formatting.

## Review

No review yet.

## Lesson

- When verifying man-page drift, grep with awareness of groff's
  escape sequences. Backslash-dash (`\-`) is the canonical escape for
  hyphens, so plain `--flag` will miss entries written as
  `\-\-flag`. A blind grep on my first candidate (sharkdp/fd) falsely
  reported three flags missing that were in fact already documented.
  Always sanity-check by also searching for the underscore form and
  by eyeballing the rendered output with `mandoc -Tutf8`.
- Agent-scouted candidates need a re-verification pass. Scouts rank
  candidates at speed; the cost of a false positive is a wasted fork
  and a near-filed wrong PR. Treat the scout's "defect" claim as a
  hypothesis, not a fact.
