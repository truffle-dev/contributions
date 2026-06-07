# tracel-ai/burn: re-export BurnConfig with fusion/autodiff getters

| Field | Value |
|---|---|
| Target | [tracel-ai/burn](https://github.com/tracel-ai/burn) |
| PR | [#4959](https://github.com/tracel-ai/burn/pull/4959) |
| Opened | 2026-05-15 |
| Status | merged 2026-05-15 |

## What

+33 / -21 across five files. Three coupled changes to the
`BurnConfig` public surface:

- `crates/burn/src/lib.rs`: re-export `BurnConfig` from the umbrella
  crate alongside the already-public `runtime_config()` function.
- `crates/burn-std/src/config/base.rs`: privatize `BurnConfig::fusion`
  and `BurnConfig::autodiff` fields; add `fusion()` and `autodiff()`
  getters returning `&FusionConfig` and `&AutodiffConfig`.
- `crates/burn-std/src/config/logger.rs` and
  `crates/burn-fusion/src/search/optimization/stream.rs`: update the
  three internal field-access call sites to the new getters.

The `impl RuntimeConfig for BurnConfig` block keeps direct field
access on `self.fusion.logger.*` inside `override_from_env`; that
stays inside the defining module so the visibility change does not
need a mutable accessor.

## Why

Reported in tracel-ai/burn#4932. `BurnConfig` was returned by a
public function but was not itself re-exported through the umbrella
crate, so downstream consumers either reached past the umbrella to
`burn_std::config::base::BurnConfig` or could not name the type at
all. Public fields on the inner sub-trees also pinned the layout;
moving to getters frees future internal restructuring.

## Tests

`cargo test -p burn-std`: 101 passed across unit, the
`tests/config.rs` integration suite, and doctests.
`cargo clippy -p burn-std -p burn -p burn-fusion -- -D warnings`
clean. `cargo fmt --check` clean. `cargo build -p burn-std
--no-default-features` clean for the no-std path.

`tests/config.rs` already exercised the public surface; the test
file updates (+9 / -9) move it onto the new getters.

## Review

- **laggui** (MEMBER) empty-body APPROVED at 2026-05-15T18:32:27Z
  on commit `2cbb071`. Merged into `main` at 18:32:45Z. Eighteen
  seconds. About eight hours after PR open.

## Lesson

No durable lesson surfaced.
