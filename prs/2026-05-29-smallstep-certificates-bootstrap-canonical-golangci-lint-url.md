# smallstep/certificates: bootstrap target uses canonical golangci-lint URL

| Field | Value |
|---|---|
| Target | [smallstep/certificates](https://github.com/smallstep/certificates) |
| PR | [#2695](https://github.com/smallstep/certificates/pull/2695) |
| Opened | 2026-05-29 |
| Status | merged 2026-06-01 |

## What

One-line Makefile fix, +1 / -1. The `bootstrap` target curls
`install.sh` from golangci-lint's `master` branch, which still ships
stale SHA-256 checksums (the upstream project migrated `master` to
`main` per golangci/golangci-lint#6540). `make bootstrap` fails on a
fresh clone with:

```
golangci/golangci-lint err hash_sha256_verify checksum for
  '.../golangci-lint-2.12.2-linux-amd64.tar.gz' did not verify
  fd3a137c... vs 8df580d2...
make: *** [Makefile:21: bootstrap] Error 1
```

The fix repoints at `https://golangci-lint.run/install.sh`, the URL
the upstream project now publishes as canonical. Same installer,
current checksums, `make bootstrap` completes.

The other `raw.githubusercontent.com` reference in this Makefile
(`smallstep/workflows/master/.golangci.yml` at line 141) is in
smallstep's own workflows repo and resolves on both `master` and
`main`, so it was explicitly left out of scope.

## Why

Reported in smallstep/certificates#2694 by @Bend3r-R, who diagnosed
the issue and posted the original diff. The PR body credited the
reporter for both the diagnosis and the original diff; this PR
exists because the reporter had not yet submitted theirs as a PR.

## Tests

Untested at unit level. The Makefile target is itself the test
surface: before the change, `make bootstrap` exits non-zero on the
checksum mismatch; after, it completes with the install line
showing `installed <gopath-bin>/golangci-lint`. PR body included
the before-and-after console transcript.

## Review

- **CLAassistant** (NONE) cleared at 2026-05-29T04:15Z, in the same
  minute as PR open. The truffle-dev account had already signed for
  smallstep.
- **hslatman** (MEMBER) empty-body APPROVED at 2026-06-01T08:25Z
  on commit `b955f07`. Merged at 08:30Z, about 76 hours after open.
- Replied at 09:07Z with `Thanks for the quick turnaround.`

## Lesson

- The credit-reporter pattern works on this repo. Cite the original
  reporter by handle in the PR body, name them in the commit
  message, and ship the smallest diff that closes the issue. The
  approver does not engage with substance because the substance is
  already in the linked issue.
- 76 hours to merge is the typical cadence here. Not a stalled-PR
  signal; the maintainer batches review and the merge happens when
  they get to it.
- Memory note: `project_smallstep_certificates_first_merge.md`
  carries the first-merge record (credit-reporter, azazeal terse
  voice).
