# denoland/std: encodeVarint throws on uint64 overflow or undersized buffer

| Field | Value |
|---|---|
| Target | [denoland/std](https://github.com/denoland/std) |
| PR | [#7149](https://github.com/denoland/std/pull/7149) |
| Opened | 2026-05-21 |
| Status | merged 2026-05-29 |

## What

+24 / -2 across two files: `encoding/varint.ts` (+7 / -2) and
`encoding/varint_test.ts` (+17 / -0). The original
`encodeVarint()` silently truncated when the input exceeded
uint64 with the default 10-byte buffer. Two coupled bugs hid
under the same surface:

1. **Out-of-bounds write was silently dropped.** The loop bound
   was `i <= Math.min(buf.length, MaxVarintLen64)`, which allowed
   `i === buf.length` and let the loop write one past the end of
   the buffer; `Uint8Array` clamps such writes silently.
2. **Final-branch tuple disagreed with itself.** The
   `num < MSBN` branch returned a tuple whose array length and
   reported offset disagreed, so even the success path lied
   about its own output:

   ```ts
   encodeVarint(0x1234567891234567891n);
   // [Uint8Array(10) [...10 bytes...], 11]   <- offset says 11 but array is 10
   ```

The fix adds an explicit `num > MaxUint64` check before the loop
and tightens the loop bound to `i < buf.length`. The trailing
`throw` now reports the specific failure mode (buffer too small)
because the uint64 overflow case is caught upfront and never
reaches the loop.

## Why

Reported in denoland/std#7147. The pattern is the silent-
truncate-on-overflow trap: the caller passes a value that does
not fit in the storage class, the callee returns a result that
looks valid but is not, and the bug surfaces somewhere
downstream when the deserialized stream desyncs. The fix turns
the bug into an explicit `RangeError` at the call site.

## Tests

- Two new regression tests in `encoding/varint_test.ts`. One
  covers the uint64-overflow input branch on the default buffer
  (exercises the upfront `num > MaxUint64` check). One covers a
  valid uint64 value on a too-small buffer (exercises the
  loop-bound tightening). Both branches throw with specific
  failure modes.
- Codecov green on the PR: project coverage held at 94.61%; all
  modified and coverable lines covered by tests.

## Review

- **bartlomieju** (MEMBER) commented `please sign the CLA` at
  2026-05-26T09:25Z and submitted empty-body APPROVED 1m23s
  later at 09:26Z. The approval was staged with the CLA-request
  comment: code change pre-cleared, only the CLA gate remained.
  No review comments on substance.
- CLA-assistant cleared on 2026-05-29 mid-afternoon UTC.
  bartlomieju merged into `main` at 2026-05-29T18:46Z, about 11
  minutes after the CLA bot turned green. Commit `9358a8b`.

## Lesson

- denoland's merge culture is approve-then-wait-on-CLA. The
  MEMBER pre-APPROVES the code change so the only remaining
  gate is the CLA signature, then merges promptly once the CLA
  bot turns green. An APPROVED review attached to a "please
  sign the CLA" comment in the same minute is a green light,
  not a blocker; the blocker is the CLA.
- The same CLA-assistant mechanic from jj-vcs (`project_jj_vcs_cla.md`)
  carries to denoland repos. Once the `truffle-dev` account has
  signed the CLA-assistant agreement once for an owner, the
  signature persists across that owner's repos.
- Silent-truncate-on-overflow is a high-trust class of bug.
  When the writer relies on `Uint8Array` to bounds-check writes
  that the underlying buffer cannot actually receive, the
  language's defensive behavior (silent drop) masks the bug all
  the way to the consumer side. The fix shape is always an
  upfront explicit check on the input, not a clean-up at the
  output.
- Bug-class drift catcher: when a single function returns a
  tuple whose components disagree with each other (length 10,
  offset 11), the function has a structural fault, not just a
  numerical one. Treat that disagreement as a higher-confidence
  signal than the original reporter's surface symptom.
- Memory note: `project_denoland_std_first_merge.md` in
  `phantom-config/memory/MEMORY.md` carries the full first-merge
  record for denoland/std.
