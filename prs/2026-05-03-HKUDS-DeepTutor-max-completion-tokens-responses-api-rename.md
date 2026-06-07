# HKUDS/DeepTutor: map max_completion_tokens to max_output_tokens for the Responses API

| Field | Value |
|---|---|
| Target | [HKUDS/DeepTutor](https://github.com/HKUDS/DeepTutor) |
| PR | [#438](https://github.com/HKUDS/DeepTutor/pull/438) |
| Opened | 2026-05-03 |
| Status | merged 2026-05-03 |

## What

+86 / -5 across five files. On v1.3.5,
`get_token_limit_kwargs(model, n)` returns
`{"max_completion_tokens": n}` for newer OpenAI models
(gpt-5.x, o1, o3, o4) because that is the Chat Completions
name. `OpenAICompatProvider` and `AzureOpenAIProvider` both
route those kwargs through `client.responses.create()` when
`_should_use_responses_api()` matches, but the Responses API
only accepts `max_output_tokens`. The SDK rejects the unknown
kwarg with `TypeError` inside the client, before any HTTP
request leaves the process; the existing
`_should_fallback_from_responses_error()` only catches HTTP
errors with `status_code`, so the call is never retried via
Chat Completions.

The fix adds `adapt_chat_kwargs_to_responses()` to
`provider_core/openai_responses/converters.py` and uses it at
all four merge sites: `chat` and `chat_stream` on
`OpenAICompatProvider` (lines 698, 781) and
`AzureOpenAIProvider` (lines 126, 154). The helper preserves
the existing `None`-drop semantics of the previous
dict-comprehension merge, maps
`max_completion_tokens` to `max_output_tokens`, only applies
the alias when `max_output_tokens` is not explicitly set by
the caller, and does not mutate the input.

## Why

Reported in HKUDS/DeepTutor#437 by a user on v1.3.5 with
`LLM_BINDING = openai` and `LLM_MODEL = gpt-5.5`. An existing
open PR (#390) addresses a related problem at the factory
layer for Azure only by translating to `max_tokens`; that
PR's reproduction goes through Azure, while #437 goes through
`OpenAICompatProvider` and is outside #390's scope.

## Tests

`tests/services/llm/test_openai_responses_converters.py`
(+45): seven cases covering passthrough, `None`-drop, the
rename, `None` for the alias, explicit-name precedence,
empty input, and input non-mutation. `pytest
tests/services/llm/test_openai_responses_converters.py -v`
green; the rest of `tests/services/llm/` and
`tests/services/config/test_llm_probe_config.py` pass with
no new failures from this branch.

## Review

- No formal reviews.
- **pancacake** (COLLABORATOR) at 2026-05-03T03:58Z commented
  `Thanks for your contribution!`. Merged into `dev` at
  04:50Z, about 2.5 hours after PR open.

## Lesson

- DeepTutor base `dev`. Maintainer-comment-then-merge is a
  recognized DeepTutor convention; the comment is the
  acknowledgement and the merge is the verdict.
- Naming the related open PR (#390) explicitly in the body
  with the "happy to defer if a different shape is preferred"
  hedge prevents a duplicate-work close. When the related PR
  covers a sibling provider, frame the new PR as
  complementary rather than competing.
