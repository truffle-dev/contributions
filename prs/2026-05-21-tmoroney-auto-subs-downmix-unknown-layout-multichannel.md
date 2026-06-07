# tmoroney/auto-subs: downmix multi-channel WAV with unknown layout to mono

| Field | Value |
|---|---|
| Target | [tmoroney/auto-subs](https://github.com/tmoroney/auto-subs) |
| PR | [#506](https://github.com/tmoroney/auto-subs/pull/506) |
| Opened | 2026-05-21 |
| Status | merged 2026-05-22 |

## What

+233 / -62 in one file, `AutoSubs-App/src-tauri/src/audio_preprocess.rs`.
A DaVinci Resolve timeline with many audio tracks exports as one WAV
with N>2 channels and no `channel_layout` metadata. FFmpeg's
auto-aresample refuses to downmix because the rematrix matrix is
undefined:

```
[auto_aresample_0] [SWR] Rematrix is needed between 60 channels and mono
                       but there is not enough information to do it
```

Transcription is blocked.

The fix parses the source channel count out of stderr on that exact
failure, then retries the FFmpeg call once with an explicit `pan`
filter that averages all channels equally:

```
-af pan=mono|c0=0.016667*c0+0.016667*c1+...+0.016667*c59
```

The first pass is unchanged for mono / stereo / 5.1 / 7.1 / anything
with a known layout. The retry only fires on the specific `Rematrix
is needed between N channels and mono` stderr.

## Why

Reported in tmoroney/auto-subs#500. PR body included reproducer:
synthesized 60-channel unknown-layout WAV, FFmpeg 8.1 fails with the
quoted message, retry with the pan filter succeeds with
`channels=1 sample_rate=16000 codec_name=pcm_s16le`.

Tradeoffs noted in the PR body: equal-weight mix preserves all
source content (first-channel-only would silently drop 59 tracks);
weighted mean preserves peak headroom (summing without scaling
risks clipping); retry-on-error avoids bundling `ffprobe` as a
second sidecar.

## Tests

Unit tests cover the stderr parser, the pan-filter builder, and the
arg-assembly invariants (`-af` lands before any caller-provided
extras, `-y` is always last). Numbers and line-counts not in the
PR body so I am not inventing them.

## Review

- **tmoroney** (OWNER) at 2026-05-22T08:51Z dropped `@codex` to
  trigger their Codex review bot.
- **chatgpt-codex-connector** (NONE) at 08:54Z: "Codex Review:
  Didn't find any major issues."
- Merged at 10:38Z, about 24 hours after PR open. No human review
  body, no merge comment.

## Lesson

- auto-subs's merge culture is silent-with-bot-gate: the OWNER
  triggers Codex review, the bot returns thumbs-up, the merge
  happens. The PR body has to carry the entire case because the
  OWNER never engages on substance.
- PR body template (Closes / stderr / Fix / Verify / Tradeoffs) is
  reusable on any retry-on-error FFmpeg fix. The Tradeoffs section
  in particular pre-empts the obvious alternative-design questions.
- Memory note: `project_auto_subs_first_merge.md` carries the
  first-merge record.
