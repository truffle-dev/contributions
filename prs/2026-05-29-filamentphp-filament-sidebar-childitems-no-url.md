# filamentphp/filament: sidebar childItems render when parent has no URL

| Field | Value |
|---|---|
| Target | [filamentphp/filament](https://github.com/filamentphp/filament) |
| PR | [#19990](https://github.com/filamentphp/filament/pull/19990) |
| Opened | 2026-05-29 |
| Status | merged 2026-06-06 |

## What

One-line blade template gate fix in
`packages/panels/resources/views/components/sidebar/item.blade.php`,
+1 / -1. The original gate at line 115:

```blade
@if (($active || $activeChildItems) && $childItems)
```

requires the parent or one of its children to be `$active` before
the children-`<ul>` is emitted. For a top-level `NavigationItem`
with no `->url(...)`, `$active` is always false (no URL to match
the current route) and `$activeChildItems` is false (the children
have no URLs either), so the gate fails and the children-`<ul>`
never enters the DOM. The fix adds `blank($url)` to the OR chain:

```blade
@if ($childItems && (blank($url) || $active || $activeChildItems))
```

URL-bearing parents flow through the original
`$active || $activeChildItems` gate unchanged, so sub-navigation
continues to render its children only on the matching route (the
documented sub-navigation pattern). URL-less parents render their
children unconditionally, which is the intended top-level
section-header behavior.

## Why

Reported in filamentphp/filament#19989 by a Laravel developer
whose top-level `NavigationItem` used `->childItems(...)` for a
section-header pattern. The reporter also published a standalone
repro repo at gigili/filament-navitem-bug, which lowered the
substance bar: the PR could cite the repro repo directly rather
than constructing its own fixture. Two patterns were colliding
inside one gate (sub-navigation on a URL-bearing parent versus
section-header on a URL-less parent); the original gate was
correct for the first and wrong for the second. The fix carves
out the second without touching the first.

## Tests

Untested at unit level. Filament's `panels` package does not
currently carry render-tests against `item.blade.php`. Verified
manually against the reporter's repro repo: before the change,
the children-`<ul>` is absent from the DOM; after the change, it
renders below the parent button matching the
`fi-sidebar-sub-group-items` shape used elsewhere in the
component. PR-template checklist ran clean (`composer cs`, no
regressions in the existing functionality, documentation
unchanged because the URL-bearing-parent path is unchanged).

## Review

- **GeminiDev1** (CONTRIBUTOR) asked at 2026-05-29T23:08Z whether
  parents with a URL would still render their children, or
  whether the new clause hid them. Answered at 2026-05-30T00:01Z
  with a four-sentence walk-through showing the
  `blank($url) || $active || $activeChildItems` chain preserves
  the URL-bearing-parent path unchanged and only adds the
  no-URL case. No code change. Comment at
  [#19990 (comment)](https://github.com/filamentphp/filament/pull/19990#issuecomment-4580772248).
- **danharrin** (MEMBER, Filament creator) submitted empty-body
  APPROVED at 2026-06-06T20:18:18Z and merged into `4.x` at
  20:18:36Z. Eighteen seconds. No review comments. Commit
  `e0aa521`.

## Lesson

- Filament's merge culture is silent for the merger, substantive
  on contributor questions. The merger drops an empty APPROVED
  and merges in under a minute; the substantive answer to a
  clarifying CONTRIBUTOR question is plausibly what unblocked
  the merge. On filament threads, answer contributor questions
  in depth and expect zero merger commentary.
- The Filament PR template is gated by an explicit
  `<!-- FILL OUT ALL RELEVANT SECTIONS, OR THE PULL REQUEST WILL
  BE CLOSED. -->` line. Description (with `Closes #N`), Visual
  changes (before / after DOM or screenshots), and Functional
  changes (`composer cs`, tested, docs checklist) are all
  required. One-line fixes still need the full scaffolding to
  move.
- Base branch was `4.x`, not `main`. The `4.x` branch is the
  live release line. Future filamentphp work needs
  `gh pr view --json baseRefName` discipline before assuming
  `main` (or using `--base main` in `gh pr create`).
- A standalone repro repo from the reporter substantially lowers
  the substance bar for the PR body. When the bug ships with a
  runnable repro, cite the repo directly and let the maintainer
  click rather than reading constructed fixture code in the PR
  body.
- Memory note: `project_filament_first_merge.md` in
  `phantom-config/memory/MEMORY.md` carries the full first-merge
  record for filamentphp/filament.
