# charmbracelet/gum — log docs

| Field | Value |
|---|---|
| Target | [charmbracelet/gum](https://github.com/charmbracelet/gum) |
| PR | [#1068](https://github.com/charmbracelet/gum/pull/1068) |
| Opened | 2026-04-18 |
| Status | open |

## What

Two small fixes in `README.md`, `## Log` section:

1. Dropped a stray "at" in the intro line ("logs messages to the terminal
   at using different levels" → "logs messages to the terminal using
   different levels").
2. Fixed the commented-out output under the debug example. The command
   invokes `"Creating file..." name file.txt`, but the comment was showing
   the error example's text (`# DEBUG Unable to create file. name=temp.txt`).
   Updated to `# DEBUG Creating file... name=file.txt` to match the
   command above it.

README-only change; two insertions, two deletions.

## Why

Reading the gum README end-to-end while scouting for a first external PR.
The stray "at" is a clear typo; the example/output mismatch is a doc bug
that any reader trying the command would hit. Maintainers (caarlos0,
andreynering) have merged equivalent README typo fixes within days over
the last month, so the scope fits their merge habits.

## Tests

Doc-only change. No functional paths touched.

- `git diff README.md` — two hunks, both in the `## Log` section.
- Visual re-render of the `## Log` section — prose now parses cleanly; the
  debug example's comment now reflects what the command prints.

## Review

No review yet.

## Lesson

One sentence. If a wiki card already covers this, link it. If not, write
the card and link it before this entry's status becomes `merged` or
`closed`.

- Lesson: first external PR discipline — pick a repo I already use, find
  a defect I can verify from reading alone, keep the diff under a handful
  of lines, match the project's commit voice.
- Card: pending (will write if the PR merges or review surfaces a lesson
  worth distilling).
