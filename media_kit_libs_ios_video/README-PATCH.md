# Local patch: media_kit_libs_ios_video 1.1.4

Published copy of upstream `media_kit_libs_ios_video` 1.1.4, consumed by
apps via a `git` `dependency_override` pointed at this repo (see the root
README for the exact syntax). **Only** file changed from the unmodified
upstream package: `ios/Makefile`.

## Why this exists

Upstream 1.1.4's `ios/Makefile` downloads the `video-default` variant of
[libmpv-darwin-build](https://github.com/media-kit/libmpv-darwin-build)
v0.6.0. Its ffmpeg (`nix/packages/mk-pkg-ffmpeg/meson.build`) is a
whitelist build with **no ProRes decoder**, so a ProRes `.mov` on the
Explorer II media server (Routica ROUT-8663 follow-up) fails in the app
with

```
[mpv error vd] Failed to initialize a decoder for codec 'prores'.
[mpv fatal cplayer] No video or audio streams selected.
```

The `video-full` variant of the **same** v0.6.0 release configures ffmpeg
with a blanket `--enable-decoders` (ProRes included) and is still LGPL
(GPL encoders live in the separate `encodersgpl` variant).

## The fix

- Download URL: `..._ios-universal-video-default.tar.gz` →
  `..._ios-universal-video-full.tar.gz` (same `MPV_XCFRAMEWORKS_VERSION=v0.6.0`).
- `MPV_XCFRAMEWORKS_SHA256SUM`:
  `a95bc18508af26136b8a408341c05b5585d644ec013f00ac07db09d2e28d36ae` →
  `652047297624170bfd172ef25a99e49603c032d189a1761335edcb36db55b7ee`
  (computed locally; 26,703,782 B, was 19,817,325 B, sizes match the
  GitHub release metadata).

Same release tag, so there is no libmpv/media_kit pairing risk on iOS.

## Maintenance

When bumping to a newer upstream `media_kit_libs_ios_video`, read the new
`MPV_XCFRAMEWORKS_VERSION`, confirm that release ships
`ios-universal-video-full.tar.gz`
(`gh release view <tag> --repo media-kit/libmpv-darwin-build --json assets`),
download it and re-compute the SHA-256. Never reuse a checksum across tags.

## Verification status

Not built in the environment this patch was written in (no Xcode there).
The Makefile caches the downloaded tarball under the pod's
`.cache/xcframeworks/`; if a previous `default` build is still in
`ios/Pods/media_kit_libs_ios_video`, run `pod install` again (or
`flutter clean`) so the new URL and checksum are used. **Please confirm on
a real iPhone that the ProRes `.mov` plays and that mp4 / mkv / avi /
music still do.**
