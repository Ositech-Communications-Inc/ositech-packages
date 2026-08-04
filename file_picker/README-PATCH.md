# Local patch: file_picker 11.0.2

Published copy of upstream `file_picker` 11.0.2, consumed by apps via a
`git` `dependency_override` pointed at this repo (see the root README for
the exact syntax). **Only** file changed
from the unmodified upstream package: `android/build.gradle`.

## Why this exists

Upstream 11.0.2's `android/build.gradle` only applies its own
`org.jetbrains.kotlin.android` plugin when the Android Gradle Plugin
version is below 9 (an `isAgp9OrAbove` check), assuming AGP 9's built-in
Kotlin compiler handles compilation instead. That assumption only holds
when `android.builtInKotlin=true` project-wide -- and this project can't
turn that on, because other plugins (e.g. `connectivity_plus`) have no
AGP9 handling of their own and hard-fail once built-in Kotlin is enabled.
Left as `android.builtInKotlin=false` (this project's actual setting),
file_picker's own Kotlin sources are never compiled by anyone, and the
Android build fails with `cannot find symbol: class FilePickerPlugin`.

Patching the plugin to apply the Kotlin plugin *from outside* (via a
Gradle lifecycle hook in the root `android/build.gradle.kts`) was tried
first and does not work: Kotlin Gradle Plugin 2.x tracks its own
internal application lifecycle separately from Gradle's project state,
and rejects being applied once a project has moved past a certain point
in evaluation (`gradle.projectsEvaluated` -> "ProjectState 'EXECUTED'";
a project-scoped `afterEvaluate` -> "ProjectState 'EXECUTING'"). It must
be applied synchronously as the plugin's own build script runs.

## The fix

Removed the `isAgp9OrAbove` conditional entirely, reverting to
file_picker's own pre-11.0.0 behavior: always apply
`org.jetbrains.kotlin.android` unconditionally. This uses the module's
own self-contained Kotlin Gradle Plugin classpath (pinned to `1.8.22` in
its own `buildscript` block) -- an old-style KGP version that predates
the strict lifecycle machinery above, applied entirely independently of
the root project's own (newer) Kotlin Gradle Plugin version. This is the
exact mechanism `connectivity_plus`, `image_picker_android`, and every
other native plugin in this project already rely on successfully.

## Maintenance

When bumping `file_picker` to a newer version, re-diff upstream's
`android/build.gradle` against this patched copy and re-apply the same
change (remove any AGP-version-conditional Kotlin plugin skip) rather
than reverting to the unmodified upstream file. Check first whether a
newer release has actually fixed its own AGP9 detection (e.g. checking
`android.builtInKotlin`/an equivalent Gradle property instead of just
the AGP major version) -- if so, this override and patch can be dropped
entirely.
