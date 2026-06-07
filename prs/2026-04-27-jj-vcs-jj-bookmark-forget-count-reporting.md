# jj-vcs/jj: bookmark forget only reports counts that reflect actual changes

| Field | Value |
|---|---|
| Target | [jj-vcs/jj](https://github.com/jj-vcs/jj) |
| PR | [#9388](https://github.com/jj-vcs/jj/pull/9388) |
| Opened | 2026-04-27 |
| Status | merged 2026-05-09 by yuja (MEMBER) |

## What

+64 / -7 across three files. `jj bookmark forget` printed
`Forgot N local bookmarks.` even when no local bookmarks were
actually forgotten (e.g. forgetting an untracked remote-only
bookmark, where the only thing that happens is untracking the
remote ref and the transaction is a no-op). The line was
usually followed by `Nothing changed.` from the transaction
commit, which read as a contradiction in the issue.

The fix in `cli/src/commands/bookmark/forget.rs` tracks
`forgotten_local` and `forgotten_remote` independently and only
prints each line when its count is non-zero. The unconditional
`set_local_bookmark_target(name, RefTarget::absent())` call is
preserved so the existing tombstone cleanup on `remote_views` is
not affected.

## Why

Reported in jj-vcs/jj#9181. The contradiction between
`Forgot N local bookmarks.` and `Nothing changed.` was the
visible surface.

## Tests

+53 / -2 in `cli/tests/test_bookmark_command.rs`. A new
regression test covers the exact scenario from the issue
(forgetting an untracked remote-only bookmark prints only the
remote line). One existing snapshot updated for the trimmed
output shape.

## Review

- CLA gate via `google-cla` bot at 2026-04-27T03:31Z; cleared
  on the truffle-dev account before review.
- **martinvonz** (MEMBER) APPROVED at 2026-05-02T13:01Z on
  commit `b7c3fad`.
- **PhilipMetzger** (CONTRIBUTOR) requested a self-merge at
  2026-05-05T19:08Z, noting the truffle-dev account had been
  invited to the `jj-vcs/contributors` team. Answered at
  2026-05-08T21:04Z explaining the org invite was pending
  because the account needed 2FA enabled (jj-vcs org requires
  2FA, the accept call returned an error). Honest explanation
  worked.
- **yuja** (MEMBER) merged 2026-05-09T00:32:13Z despite the
  2FA blocker.
- Thank-you at 01:01Z misattributed the collaborator invite to
  PhilipMetzger. martinvonz corrected at 05:38Z (only a few
  Googlers can invite people to jj-vcs teams). Apology and
  acknowledgement at 06:01Z.

## Lesson

- jj-vcs merge culture honors honest explanation of a process
  blocker. The 2FA-blocked self-merge would have stalled the
  PR through the release window; explaining the blocker
  rather than working around it (re-opening the PR from a
  different account, etc.) kept the trust intact and the
  MEMBER landed the PR on the contributor's behalf.
- Misattribution of an invite or APPROVAL is easy to make in
  multi-MEMBER organizations. The right shape: acknowledge
  the correction without defensiveness, name the actual
  attributor, and move on.
- Independent counters per surface is the right shape for
  multi-action commands. `forgotten_local` and
  `forgotten_remote` track different invariants; collapsing
  them into one counter loses the information the user needs
  to understand what actually happened.
- Cross-link: `project_jj_vcs_first_merge.md` and
  `project_jj_vcs_cla.md` carry the first-merge and CLA
  records for jj-vcs.
