# starship/starship: honor CLOUDSDK_COMPUTE_REGION env override in gcloud module

| Field | Value |
|---|---|
| Target | [starship/starship](https://github.com/starship/starship) |
| PR | [#7451](https://github.com/starship/starship/pull/7451) |
| Opened | 2026-05-02 |
| Status | merged 2026-05-10 |

## What

+69 / -4 across `src/modules/gcloud.rs` (+68 / -4) and
`docs/config/README.md` (+1 / -0). The gcloud CLI lets
`CLOUDSDK_COMPUTE_REGION` override the active configuration's
`[compute] region`; `gcloud config list` even warns
explicitly:

```
WARNING: Property [region] is overridden by environment setting
[CLOUDSDK_COMPUTE_REGION=europe-west2]
```

Starship was reading the region only from the config file, so
the prompt drifted from what gcloud itself reports when the
env var was set. The fix mirrors the
`CLOUDSDK_CORE_PROJECT` precedent from #2596: check the env
var first, fall back to `gcloud_context.get_region()`, then
run the resulting value through `region_aliases` so aliases
keep working for env-sourced regions.

The doc edit covers both the previously-undocumented
`CLOUDSDK_CORE_PROJECT` override and the new region override
in one line, so users see the full override surface.

## Why

Reported in starship/starship#7448.

## Tests

Two new tests in `src/modules/gcloud.rs` mirror the
`CLOUDSDK_CORE_PROJECT` counterparts:

- `region_set_in_env`: env var + config file both set, env
  var wins.
- `region_set_in_env_with_alias`: env var set, `region_aliases`
  still applies.

`cargo test --lib gcloud`: 14 tests pass (12 existing + 2 new).

## Review

- **cubic-dev-ai** (NONE) at 2026-05-02T03:15Z posted
  `No issues found across 2 files. Confidence score: 5/5.
  Shadow auto-approve: would auto-approve.`
- **davidkna** (MEMBER) APPROVED at 2026-05-03T16:06Z on
  commit `d6992a9`. Merged into `main` at 2026-05-10T19:57Z,
  about a week after approval.

## Lesson

- Mirror an explicit prior-art precedent in the PR body
  (the `CLOUDSDK_CORE_PROJECT` precedent from #2596) when
  adding a sibling env-var override. The reviewer can
  verify the new module follows the same shape without
  reading the module from scratch.
- Quote the upstream tool's own warning text
  (`gcloud config list` printing the override warning) as
  the strongest evidence that starship was lagging the
  authoritative source.
