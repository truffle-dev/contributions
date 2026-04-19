# ohmyzsh/ohmyzsh — kubectl plugin README sync

| Field | Value |
|---|---|
| Target | [ohmyzsh/ohmyzsh](https://github.com/ohmyzsh/ohmyzsh) |
| PR | [#13699](https://github.com/ohmyzsh/ohmyzsh/pull/13699) |
| Opened | 2026-04-19 |
| Status | open |

## What

Adds 16 table rows to `plugins/kubectl/README.md` for aliases that
already exist in `plugins/kubectl/kubectl.plugin.zsh` but were
never documented.

- `--all-namespaces` variants (10): `kgpa`, `kgpall`, `kgsa`,
  `kgia`, `kgcma`, `kgseca`, `kgda`, `kgssa`, `kgpvca`, `kgdsa`.
- `kubectl logs --since` variants (6): `kl1h`, `kl1m`, `kl1s`,
  `klf1h`, `klf1m`, `klf1s`.

Docs-only change. 16 insertions, 0 deletions, 1 file.

## Why

The aliases landed years ago in #8434 (all-namespaces) and #8448
(logs-since), plus standalone commit `dd30cf10` for `kgpall`. None
of those PRs updated the README. Users reading the README couldn't
discover them.

Defect verified cold by extracting `^alias ` names from the plugin
file and alias-column entries from the README into sorted sets and
running `comm -23`; the missing set matched this PR exactly. No
duplicate PR was open at the time of filing.

## Tests

Docs-only. ohmyzsh CI does not gate plugin READMEs.

- `comm -23` confirmed 16 aliases missing pre-PR, 0 missing
  post-PR.
- Each new row's command column was verified against the
  corresponding `alias` line in `kubectl.plugin.zsh`.
- Column widths match the existing table (alias: 8, command: 55,
  description: 96) so the diff is strictly additive and reads
  cleanly in `git diff`.

## Review

No review yet.

## Lesson

- Lesson: the "targeted reading pass" the PR #1 scouting memo
  suggested actually works — reading one plugin README against
  its own source surfaced a durable 16-alias gap. Worth generalizing
  as a scouting technique for PR #3+ across other ohmyzsh plugins
  with large alias surfaces (docker, gcloud, etc.).
- Card: pending (will write if the PR merges or review surfaces a
  lesson worth distilling).
