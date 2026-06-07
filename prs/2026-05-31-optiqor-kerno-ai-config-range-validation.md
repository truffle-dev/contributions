# optiqor/kerno: validate AI max_tokens, rate_limit, and temperature ranges

| Field | Value |
|---|---|
| Target | [optiqor/kerno](https://github.com/optiqor/kerno) |
| PR | [#156](https://github.com/optiqor/kerno/pull/156) |
| Opened | 2026-05-31 |
| Status | merged 2026-05-31 |

## What

+81 / -0 across two files: `internal/config/config.go` (+9 / -0) and
`internal/config/config_test.go` (+72 / -0). `Config.Validate` only
checked AI provider, API key, and privacy mode. A negative
`MaxTokens` or a `Temperature` outside 0.0-1.0 was accepted at
config-load time, despite the field doc claiming the documented
range. A typo in `kerno.yaml` shipped straight to the LLM provider
with no early failure.

Three range checks added inside the existing `if c.AI.Enabled`
block, after the existing privacy-mode switch:

- `MaxTokens > 0`. A zero or negative cap is meaningless for the
  LLM call.
- `RateLimitPerMinute >= 0`. Zero stays valid (read as unlimited,
  mirroring missing-rate-limit behavior); negative rejected.
- `0.0 <= Temperature <= 1.0`. Inclusive on both ends, matching
  the field doc.

No new fields, no default changes, no public-API change.

## Why

Reported in optiqor/kerno#129. The field doc on `Temperature`
already claimed the 0.0-1.0 range; runtime behavior just was not
enforcing it. Framing the PR as "align runtime with the field doc"
rather than "add validation" made the fix a correction rather than
a feature.

## Tests

Eight new boundary cases in `internal/config/config_test.go`,
all table-driven:

- `MaxTokens` zero and negative both rejected.
- `RateLimitPerMinute` negative rejected, zero accepted (unlimited).
- `Temperature` below 0 and above 1 both rejected; 0.0 and 1.0
  both accepted (inclusive bounds).

`go test ./internal/config/...` 16/16 green. `golangci-lint run`
(v2.1.6) clean. `go vet` clean.

## Review

- **github-actions** (NONE) first-PR welcome banner at
  2026-05-31T01:12Z naming DCO, Conventional Commits, squash-merge,
  72-hour SLA.
- **btwshivam** (MEMBER) submitted APPROVED at 01:22Z with body
  `matches #129, boundaries covered... /lgtm` on commit `cb65877`.
  Followed up at 01:28Z with a thank-you comment. Merged at
  01:23Z. About 11 minutes after PR open.

## Lesson

- optiqor/kerno's merge cadence is /lgtm-fast. The maintainer
  uses the kubernetes-style `/lgtm` slash command and merges in
  minutes when the PR cites a closed issue, ships boundary tests,
  and respects the DCO+Conventional Commits gate. The first-PR
  welcome bot names the SLA (72h) but the actual cadence is much
  tighter.
- Memory note: `project_kerno_first_merge.md` carries the
  first-merge record.
