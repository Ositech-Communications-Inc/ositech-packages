# Vendored: open_filex 4.7.0

Published copy of upstream `open_filex` 4.7.0, consumed by apps via a
`git` `dependency_override` pointed at this repo (see the root README for
the exact syntax). **No files have been
changed from the unmodified upstream package.**

## Why this exists

Requested alongside the `video_thumbnail` vendoring/patch (see
`../video_thumbnail/README-PATCH.md`) so both packages are
available as local, directly-editable copies rather than pinned to a
pub.dev release that has no newer version addressing either of the two
issues below. Vendoring now means a future fix (to either package) can
be applied locally without waiting on upstream, the same way
`file_picker` and `video_thumbnail` already are.

Two known issues, neither of which required changing this package's own
code as of this vendoring:

1. **Android:** `android/build.gradle` already avoids `jcenter()` (uses
   `mavenCentral()`) and applies its own `com.android.tools.build:gradle:8.1.0`
   classpath directly, with no Kotlin plugin involved (this package is
   plain Java). No build failure for this package was reproduced in the
   error report that prompted this vendoring -- only `video_thumbnail`
   actually failed the Android Gradle build.

2. **iOS:** `flutter run` prints a (currently non-fatal) warning that
   this package "does not support Swift Package Manager for ios" --
   it still builds via CocoaPods today. This is expected to become a
   hard error in a future Flutter release per the warning text itself.
   No SPM `Package.swift` has been added to this vendored copy yet;
   revisit if/when Flutter actually starts failing builds over this
   instead of warning.

## Maintenance

If either issue above actually starts failing a real build, patch this
vendored copy directly (following the pattern in
`../video_thumbnail/README-PATCH.md` /
`../file_picker/README-PATCH.md`) rather than waiting for an
upstream release, and update this file to describe the change made.
