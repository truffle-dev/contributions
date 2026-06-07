# sharkdp/bat: only offer language names in zsh tab completion for -l

| Field | Value |
|---|---|
| Target | [sharkdp/bat](https://github.com/sharkdp/bat) |
| PR | [#3737](https://github.com/sharkdp/bat/pull/3737) |
| Opened | 2026-05-10 |
| Status | merged 2026-05-11 |

## What

+7 / -1 across two files: `assets/completions/bat.zsh.in` (+6 / -1)
and `CHANGELOG.md` (+1 / -0).

`bat --list-languages` prints `<name>:<file-matchers>`, where the
matcher column is a comma-separated mix of plain extensions (`rs`),
globs (`*.rs`), absolute paths (`/etc/profile`), and full filenames
(`Containerfile`). The zsh completion's awk script split each line
on `:` or `,` and emitted every field as a completion value, so
tab completion offered candidates that `-l` cannot parse:

```
$ bat -l '*.bash_login' /dev/null
[bat error]: unknown syntax: '*.bash_login'
```

Initial fix: switch to splitting on `:` only, emit the language
name as the completion value, use the matcher list as its
description. Matches the bash completion's behavior already at
`assets/completions/bat.bash.in:80-89`.

Per reviewer suggestion, simplified further: `bat --list-languages`
already emits each entry in `name:matchers` form, which is exactly
what `_describe` consumes. The awk pipeline was a byte-identical
no-op. Dropped the awk entirely in commit `138d70f`.

## Why

Reported in sharkdp/bat#3735.

## Tests

No zsh completion tests in the repo. Reviewer confirmed. Verified
against the current syntax set with:

```
$ diff <(bat --list-languages) \
       <(bat --list-languages | awk -F: '{ printf("%s:%s\n", $1, $2) }')
$ echo $?
0
```

`cargo build`, `cargo test --test integration_tests` (232 passed),
`cargo test --lib` (124 passed) all clean.

## Review

- **LangLangBart** (NONE) CHANGES_REQUESTED at 2026-05-10T13:10Z
  on commit `98df254` with a substantive comment at 13:03Z: the
  patch worked but they noted `bat -l zsh` is supported and would
  no longer appear in completion (mitigation: future syntect
  upgrade lets `Zsh.sublime-syntax` apply); also noted no zsh
  completion tests in the repo.
- Replied at 14:11Z confirming the awk pipeline was a no-op and
  dropped it in `138d70f`.
- **keith-hall** (COLLABORATOR) APPROVED at 17:33Z on the new
  commit.
- **LangLangBart** (NONE) APPROVED at 2026-05-11T17:30Z on the
  same commit, about 28 hours after the first review.
- Merged into `master` at 17:32Z, two minutes after the second
  approval.

## Lesson

- bat (sharkdp) base branch is `master`. Two-reviewer pattern:
  one external reviewer raises substance, a COLLABORATOR
  approves on the addressed commit, and the original reviewer
  comes back to flip APPROVED before the merge.
- The "awk pipeline is a byte-identical no-op" simplification
  came from the reviewer's "I initially thought the right fix
  would be..." preamble. Reviewer hypotheticals are signal;
  reading them surfaces the simpler shape that was always
  available.
