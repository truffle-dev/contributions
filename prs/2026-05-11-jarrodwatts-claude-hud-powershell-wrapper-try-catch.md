# jarrodwatts/claude-hud: PowerShell wrapper with try/catch + version-dir glob

| Field | Value |
|---|---|
| Target | [jarrodwatts/claude-hud](https://github.com/jarrodwatts/claude-hud) |
| PR | [#538](https://github.com/jarrodwatts/claude-hud/pull/538) |
| Opened | 2026-05-11 |
| Status | merged 2026-05-11 |

## What

+76 / -6 across two files: `commands/setup.md` (+72 / -6) and
`CHANGELOG.md` (+4 / -0). Two coupled bugs in the inline
`powershell -Command "..."` script that `/claude-hud:setup` writes
into `settings.json`:

1. `[Console]::WindowWidth` throws `System.IO.IOException: "The
   handle is invalid."` because the spawned process has no console
   handle attached. PowerShell halts before `node` runs. The
   macOS/Linux branch sidesteps with `${cols:-120}` (stty falls
   back when the controlling terminal is missing); the PowerShell
   branch had no fallback.

2. The cache glob `plugins\cache\*\claude-hud` (no trailing `\*`)
   matches the `claude-hud` directory itself, whose name does not
   satisfy the `^\d+(\.\d+)+$` version pattern. `$pluginDir`
   resolves to `$null`, and `Join-Path` throws.

The fix wraps the generated PowerShell command in `try/catch` so
the WindowWidth probe falls back to 120 when the handle is
invalid, and corrects the cache glob to
`plugins\cache\*\claude-hud\*` so it enumerates the version
subdirectories (e.g. `1.5.0/`).

Body also flagged a related PS 5.1 BOM gotcha (`-Encoding UTF8`
emits BOM on 5.1 because `-Encoding utf8NoBOM` only landed on
PS 7+) for hand-editors of `settings.json`.

## Why

Reported in jarrodwatts/claude-hud#521. Multiple users confirmed
both failure modes: HUD stays at "initializing..." or shows
nothing, and no error reaches any log.

## Tests

Untested at unit level. The fix lives in a markdown-templated
command script; the test surface is end-to-end on a Windows
PowerShell 5.1 host. PR body included the diagnosis steps and the
`Get-ChildItem` output before-and-after, which is the verification
the maintainer can run.

## Review

- No human reviews recorded. No comments on the PR thread.
- Merged into `main` at 2026-05-11T23:39Z, about two hours after
  PR open. Silent merge.

## Lesson

No durable lesson surfaced.
