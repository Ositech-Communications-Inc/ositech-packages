# Local patch: media_kit_libs_android_video 1.3.8

Published copy of upstream `media_kit_libs_android_video` 1.3.8, consumed by
apps via a `git` `dependency_override` pointed at this repo (see the root
README for the exact syntax). **Only** file changed from the unmodified
upstream package: `android/build.gradle`.

## Why this exists

Upstream 1.3.8 downloads the `default` flavor of
[libmpv-android-video-build](https://github.com/media-kit/libmpv-android-video-build)
v1.1.7. That flavor is a whitelist build of ffmpeg
(`buildscripts/flavors/default.sh`): it has the `mov` demuxer and h264,
hevc, mpeg4, mjpeg, vp8, vp9, av1, aac, alac, pcm, ac3, eac3, mp3, opus,
flac and vorbis decoders, but **no ProRes decoder**. A ProRes `.mov` on the
Explorer II media server (Routica ROUT-8663 follow-up) therefore fails in
the app with

```
[mpv error vd] Failed to initialize a decoder for codec 'prores'.
[mpv fatal cplayer] No video or audio streams selected.
```

The `full` flavor (`buildscripts/flavors/full.sh`) configures ffmpeg with a
blanket `--enable-decoders`, which includes ProRes. It is still LGPL (no
GPL encoders; those are the separate `encoders-gpl` flavor).

## The fix

- The four `filesToDownload` entries now point at
  `releases/download/v1.1.11/full-<abi>.jar` instead of
  `v1.1.7/default-<abi>.jar`, with the MD5 of each downloaded jar
  (computed locally, sizes match the GitHub release metadata):

  | ABI | jar | size |
  |---|---|---|
  | arm64-v8a | `86f2c8faeb66af1878b3a16f67831cb3` | 7,835,755 B (was 5,729,978) |
  | armeabi-v7a | `93542d40e44f3afad3aa773674e8eaa5` | 7,553,542 B (was 5,499,353) |
  | x86_64 | `44b7efdbaf3626d6b24afaf5d497a369` | 8,668,739 B (was 6,516,632) |
  | x86 | `e69a9bbd7fb587deb33dd10fecc2fecc` | 7,940,960 B (was 5,839,220) |

  About +2.1 MB per ABI.
- `v1.1.11` because v1.1.7 (the tag upstream pins) publishes **only**
  `default-*.jar`; `full-*.jar` exists for v1.1.6 and v1.1.8 to v1.1.11.
  Checked with
  `gh api repos/media-kit/libmpv-android-video-build/releases --paginate`.

## Pairing caveat

Upstream `media_kit` 1.2.6 has only ever been released against the v1.1.7
jars. Pairing its FFI bindings with the v1.1.11 libmpv is untested
upstream. If the player fails to start or previously working formats
(mp4, mkv, avi, flac) regress, try the v1.1.6 `full-*.jar` before
abandoning the flavor swap.

## Maintenance

When bumping to a newer upstream `media_kit_libs_android_video`, check
which libmpv tag it pins and whether that tag ships `full-*.jar`
(`gh release view <tag> --repo media-kit/libmpv-android-video-build --json assets`).
Re-download and re-compute the MD5s; never copy them from another tag.

## Verification status

Not built in the environment this patch was written in (no Android SDK
there). `libmpv.so` inside `full-arm64-v8a.jar` was inspected and contains
the `prores` decoder symbols; the `default` build does not. **Please
confirm on a real Android device that the ProRes `.mov` plays and that
mp4 / mkv / avi / music still do.**
