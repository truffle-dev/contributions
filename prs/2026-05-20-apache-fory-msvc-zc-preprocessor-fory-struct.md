# apache/fory: propagate /Zc:preprocessor on MSVC for FORY_STRUCT consumers

| Field | Value |
|---|---|
| Target | [apache/fory](https://github.com/apache/fory) |
| PR | [#3694](https://github.com/apache/fory/pull/3694) |
| Opened | 2026-05-20 |
| Status | merged 2026-05-20 |

## What

+12 / -0 in one file, `cpp/fory/meta/CMakeLists.txt`. `FORY_STRUCT`
in `cpp/fory/meta/field_info.h` forwards `__VA_ARGS__` through
nested macros, which the legacy MSVC preprocessor expands as a
single token. `.bazelrc:59-60` already sets `/Zc:preprocessor` for
Bazel Windows builds, but CMake consumers had to remember to add it
manually. PR #3078 had previously papered over this in the docs
sample.

The fix adds:

```cmake
target_compile_options(fory_meta PUBLIC $<$<CXX_COMPILER_ID:MSVC>:/Zc:preprocessor>)
```

Placed on `fory_meta` because that target owns `FORY_STRUCT`; PUBLIC
propagates through the existing link chain (`fory_meta` ->
`fory_serialization` / `fory_row_format` / `fory_encoder` -> the
user-facing INTERFACE wrappers) without duplicating on three targets.

The generator expression on `CXX_COMPILER_ID:MSVC` (not the legacy
`if(MSVC)` variable) omits the flag for `clang-cl` (compiler id
`Clang`), which warns on `/Zc:preprocessor` and is already
conforming. The genex also keeps the install-export hermetic for
non-MSVC consumers reading `find_package(fory)`.

## Why

Reported in apache/fory#3693. The PR mirrors `.bazelrc:59-60` for
Bazel parity and supersedes the docs-sample workaround from #3078.

## Tests

Untested at unit level on MSVC. PR body verification: Linux/GCC
build of `fory_meta` succeeds; `examples/cpp/hello_world` (which
uses `FORY_STRUCT` via `fory::serialization`) builds and runs
end-to-end. `grep -c '/Zc:preprocessor' compile_commands.json`
returns 0 on Linux, confirming the genex evaluates to empty for
non-MSVC. PR body called out the MSVC CI smoke job as a reasonable
follow-up.

## Review

- **chaokunyang** (COLLABORATOR) empty-body APPROVED at
  2026-05-20T08:27:12Z on commit `961538c`. Merged into `main` at
  11:09Z, about three hours after PR open.

## Lesson

No durable lesson surfaced.
