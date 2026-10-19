# MUGEN-MOBILE Engine Working Copy — Build & Workflow Guide

## Source selection

This working copy is based on the uploaded official IKEMEN-GO `develop` source archive, not the older `release/1.0` archive. The two uploaded ZIPs did not include Git history, so an exact upstream commit cannot be proven from archive contents alone. The file contents themselves show that `develop` is the newer build base: `go.mod` declares Go 1.27.0, while `release/1.0` declares Go 1.20; the Android Docker recipe in `develop` uses Go 1.27.1, Android NDK r27d, SDK platform 34, and build-tools 34.0.0.

## What was deliberately preserved

- All engine Go source files, platform-specific files, shaders, data and font assets.
- `go.mod` and `go.sum`, the license, README, contribution guide, Makefile and `.gitignore`.
- `build/build.sh`, `build/build_android.sh`, the Android Dockerfile and Compose recipe. These are the official build path and are not disposable workflow clutter.
- `.github/dependabot.yml`: it proposes GitHub Actions and Go module updates but does not merge them automatically. Review and test any proposals before merging.
- `.github/release.yml`: passive release-note category configuration. It does not run a workflow by itself and may be reviewed later if release publishing is added.
- Issue/discussion templates, with feature/help prompts redirected away from the upstream project's issue tracker.

## What was removed from active automation

All eight inherited files under `.github/workflows/` were reviewed and removed from the active workflows directory. Their behaviors were not silently carried over:

- `releases.yml`: triggers on pushes to `develop` and `ci-test`, builds several platforms, uploads artifacts, moves rolling tags, and publishes GitHub Releases. It has `contents: write` and requires the original `IKEMEN_TOKEN`; unsuitable for an initial mobile-library test.
- `lint-code-style.yml`: runs auto-fixing lint tools and can commit changes back using `IKEMEN_TOKEN`; not safe to inherit as-is.
- `update-licence.yml`: writes `LICENCE.txt` and pushes to the hard-coded `develop` branch using `IKEMEN_TOKEN`.
- `update-script-wiki.yml`: writes and pushes a page to the repository Wiki and may use `WIKI_PUSH_TOKEN`.
- `conventional-label.yml` and `conventional-title.yml`: PR classification/enforcement; not required for compiling the engine.
- `repro-needed-stale.yml`: scheduled issue moderation that labels and closes issues; not a build requirement.
- `wiki-docs.yml`: PR comment/check gate for the upstream maintainers' documentation policy; not needed for our first build.

The four helper scripts under `.github/scripts/` were removed because each was referenced only by one of the removed workflows/release jobs: release notes generation, AltStore metadata generation, Lua Wiki generation, and contributor-list updating. The original upstream archive remains the source of record if one of these capabilities is deliberately reintroduced in a future phase.

## The one active workflow

`.github/workflows/android-native-build.yml` is manual-only (`workflow_dispatch`), uses read-only repository permissions, does not use secrets, does not publish releases, and does not modify source files. It runs the repository's official Docker-based Android build with `BUILD_ANDROID_APK=0`, checks `bin/libmain.so` and `bin/libmain.h`, verifies AArch64 architecture, collects dependency `.so` files, writes SHA-256 checksums, and uploads an artifact for 14 days.

To run it: open **Actions** → **Android Native Libraries (Test Build)** → **Run workflow**. GitHub only exposes a manual-dispatch workflow when its YAML exists on the repository's **default branch**. Put this workflow on the default branch before expecting the manual button to appear; then choose the desired source branch in the run dialog.

## How the upstream release workflow publishes releases (reference, not active here)

The upstream `releases.yml` is a multi-stage workflow:

1. `workflow_dispatch` accepts a version tag, draft flag, discussion category, Android-APK option and screenpack branch; push events on upstream development/test branches also trigger it.
2. A `tag` job validates semver tags and determines whether the run is nightly, a temporary branch build, CI test or a tagged release.
3. A matrix `build` job creates platform artifacts, including Android via Docker Compose, and uploads each platform's files with `actions/upload-artifact`.
4. The `release` job downloads platform artifacts, generates checksums and release notes, and stages metadata.
5. For nightly/CI-test runs it force-moves a rolling Git tag and deletes/recreates the GitHub Release object; the publishing step calls `ncipollo/release-action` with `IKEMEN_TOKEN`.

**Do not re-enable that workflow unchanged.** It assumes upstream branch names (`develop`, `ci-test`, `release/X.Y`), upstream release conventions, a dedicated write token, multi-platform build inputs and supporting scripts. When we are ready to publish our own release, create a separate manually triggered draft-release workflow with the exact artifacts we intend to publish, minimum `contents: write` permission, and no rolling-tag force-push unless explicitly required and reviewed.

## Safe development rules

- Keep the Android app repository separate from this engine source repository.
- Do not automatically copy or replace app `.so` files from an artifact.
- First inspect the artifact, ABI, file hashes and dependency list; then integrate it in a separate app branch and test on the device.
- Do not add secrets to the repository or print tokens in logs.
- Keep upstream license notices and copyright information intact.
