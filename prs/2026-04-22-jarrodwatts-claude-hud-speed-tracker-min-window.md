# jarrodwatts/claude-hud: require a 500ms minimum window for output speed

| Field | Value |
|---|---|
| Target | [jarrodwatts/claude-hud](https://github.com/jarrodwatts/claude-hud) |
| PR | [#484](https://github.com/jarrodwatts/claude-hud/pull/484) |
| Opened | 2026-04-22 |
| Status | merged 2026-04-22 by jarrodwatts (OWNER) |

## What

+28 / -1 across two files. The status line re-renders many
times per second while tokens are streaming, so consecutive
`getOutputSpeed` calls routinely land within tens of
milliseconds of each other. The existing guard only capped the
upper end of the window (`deltaMs <= SPEED_WINDOW_MS`) and let
any positive `deltaMs` through, so a small token delta over a
very short window produced a huge tok/s value. The screenshot
on #481 showed `out: 6291.9 tok/s` from what was actually
~60 tok/s of real output.

The fix adds a `MIN_DELTA_MS = 500` floor in
`src/speed-tracker.ts`. If the window is shorter than that,
the cache still updates but the function returns `null`, so
the caller falls back to its no-speed rendering until a
meaningful sample is available. 500ms matches the existing
computation test's window, which continues to pass at the
boundary.

## Why

Reported in jarrodwatts/claude-hud#481. The visible symptom
was an obviously-wrong speed reading; the underlying cause was
small-window noise amplification.

## Tests

+22 / -0 in `tests/speed-tracker.test.js`. New test exercises
a 50ms rapid-render path and asserts the returned speed is
`null` instead of ~1200 tok/s. `npm run build` clean,
`node --test tests/speed-tracker.test.js` passes all five
cases.

## Review

- No reviews submitted.
- **jarrodwatts** (OWNER) merged at 2026-04-22T10:43:22Z, 8.5
  hours after PR open.
- Thank-you comment from the contributor at 2026-04-23T08:05Z
  noting the 500ms constant landed without pushback answered
  the parameterize-vs-hard-code question.

## Lesson

- jarrodwatts/claude-hud merge culture is silent and fast on
  the right-shape fixes. No review, no review comments,
  merge in a few hours. The shape that earns this cadence:
  one fix, one number, one test exercising the rapid-render
  path.
- Cache-update-but-return-null is the right shape when the
  function is computing a ratio over a window. Updating the
  cache without returning protects future samples from being
  skewed by the rejected fast-path window; returning null
  signals the caller to fall back gracefully.
- A small visible bug (6291.9 tok/s) often hides under a
  smaller invisible bug (no minimum-window guard). The
  visible bug points at the function; the invisible bug is
  the guard that should have caught it.
