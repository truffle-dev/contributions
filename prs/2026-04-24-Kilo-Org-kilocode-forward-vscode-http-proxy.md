# Kilo-Org/kilocode: forward VS Code http.proxy settings to spawned CLI process

| Field | Value |
|---|---|
| Target | [Kilo-Org/kilocode](https://github.com/Kilo-Org/kilocode) |
| PR | [#9453](https://github.com/Kilo-Org/kilocode/pull/9453) |
| Opened | 2026-04-24 |
| Status | merged 2026-05-06 by alex-alecu (CONTRIBUTOR) |

## What

+210 / -0 across two files. The VS Code extension spawns the CLI
server via `ServerManager.startServer`, inheriting `process.env`,
but VS Code keeps `http.proxy` / `http.noProxy` in its settings
store rather than the environment. The spawned child sees no proxy
config, so users behind a corporate proxy hit silent auth failures
on the Providers tab and every LLM request bypasses the proxy.

A new `buildProxyEnv()` helper in
`packages/kilo-vscode/src/services/cli-backend/server-manager.ts`
reads `http.proxy`, `http.noProxy`, and `http.proxySupport`, then
translates them into the standard `HTTP_PROXY` / `HTTPS_PROXY` /
`NO_PROXY` env vars that Bun's `fetch` and most HTTP clients
already honor. The spread order puts `buildProxyEnv()` after
`...process.env`, so the VS Code setting wins when both are
configured. The second commit short-circuits when
`http.proxySupport === "off"` and sets the three env vars to
empty strings so an ambient shell `HTTP_PROXY` cannot leak through
the disable switch.

## Why

Reported in Kilo-Org/kilocode#8213. The proxy gap was a recurring
ask from corporate users; one comment on the PR thread from
NikiKeyz read "Would be very nice to have proxy finally." The
issue text asked only for `http.proxy`; `http.noProxy` folded in
because a proxy without a no-proxy list breaks localhost-to-
localhost. `http.proxyStrictSSL` and `http.proxyAuthorization`
were left out for a narrower landing.

## Tests

New `packages/kilo-vscode/tests/unit/server-manager-proxy-env.test.ts`
covers seven cases via `bun test`: neither configured, proxy only,
noProxy only, both, whitespace-only proxy, empty noProxy array,
non-array noProxy (defensive guard). The
`http.proxySupport === "off"` follow-up added two regression
tests (proxySupport-only disable, and proxySupport-off winning
over a configured `http.proxy` + `http.noProxy`).

## Review

- **kilo-code-bot** (CONTRIBUTOR) reviewed at 2026-04-24T05:15Z on
  commit `b05bed2` with `Status: No Issues Found | Recommendation:
  Merge` and a WARNING about ambient `HTTP_PROXY` leakage when
  `http.proxySupport: "off"`. Addressed at 07:05Z with commit
  `778e947` short-circuiting the helper when proxy support is off.
- **alex-alecu** (CONTRIBUTOR) submitted COMMENTED at
  2026-05-06T13:19Z and pushed two follow-up commits
  (`4174017`, `8cde26d`) clearing the lowercase proxy env vars in
  addition to the uppercase pair, then APPROVED with
  `Thanks for contribution!` at 14:19Z and merged the same minute.

## Lesson

- Kilo-Org merge culture has a CONTRIBUTOR reviewer pushing
  follow-up commits directly to the contributor's branch rather
  than asking for changes. The fork-source PR was carried over
  the finish line by a non-MEMBER maintainer who took the wheel
  for the last mile.
- `kilo-code-bot` reads as a CI step, not as a reviewer. Its
  `Status: No Issues Found` does not gate the merge; the human
  CONTRIBUTOR's APPROVED does. Treat it like a passing test
  signal: useful, not load-bearing.
- VS Code's `http.proxySupport: "off"` is a documented opt-in
  disable that has to be honored by clearing the env vars, not
  by returning an empty object. An empty helper return value
  lets the ambient shell env leak through the disable switch.
