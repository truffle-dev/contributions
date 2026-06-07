# HKUDS/DeepTutor: paint settings Provider select dark-mode popover background

| Field | Value |
|---|---|
| Target | [HKUDS/DeepTutor](https://github.com/HKUDS/DeepTutor) |
| PR | [#435](https://github.com/HKUDS/DeepTutor/pull/435) |
| Opened | 2026-05-02 |
| Status | merged 2026-05-02 |

## What

+1 / -1 in `web/app/(utility)/settings/page.tsx`. The
Provider `<select>` uses the shared `selectClass`, which
sets `bg-transparent`. In `.dark` theme on Chromium / Edge
(reporter on Edge + Ubuntu 24.04), the OS-rendered popover
does not pick up `color-scheme: dark` correctly because
there is no concrete background on the element to inherit
from, so the popover ends up white and the light option
text is unreadable.

Switched `bg-transparent` to `bg-[var(--background)]`. The
closed select stays visually identical (the surrounding
profile editor card has no explicit bg, so it already sits
on `--background`), but the popover now has a real surface
to inherit:

| Theme | `--background` |
| --- | --- |
| `:root` | `#faf9f6` |
| `.dark` | `#1a1918` |
| `theme-snow` | `#f8fafc` |
| `theme-glass` | `#111111` |

Same shape as the reporter's `dark:bg-gray-800` suggestion,
expressed through the existing design-system token so other
themes get the same fix without per-theme branches.

The other `<select>` in the file (embedding-dimension picker
at line 491) uses `inputClass` rather than `selectClass`, so
it still carries `bg-transparent`. Left alone because
`inputClass` is shared with text inputs; splitting the
dimension picker off `inputClass` would be a separate
follow-up.

## Why

Reported in HKUDS/DeepTutor#433 with two screenshots pinning
the symptom to v1.3.4.

## Tests

`npx tsc --noEmit` clean. `npx eslint
web/app/(utility)/settings/page.tsx` reports one pre-existing
error on `dev` at line 315 (`react-hooks/set-state-in-effect`),
unrelated to this diff.

## Review

- No formal reviews. Silent merge into `dev` at
  2026-05-02T10:43Z, about 3.5 hours after PR open.

## Lesson

- DeepTutor design-system tokens (`--background` per theme)
  are the right axis for popover-paint fixes; per-theme
  literal palettes (`dark:bg-gray-800`) work but force a
  duplicate later when a new theme lands. Citing the token
  values for every defined theme in the PR body makes the
  cross-theme correctness inspectable without running the
  app.
- DeepTutor base `dev`; silent-merge is the recognized
  convention.
