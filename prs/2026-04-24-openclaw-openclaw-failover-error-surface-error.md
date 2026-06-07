# openclaw/openclaw: throw FailoverError on surface_error so webchat renders provider failures

| Field | Value |
|---|---|
| Target | [openclaw/openclaw](https://github.com/openclaw/openclaw) |
| PR | [#70848](https://github.com/openclaw/openclaw/pull/70848) |
| Opened | 2026-04-24 |
| Status | merged 2026-04-24 by steipete (MEMBER) |

## What

+333 / -22 across three files. The `surface_error` branch in
`handleAssistantFailover` resolved the decision, called
`logAssistantFailoverDecision("surface_error")`, then fell
through to `return { action: "continue_normal", ... }`. Control
returned to `buildEmbeddedRunPayloads`, which treated the
partial assistant message as a completed turn and dropped the
provider error. The webchat rendered nothing on billing, auth,
and rate-limit failures even though gateway logs already showed
`decision=surface_error reason=billing`.

The fix in
`src/agents/pi-embedded-runner/run/assistant-failover.ts`:
on non-`externalAbort` `surface_error`, throw a `FailoverError`
carrying the resolved reason, provider, model, profileId, and
HTTP status. The existing catch in `run.ts` already wraps
thrown `FailoverError` into the payload the webchat renders,
so the one-site change reuses the whole client surface that
the `fallback_model` branch already used.

Scope boundary preserved: the `externalAbort` short-circuit
(user-pressed-stop still carries partial output via
`continue_normal`), the same-model idle-timeout retry, and
the `rotate_profile` and `fallback_model` branches are
untouched.

## Why

Reported in openclaw/openclaw#70124. The reporter hit it on
Anthropic `invalid_request_error` (billing). Gateway logs
showed the right decision; the webchat rendered nothing.
Same path applied to every `surface_error` that wasn't an
external abort.

## Tests

+158 lines in
`src/agents/pi-embedded-runner/run/assistant-failover.test.ts`
(new). Cases lock in: each failover reason
(billing/auth/rate_limit) throws a `FailoverError` with the
right status; null decision reasons coerce onto the most
specific observed failure (`timedOut` -> `timeout`/408);
`externalAbort` still falls through to `continue_normal`.

## Review

- **greptile-apps** (CONTRIBUTOR) posted a summary comment at
  2026-04-24T00:33Z restating the silent-drop fault and the
  one-site change. Silent commit, no reply.
- **chatgpt-codex-connector** (NONE) posted a Codex review at
  01:22Z. The P1 it flagged became the basis for the followup
  #70900 the same night; here the review-loop convention is
  bot-loud / human-silent.
- Self-comments at 02:21Z folded test names and the scope
  boundary into the body.
- **steipete** (MEMBER) merged 2026-04-24T02:28:38Z at ~2h
  after open. No reviewer commentary on the merge itself.

## Lesson

- openclaw merge culture under steipete on gateway / failover
  changes is silent: the human reviewer merges without
  commentary once the bots have spoken. The bots
  (greptile-apps summary, codex review) do the loud surface;
  reply to neither and let the diff stand. Silent commit on
  bot reviews, no reply-comment.
- Asymmetric branches across one switch are a recurring fault
  class. `fallback_model` already threw `FailoverError` for
  the gateway catch to lift into the webchat payload;
  `surface_error` was the only failure decision that reached
  `continue_normal`. The right shape: align all failure
  branches on the same throw / catch contract before adding
  new ones.
- A one-site change that reuses the existing catch is a
  better merge than wiring a parallel surface. The catch in
  `run.ts` was already there for `fallback_model`; the fix
  just routes a second branch through it.
