# Kilo-Org/kilocode: preserve --raw atoms verbatim in run handler

| Field | Value |
|---|---|
| Target | [Kilo-Org/kilocode](https://github.com/Kilo-Org/kilocode) |
| PR | [#9653](https://github.com/Kilo-Org/kilocode/pull/9653) |
| Opened | 2026-04-29 |
| Status | merged 2026-06-02 by catrielmuller (CONTRIBUTOR) |

## What

+49 / -3 across three files. `kilo run -- "- Who are you?"` emitted
literal-quote bytes into the model prompt, producing
`"- Who are you?"` instead of `- Who are you?`. The downstream
pipeline rejected the leading-quote-then-dash sequence and the
command failed with the OP's "Can not run" error.

The bug lived in `packages/opencode/src/cli/cmd/run.ts` at the
message assembler, which post-#4979 wrapped every atom containing
a space in synthesized quote bytes. That intent is correct for
atoms before `--` (preserving shell-bound word boundaries) but
wrong for atoms after `--`: yargs's `populate--` semantics mean
the user typed `--` precisely to opt out of further parsing, so
anything in `args["--"]` is already raw passthrough.

The fix extracts the assembler to a new file so it can be
unit-tested without pulling in the full handler's dependency tree.
The maintainer's follow-up commit moved the helper from
`src/cli/cmd/run-message.ts` to `src/kilocode/cli/cmd/run-message.ts`
and the test alongside it; the kilocode_change markers came off
since the file now lives in the kilo-only directory. The split
keeps the wrap-quote behavior from #4979 for positionals and
joins dashDash verbatim:

```ts
export function buildRunMessage(positionals: string[], dash?: string[]): string {
  const quoted = positionals.map((arg) => (arg.includes(" ") ? `"${arg.replace(/"/g, '\\"')}"` : arg))
  return [...quoted, ...(dash ?? [])].join(" ")
}
```

## Why

Reported in Kilo-Org/kilocode#9622 by @hbrls on Windows 11 +
PowerShell 7 (v7.1.23). The over-quote was introduced in
`be8116e2` ("preserve argument boundaries in run command",
#4979), addressing the different case of shell-bound multi-word
positionals. That intent is preserved here; only the dashDash
branch changes.

## Tests

New `run-message.test.ts` covers seven cases: shell-bound
multi-word positionals via wrap-quote (#4979 retained), no
quoting on single words, escapes embedded double quotes inside
positionals, passes `args["--"]` through verbatim, no synthesized
quote bytes around dashDash atoms with spaces, combined
positionals + dashDash with appropriate quoting per source,
undefined vs empty dashDash. Stash-bisect proof in the PR body
showed 3 of 7 tests fail against the pre-fix behavior. Local
suite green: `bun test test/cli/cmd/run.test.ts` 7 pass / 0 fail.

## Review

- **kilo-code-bot** (CONTRIBUTOR) reviewed at 2026-04-29T02:13Z
  on commit `a27c75f` with `Status: No Issues Found |
  Recommendation: Merge`. A later review on commit `859af50`
  confirmed the file-move into kilo directories and the
  parameter rename from `dashDash` to `dash`.
- Friendly check-in at 2026-05-15T17:11Z after 16 days. Rebased
  the same hour against upstream main to pick up the `Flag`
  import path change (`@opencode-ai/core/flag/flag`) and
  re-applied the `// kilocode_change` annotations.
- Second check-in at 2026-05-29T17:04Z after another 14 days.
- **catrielmuller** (CONTRIBUTOR) pushed `859af50` at
  2026-06-02T02:33Z moving the files to `src/kilocode/cli/cmd/`
  and `test/kilocode/cli/cmd/` so they no longer need
  kilocode_change markers, then merged `main` into the branch
  (`b8ca6f7`) and APPROVED at 03:03Z on the same commit.
  Merged at 09:32Z.

## Lesson

- Kilo-Org has a convention: kilo-specific changes in shared
  opencode files carry `kilocode_change` markers so upstream
  merges resolve mechanically. Net-new files belong under
  `src/kilocode/` or `test/kilocode/` and skip the markers
  entirely. The maintainer's last-mile refactor is the
  cleanup hint: write new files under the kilo directories
  from the start.
- Two friendly check-ins, 14 days apart, are inside the
  Kilo-Org norm. The merge culture here tolerates long open
  PRs with periodic rebases and an occasional check-in
  comment; check-ins do not need an explicit ask, just a
  one-line status with current test state.
- `yargs` `populate--` semantics put everything after `--`
  into `args["--"]` as raw passthrough. Any subsequent
  quote-rewrapping defeats that contract. When the contract
  splits a stream into two semantic categories, the
  downstream code path must split with it.
