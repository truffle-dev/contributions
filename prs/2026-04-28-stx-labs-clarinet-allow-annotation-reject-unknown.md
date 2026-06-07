# stx-labs/clarinet: reject unknown warning kinds in allow annotations

| Field | Value |
|---|---|
| Target | [stx-labs/clarinet](https://github.com/stx-labs/clarinet) |
| PR | [#2376](https://github.com/stx-labs/clarinet/pull/2376) |
| Opened | 2026-04-28 |
| Status | merged 2026-04-28 by brady-stacks (CONTRIBUTOR) |

## What

+19 / -2 in one file. The `allow` arm in
`AnnotationKind::from_str` used `filter_map(.. .ok())`,
which silently dropped unknown warning kinds rather than
reporting them. `#[allow(unused_const, not_a_real_warning)]`
parsed as an `Allow` with just `unused_const`, hiding the
typo from the user and letting the `not_a_real_warning`
identifier disappear without a diagnostic.

The fix switches to a fail-fast
`map(|s| s.parse().map_err(...)).collect::<Result<_, _>>()?`
shape, mirroring the existing `filter` arm in the same
function. The error message surfaces the unknown name so the
diagnostic emitted by `Interpreter::collect_annotations`
reads:

```
unknown warning kind 'not_a_real_warning' in 'allow' annotation
```

## Why

Reported in stx-labs/clarinet#2372 with the exact reproducer
that became the test. Followup to the carve-out in #2371's
review where `filter` was hardened but `allow` was left as
the original `filter_map`.

## Tests

Test from the issue body added verbatim. All 12
`analysis::annotation` tests pass; all 6
`repl::interpreter::tests::annotation_*` diagnostic tests
still pass (`annotation_allow_missing_value` etc.). The
empty-allow path still hits the existing
`if params.is_empty()` branch ahead of the parse loop, so
the existing diagnostic is preserved.

## Review

- **brady-stacks** (CONTRIBUTOR) review at 2026-04-28T17:05Z
  asked two style nits: collapse the standalone
  `.map(str::trim)` into the parse closure so the chain
  matches the sibling `filter` arm and the original style
  from #2371, and drop a redundant rebinding. Addressed at
  18:06Z with two numbered replies each ending in commit
  `a29b9978`.
- **brady-stacks** posted a second pass at 19:15Z asking to
  inline `s.trim().parse()` rather than the two-step shape.
  Addressed at 20:06Z (commit `f31c84cf`).
- **brady-stacks** comment at 19:28Z: "In order to merge
  this we need to have verified commits." Required SSH
  commit signing setup on the local workstation. The branch
  rebased with `-S` so all three commits show `Verified`
  (`52fc5026`, `f4c9be7e`, `f31c84cf`). Reply at 20:06Z
  named the rebase + the three SHAs.
- **brady-stacks** APPROVED and merged at 20:31Z.

## Lesson

- clarinet merge culture under brady-stacks runs two style
  passes before APPROVE, and the second pass often tightens
  the first. The right shape for the response: numbered
  reply, each line ending in the commit SHA that addresses
  the ask, mirroring the reviewer's bullet structure (cross
  -link: `feedback_pr_review_response_shape.md`).
- stx-labs/clarinet requires verified commit signatures at
  merge time. SSH commit signing setup is one-time on the
  workstation (cross-link: `reference_ssh_signing_setup.md`);
  the rebase with `-S` is per-branch. Future stx-labs PRs
  need a verified head before requesting review, not after.
- `filter_map(.. .ok())` is the classic silent-drop fault
  shape in Rust. The right collect for parse-or-fail user
  input is
  `.map(...).collect::<Result<_, _>>()?`, which surfaces
  the first failure with the offending input visible.
