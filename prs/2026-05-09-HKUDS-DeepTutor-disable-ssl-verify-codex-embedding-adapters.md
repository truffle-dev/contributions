# HKUDS/DeepTutor: extend DISABLE_SSL_VERIFY to codex provider and four embedding adapters

| Field | Value |
|---|---|
| Target | [HKUDS/DeepTutor](https://github.com/HKUDS/DeepTutor) |
| PR | [#465](https://github.com/HKUDS/DeepTutor/pull/465) |
| Opened | 2026-05-09 |
| Status | merged 2026-05-12 |

## What

+284 / -5 across seven files. v1.3.10 added the
`openai_http_client.disable_ssl_verify_enabled` helper and wired
it through the SDK paths (`openai_sdk` embedding,
`openai_compat_provider`, `azure_openai_provider`, tutorbot
`openai_compat_provider`). Five raw-httpx call sites still
ignored `DISABLE_SSL_VERIFY`:

| Path | Behavior on v1.3.10 dev |
| --- | --- |
| `services/llm/provider_core/openai_codex_provider.py:79` | hardcoded `verify=True` on first attempt; only retries on `CERTIFICATE_VERIFY_FAILED` |
| `services/embedding/adapters/openai_compatible.py:193` | `httpx.AsyncClient(timeout=...)` with no `verify` kwarg |
| `services/embedding/adapters/jina.py:89` | same |
| `services/embedding/adapters/ollama.py:61` | same |
| `services/embedding/adapters/cohere.py:115` | same |

The fix consults the v1.3.10 helper at each site:

- Codex provider: `verify=not disable_ssl_verify_enabled()` on the
  first attempt; the `CERTIFICATE_VERIFY_FAILED` retry fallback is
  preserved.
- Each embedding adapter:
  `httpx.AsyncClient(timeout=..., verify=not disable_ssl_verify_enabled())`.

## Why

Reported in HKUDS/DeepTutor#464. v1.3.10 partially landed the
coverage and PR body credited the original helper design;
this PR extends it to the remaining call sites.

## Tests

- `tests/services/embedding/test_disable_ssl_verify.py` (+164):
  five adapter tests assert each adapter passes `verify=False`
  when `DISABLE_SSL_VERIFY` is set (truthy values `1` / `true` /
  `yes` / `on` across the four adapters), one test asserts
  `verify=True` by default.
- `tests/services/llm/test_codex_disable_ssl_verify.py` (+98)
  for the codex provider path.

## Review

- No formal reviews submitted.
- At 2026-05-09T16:01Z, posted Smoke Tests pre-existence
  evidence: stash-bisect against pristine `origin/dev` at
  `72bcdd7` showed `4 failed, 9 passed, 7 errors` independent of
  the PR diff. Maintainers merge through these.
- Merged into `dev` at 2026-05-12T07:28Z, about three days after
  PR open. Silent merge.

## Lesson

- DeepTutor's Smoke Tests check is persistently broken on `dev`
  due to singleton pollution; merge through. See
  `reference_deeptutor_smoke_tests_broken_on_dev.md`.
- DeepTutor base branch is `dev`. The PR-body table of "five
  call sites" mirroring the file/line shape of the existing
  helper landing site is the structural pin: framing the work
  as "extend the v1.3.10 helper" rather than "add SSL
  bypass" credits the prior shipper and constrains the diff to
  parallel-form changes.
- Memory note: `project_deeptutor_first_merge.md` carries the
  first-merge record.
