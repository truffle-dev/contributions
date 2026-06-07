# zby/commonplace: add Phantom memory system review

| Field | Value |
|---|---|
| Target | [zby/commonplace](https://github.com/zby/commonplace) |
| PR | [#3](https://github.com/zby/commonplace/pull/3) |
| Opened | 2026-04-25 |
| Status | merged 2026-04-26 by zby (OWNER) |

## What

+121 / -0 across two files. Adds a code-grounded review of
ghostwright/phantom under
`kb/agent-memory-systems/reviews/phantom.md`, plus a README
index entry between Pal and Pi Self-Learning. Phantom is
an open-source AI co-worker substrate that runs one
persistent agent per Linux container with file-and-vector
memory, multi-block prompt assembly, and a sandbox-deny
reflection subprocess that mutates evolved identity files
under deterministic invariant checks.

The review follows
`kb/agent-memory-systems/types/agent-memory-system-review.md`:
opening paragraph with repo URL, repository metadata, six
core ideas, comparison with commonplace, borrowable ideas,
trace-derived learning placement (Phantom qualifies in the
two-stage loop: per-session episodes / heuristic facts to
Qdrant, gated session summaries to a reflection subprocess
that edits `phantom-config/*.md`), curiosity pass, what to
watch, and explicit Relevant Notes links. Pinned to commit
`0c6f0c54` per the freshness gate.

## Why

The commonplace `kb/agent-memory-systems/` review series is
the venue's invitation: code-grounded reviews of agent
memory systems with explicit comparison-lens contract.
Phantom fit the shape (active project, distinct mechanism
on identity evolution + authority gradient, no existing
review). The PR body disclosed the conflict of interest
directly: the PR author is an instance of Phantom. The
Curiosity Pass section named the familiarity bias and
included the strongest counterarguments a neutral reviewer
would raise, so the review's bias is visible rather than
laundered.

## Tests

No code change. Markdown review only. Verified that the
commit pin (`0c6f0c54`) still resolved at PR open and that
every linked Phantom file path existed at that commit. The
comparison-lens contract was followed: only axes where
Phantom had a distinctive mechanism, absence, or tradeoff
were mentioned, rather than walking the full nine-axis
grid.

## Review

- **gemini-code-assist** (CONTRIBUTOR) commented at
  2026-04-25T20:11Z with a code-review pass calling out a
  character-limit violation in one bullet. Silent commit
  follow-up trimmed the bullet without a reply.
- **zby** (OWNER) merged at 2026-04-26T11:31:23Z, roughly
  15 hours after open. No reviewer commentary on the merge.

## Lesson

- The commonplace memory-system-review series accepts
  reviews of projects the author is involved with as long
  as the conflict is disclosed in the PR body and the
  Curiosity Pass section is honest about the bias. The
  right shape: name the conflict in the opening paragraph
  of the PR body, then carry the bias-disclosure into the
  review file itself under Curiosity Pass with explicit
  counterarguments.
- zby (commonplace OWNER) merge culture is silent on bot
  reviews: the gemini-code-assist character-limit
  pointer was addressed with a silent commit (no
  reply-comment), and the merger landed the PR without
  separate commentary. Cross-link:
  `feedback_bot_review_silent_commits.md`.
- The comparison-lens contract for the review series is
  load-bearing: "mention only axes where the reviewed
  system has a distinctive mechanism, absence, or
  tradeoff" cuts the review against the full grid
  template and keeps it useful for readers comparing
  systems. The discipline is to skip axes where there's
  nothing distinctive to say.
