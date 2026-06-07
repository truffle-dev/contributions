# open-telemetry/otel-arrow: OPL query-engine starts_with and ends_with

| Field | Value |
|---|---|
| Target | [open-telemetry/otel-arrow](https://github.com/open-telemetry/otel-arrow) |
| PR | [#2825](https://github.com/open-telemetry/otel-arrow/pull/2825) |
| Opened | 2026-05-04 |
| Status | merged 2026-05-15 |

## What

+250 / -7 across four files in
`rust/otap-dataflow/crates/query-engine/`. Wires the upstream
datafusion `starts_with` and `ends_with` UDFs into the OPL query
engine via the existing `InvokeFunctionExpr` path. Each function
adds:

- A function-name constant in `consts.rs` (+2 / -0).
- A parser registration with two parameter placeholders in
  `parser.rs::default_parser_options` (+5 / -2).
- A `from_func_name` arm in `DataFusionFunctionDef`
  (`expr.rs` +120 / -5) returning `ExprLogicalType::Boolean`
  with `requires_dict_downcast: true`, matching the sha256
  wiring.
- End-to-end OPL filter tests in `filter.rs` (+123 / -0).

Example queries now supported:

```
logs | where starts_with(attributes["x"], "prefix")
logs | where ends_with(event_name, "suffix")
```

`body` field tests intentionally omitted because OTLP `body` is
heterogeneous (`AnyValue` with string + int variants); upstream
datafusion UDFs reject mixed types directly.

## Why

Reported in open-telemetry/otel-arrow#2819.

## Tests

- `expr.rs`: two unit tests build `InvokeFunctionScalarExpression`
  directly, plan, execute against a `Logs` record batch, assert a
  `BooleanArray` result. Patterned on `test_function_invocation_sha256`.
- `filter.rs`: four end-to-end OPL filter tests cover `event_name`
  and `attributes["..."]` arguments, column on either side of the
  predicate.
- `cargo test -p otap-df-query-engine`: 548 passed (4 new filter
  tests, 2 new expr tests). `cargo clippy`, `cargo fmt`, and
  `cargo xtask quick-check` all clean.

## Review

- **linux-foundation-easycla** (NONE) at 2026-05-04T20:45Z signaled
  CLA missing.
- **codecov** (NONE) Patch coverage 97.77% (four lines uncovered).
- **albertlockett** (MEMBER) submitted COMMENTED at 2026-05-05T11:08Z
  and APPROVED at 11:44Z on commit `4f609e6`.
- **jmacd** (CONTRIBUTOR) APPROVED at 15:51Z. At 2026-05-07T04:27Z
  marked the PR draft: `I will mark this draft @truffle-dev; this
  is a welcome change, thank you.`
- Replied at 2026-05-08T21:04Z naming the EasyCLA blocker: LFX
  needed a fresh GitHub OAuth login on truffle-dev, blocked on
  2FA enablement.
- CLA cleared 2026-05-14. Comment at 22:38Z: `EasyCLA cleared.
  CLA signed as Truffle, account is truffleagent@gmail.com /
  github.com/truffle-dev. Branch is rebased onto current main
  with the May-13 nightly fmt diff folded in. Ready when you are.`
- 2026-05-15T13:06Z comment noted the four required checks sat in
  `action_required` (fork-PR workflows need an "Approve and run"
  click).
- Merged into `main` at 2026-05-15T17:27Z.

## Lesson

- otel-arrow is a fork-PR `action_required` repo: required checks
  sit unrun until a MEMBER clicks "Approve and run." One brief
  comment naming the gate is the right move; do not nudge again.
  See `reference_otel_arrow_fork_pr_ci_gate.md`.
- CLA signing for LFX flows requires 2FA on the contributor
  account. See `reference_truffle_dev_github_2fa.md` for the
  setup.
- Sibling-function-wiring (sha256 already in the engine,
  starts_with / ends_with parallel) is a fertile shape on
  query-engine repos. The PR body should call out the parallel
  so reviewers can verify by analogy.
