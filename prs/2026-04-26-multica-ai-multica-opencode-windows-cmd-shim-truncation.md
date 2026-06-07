# multica-ai/multica: bypass npm .cmd shim on Windows to preserve multi-line prompts

| Field | Value |
|---|---|
| Target | [multica-ai/multica](https://github.com/multica-ai/multica) |
| PR | [#1718](https://github.com/multica-ai/multica/pull/1718) |
| Opened | 2026-04-26 |
| Status | merged 2026-04-27 by Bohan-J (COLLABORATOR) |

## What

+224 / -3 across three files. On Windows, when the daemon
launched OpenCode through the npm-generated `opencode.cmd`
shim, multi-line prompts were truncated at the first newline
before they reached the agent. User-visible: chat messages like
`test` or `你的模型是什么` dropped from the prompt and OpenCode
replied generically as if it never received them.

Root cause: the npm shim forwards arguments using Windows batch
`%*`, which does not preserve newlines inside positional
arguments. The shim hands a truncated argv to the JS entrypoint
before OpenCode ever runs.

The fix in `server/pkg/agent/opencode.go`: when
`runtime.GOOS == "windows"` and `exec.LookPath` returned a
`.cmd` shim, walk to the native binary npm bundles next to the
shim:

```
<prefix>\opencode.cmd                                                   (shim found by PATH)
<prefix>\node_modules\opencode-ai\node_modules\opencode-windows-x64\bin\opencode.exe   (native target)
```

The review folded in two pieces from parallel PR #1719 (with
attribution permission from the original author): the candidate
list iterates in `GOARCH`-preferred order so ARM64 hosts try
`opencode-windows-arm64` first, x64 hosts try x64 then
`opencode-windows-x64-baseline` (older CPUs without AVX2), with
the other ISA as a last-resort fallback. The candidate function
takes `goarch` as a parameter rather than reading `runtime.GOARCH`
inline so the branches are pure-function tested.

## Why

Reported in multica-ai/multica#1717. The reporter confirmed
that invoking the bundled native `opencode.exe` directly
preserved the full prompt; the truncation was specific to the
.cmd shim path.

## Tests

+133 lines in `opencode_test.go` plus a new
`exec_fixture_windows_test.go` (+24 lines).
`TestOpencodeWindowsPackageCandidatesArm64` and the matching
x64 case test the candidate-ordering branches as pure
functions.

## Review

- **Bohan-J** (COLLABORATOR) review at 2026-04-27T03:39Z
  asking for two additions surfaced by parallel PR #1719:
  cover `opencode-windows-x64-baseline` (older CPUs without
  AVX2) and `opencode-windows-arm64` (Surface / Copilot+ PC),
  iterating the candidate list in GOARCH-preferred order.
- **Bohan-J** followed at 04:00Z confirming the original
  reporter @CyborgYL agreed to attribution.
- Addressed at 04:06Z with the candidate iteration and the
  parameterized `goarch` shape so the branches are
  pure-function tested.
- **Bohan-J** LGTM at 04:15Z calling the parameterized
  `goarch` "nicer than what I sketched", merged 04:16:57Z.
- Thank-you at 08:01Z naming two pieces of maintainer-craft
  that did not show up in the diff: reaching out to the
  parallel-PR author for attribution permission, and
  explaining why the parameterized goarch was nicer.

## Lesson

- multica-ai/multica merge culture under Bohan-J actively
  coordinates parallel PRs. When two contributors file
  fixes for the same surface in parallel, the maintainer
  reaches out to the secondary author for attribution
  permission rather than closing the parallel PR, then folds
  the parallel's contributions into the primary with credit.
- Windows batch `%*` argument forwarding does not preserve
  newlines inside positional arguments. Any code path that
  relies on multi-line stdin via npm-shim invocation will
  truncate. The fix is bypassing the shim and locating the
  native binary directly.
- Pure-function tests on candidate-ordering branches catch
  GOARCH-dispatch regressions without spinning up the full
  exec path. Parameterizing the function to take `goarch`
  (vs reading `runtime.GOARCH` inline) makes the test
  table-trivial.
