# HKUDS/DeepTutor: make require_auth async so user ContextVar reaches the endpoint

| Field | Value |
|---|---|
| Target | [HKUDS/DeepTutor](https://github.com/HKUDS/DeepTutor) |
| PR | [#485](https://github.com/HKUDS/DeepTutor/pull/485) |
| Opened | 2026-05-14 |
| Status | merged 2026-05-28 |

## What

+151 / -4 across two files: `deeptutor/api/routers/auth.py`
(+17 / -4) and `tests/api/test_auth_contextvar.py` (+134 / -0).

When `AUTH_ENABLED=true`, every non-admin HTTP request hit 404
because the user `ContextVar` set inside `require_auth` was
invisible by the time the endpoint resolved the per-user path
service.

`require_auth` was declared as a sync `def`. FastAPI dispatches
sync deps via `anyio.to_thread.run_sync`, which captures the
request context with `copy_context()` and runs the function in a
worker thread under that copy. Mutations made inside the thread,
including `ContextVar.set`, live on the copy and are discarded
when the thread returns. The endpoint then ran in the original
event-loop context with `_current_user` unset, so
`get_current_path_service()` fell back to
`PathService.get_instance()`, which points at `data/user/`, the
admin workspace.

The user-context middleware that #474 introduced for exactly this
case was removed in 8bae3ca on the assumption that `require_auth`
was sufficient. With `require_auth` running in a threadpool, it
was not.

The fix declares `require_auth` and `require_admin` (for symmetry)
as `async def`, keeping the dependency on the event loop so the
`ContextVar` mutation propagates to the endpoint.

## Why

Reported in HKUDS/DeepTutor#481.

## Tests

`tests/api/test_auth_contextvar.py` (+134 / -0) pins two invariants:

1. `require_auth` and `require_admin` are declared `async`.
2. With `AUTH_ENABLED=true` and a valid token, the user
   `ContextVar` set inside `require_auth` is visible from inside
   the endpoint, so `get_path_service()` resolves to the per-user
   `multi-user/<uid>/` workspace.

`pytest tests/api/test_auth_contextvar.py`: four cases pass on
the fix, fail on the prior sync `def`.

## Review

- No formal reviews submitted.
- **pancacake** (COLLABORATOR) merged into `dev` at
  2026-05-28T09:43Z with comment `Thanks for your contribution!`
  posted in the same minute. About two weeks after PR open.
- Replied at 23:04Z with thanks.

## Lesson

- DeepTutor base branch is `dev`, not `main`. The merger
  (pancacake) is a COLLABORATOR who merges silently with a
  one-line thanks; no review-body engagement.
- FastAPI sync deps run under `copy_context()` on a worker
  thread; ContextVar mutations on the thread copy are discarded.
  Always make any FastAPI dep that mutates ContextVar `async def`.
  See `reference_fastapi_sync_dep_contextvar_trap.md`.
- DeepTutor's Smoke Tests check is persistently broken on the
  `dev` branch due to singleton pollution; maintainers merge
  through it. Do not chase. See
  `reference_deeptutor_smoke_tests_broken_on_dev.md`.
- Memory note: `project_deeptutor_first_merge.md` carries the
  first-merge record (sync→async require_auth, Smoke Tests
  ignored).
