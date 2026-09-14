# ositech-packages

Shared home for pub.dev packages that Ositech's Flutter apps depend on
but can't consume unmodified from pub.dev -- each one has a small local
patch (or is pinned as-is with no upstream fix in sight) to work around a
build-breaking bug in the published release. One directory per package;
each is a normal, self-contained pub package (its own `pubspec.yaml`,
`LICENSE`, source, etc.) at the repo root.

## Why this repo exists

Before this repo, each app vendored its own copy of these packages under
its own `third_party/` directory. That meant the same patch (e.g.
removing a `jcenter()` call that no longer resolves under modern Gradle)
had to be re-discovered and re-applied independently in every app that
happened to depend on the same broken package -- easy to fix in one app
and silently leave broken, or fixed differently, in the next.

Centralizing them here means:
- One patch, applied once, used by every app via a git dependency.
- One place to check "has this already been fixed/patched" before
  re-deriving the same fix from scratch in a new app.
- One place to bump a package's version and re-verify the patch still
  applies, instead of N copies drifting out of sync.

## What's here

| Package | Why it's here |
|---|---|
| [`file_picker`](file_picker/README-PATCH.md) | Upstream 11.0.2's `android/build.gradle` skips applying its own Kotlin plugin under AGP 9, which this project can't work around any other way. |
| [`video_thumbnail`](video_thumbnail/README-PATCH.md) | Upstream 0.5.6's `android/build.gradle` calls the removed `jcenter()` repository, which fails outright under modern Gradle. |
| [`open_filex`](open_filex/README-VENDORED.md) | Vendored alongside `video_thumbnail` so a future fix can be applied locally without waiting on upstream; no patch needed yet. |
| [`media_kit_libs_android_video`](media_kit_libs_android_video/README-PATCH.md) | Upstream 1.3.8 bundles the `default` libmpv flavor, which has no ProRes decoder; swapped to the `full` flavor jars (v1.1.11) so ProRes `.mov` files from the media server play. |
| [`media_kit_libs_ios_video`](media_kit_libs_ios_video/README-PATCH.md) | Upstream 1.1.4 bundles the `video-default` libmpv xcframeworks, which have no ProRes decoder; swapped to `video-full` of the same v0.6.0 release. |

Each package's own `README-PATCH.md` (or `README-VENDORED.md`) is the
source of truth for exactly what was changed, why, and what to check
before re-applying the patch to a newer upstream version.

## How an app consumes a package from here

Add it as a `git` `dependency_override` in the consuming app's
`pubspec.yaml`, pointing `path` at the package's subdirectory in this
repo:

```yaml
dependency_overrides:
  file_picker:
    git:
      url: https://github.com/Ositech-Communications-Inc/ositech-packages.git
      path: file_picker
      ref: <commit-sha>
  video_thumbnail:
    git:
      url: https://github.com/Ositech-Communications-Inc/ositech-packages.git
      path: video_thumbnail
      ref: <commit-sha>
  open_filex:
    git:
      url: https://github.com/Ositech-Communications-Inc/ositech-packages.git
      path: open_filex
      ref: <commit-sha>
```

**Always pin `ref` to a specific commit SHA (or a tag), never a branch
name.** This repo has no per-package versioning of its own -- a later
patch to `video_thumbnail` pushed to `main` must not silently change
what an app depending on `file_picker` resolves to. Bumping to pick up a
new patch is a deliberate, one-line `ref` change in the consuming app,
not something that happens automatically on the next `pub get`.

This is a **private** repo, so `flutter pub get`/`dart pub get` need
working `git` credentials for `github.com` on whatever machine (or CI
runner) resolves the dependency -- an SSH key or an HTTPS credential
helper (e.g. `gh auth git-credential`) with access to the
`Ositech-Communications-Inc` org, the same as cloning any other private
Ositech repo.

## Adding or patching a package here

1. Copy the package's pub.dev source into a new (or existing) top-level
   directory here, named after the package.
2. Make the minimal change needed -- prefer the smallest patch that
   fixes the actual build failure over a wholesale rewrite.
3. Add (or update) that package's own `README-PATCH.md` documenting
   *why* the patch exists, *what* changed, and what to check before
   re-applying it to a future upstream version -- follow the existing
   files in `file_picker/` and `video_thumbnail/` as the template.
4. Push to `main`, then update each consuming app's `pubspec.yaml` `ref`
   to the new commit SHA once it's confirmed working.
