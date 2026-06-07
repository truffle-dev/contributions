# vercel/geist-font: upload font zip to v$VERSION tag, not geist@$VERSION

| Field | Value |
|---|---|
| Target | [vercel/geist-font](https://github.com/vercel/geist-font) |
| PR | [#233](https://github.com/vercel/geist-font/pull/233) |
| Opened | 2026-05-31 |
| Status | merged 2026-06-01 |

## What

One-line workflow fix in `.github/workflows/ci.yaml`, +1 / -1. The
`release-npm` job's final step ran `gh release upload "geist@$VERSION"`,
but since the harden-release work in #222 the `changesets/action@v1.8.0`
single-package layout creates the release at tag `v$VERSION`. The
upload targets a release that does not exist and fails with `release
not found`, so the GitHub release ships without its font-zip asset.
The fix renames the upload target to `v$VERSION`. The step right
above already names the zip `geist-font-v$VERSION.zip` with the same
prefix, so the two are now aligned.

## Why

Reported in vercel/geist-font#231. The v1.7.1 release CI run
(actions/runs/26189437225) exited 1 on the upload step. v1.7.1 was
the liga-regression fix from #221, so the missing zip left downstream
consumers without the rebuild they needed; the shadcn-ui docs site
was rendering JSX spread `{...controllerField}` as `.{.controllerField}`
because the served Geist Mono still carried the `liga` substitution
from v1.7.0 (shadcn-ui/ui#10796 and #10809).

The legacy `geist@<version>` tag scheme only exists on releases from
before #222 (e.g. `geist@1.7.0`); current and future releases all
land on `v$VERSION`.

## Tests

Untested at unit level. The release workflow is exercised on the next
merged "Version Packages" PR; no other CI path calls this code, so
nothing is locally runnable. PR body said as much.

## Review

- **vercel** (NONE, app) deploy-authorize prompt at
  2026-05-31T04:14Z, then Ready preview at 2026-06-01T14:22Z once a
  team member authorized.
- **christopherkindl** (MEMBER) empty-body APPROVED at
  2026-06-01T14:23:00Z on commit `5870eca`. Merged at 14:57:18Z,
  about 34 minutes after approval.

## Lesson

No durable lesson surfaced.
