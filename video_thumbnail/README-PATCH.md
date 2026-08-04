# Local patch: video_thumbnail 0.5.6

Published copy of upstream `video_thumbnail` 0.5.6, consumed by apps via
a `git` `dependency_override` pointed at this repo (see the root README
for the exact syntax). **Only** file changed
from the unmodified upstream package: `android/build.gradle`.

## Why this exists

Upstream 0.5.6's `android/build.gradle` calls `jcenter()` in both its
`buildscript` and `rootProject.allprojects` repository blocks. JCenter
shut down and Gradle removed the `jcenter()` repository method entirely
in modern Gradle releases -- this project's Gradle 9.1.0 doesn't have
that method at all, so evaluating this plugin's build script fails
immediately with `Could not find method jcenter()`. That failure aborts
the script before it ever reaches `apply plugin: 'com.android.library'`,
which cascades into a second, more confusing error ("'kotlin-android'
plugin requires one of the Android Gradle plugins") purely because the
plugin was never actually applied.

Package has no Kotlin source at all (`VideoThumbnailPlugin.java` is
plain Java) -- unlike `file_picker`'s patch, this isn't an AGP9/Kotlin
lifecycle issue, just the removed `jcenter()` call.

## The fix

- Replaced both `jcenter()` calls with `mavenCentral()`.
- Bumped the self-contained AGP classpath from `4.1.0` (predates Gradle
  9 entirely) to `8.1.0`, matching the version already proven to coexist
  with this project's Gradle 9 / AGP 9 setup in `open_filex`'s own
  (unpatched) `android/build.gradle`.
- Wrapped the `namespace` assignment in the same
  `if (project.android.hasProperty('namespace'))` guard `open_filex`
  uses, for consistency (harmless under this project's AGP 9, which
  requires `namespace` unconditionally either way).
- Bumped `compileSdkVersion` from `33` to `36`. Root-caused from a real
  `fvm flutter run` build failure the user reported on a physical Android
  device (`Pixel 7`, API 37): several of this package's own transitive
  androidx dependencies (`androidx.fragment:fragment:1.7.1`,
  `androidx.window:window:1.2.0`, `androidx.activity:activity:1.8.1`, and
  others) declare AAR metadata requiring `compileSdk >= 34`; Gradle's
  `checkDebugAarMetadata` task fails the build outright if the library
  compiling against them is still on `33`, regardless of what the app's
  own `compileSdk` is set to (that check is per-module, not just
  app-level). This specific bump has **not** been re-verified against a
  real build in this dev environment (still no Android SDK here) --
  please re-run `flutter run` on Android to confirm it resolves the
  reported failure.

## Maintenance

When bumping `video_thumbnail` to a newer version, check first whether
upstream has replaced `jcenter()` itself (the package looks unmaintained
-- no release addressing this as of 0.5.6) before re-applying this
patch to the new version's `android/build.gradle`.

## Verification status

Not verified against a real Android build in the environment this patch
was written in (no Android SDK available there). `flutter analyze`/
`flutter test` pass unaffected, since only a native Gradle file changed,
not any Dart source. **Please confirm `flutter run` on a real Android
device/emulator succeeds after this change.**
