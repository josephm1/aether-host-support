# Aether native host support — review before upstream submission

This branch adds generic Android platform hooks for the separately installed
[Apps extension](https://github.com/josephm1/aether-apps). The editable applications
live in [aether-webapps](https://github.com/josephm1/aether-webapps). Neither the
extension nor any application source/data is bundled into this native change.
Upstream draft PR: [Zhou-Shilin/Aether#120](https://github.com/Zhou-Shilin/Aether/pull/120). It contains the implementation and API documentation only; this review note remains in the fork.

## Comparison and size

Based on current upstream `01b764a60634323403de89fbe3bb12467d833dee` (6 November 2026).
The desktop snapshot was based on `849696285d96657bb2f6e77229e795fa6822c2df`.
Only the local patch was carried forward with a three-way apply. The three newer
upstream commits, including summarizing-model and raw-thinking changes, remain.
A naive snapshot overwrite would incorrectly revert those changes.

| Scope | Before cleanup | After cleanup |
| --- | ---: | ---: |
| Native/bridge code and configuration files | 16 | 16 |
| Existing files touched | 8 | 8 |
| New generic platform files | 8 | 8 |
| Added lines | 1,516 | 1,438 |
| Removed lines | 63 | 62 |

That removes 79 changed lines (about 5%) from an already focused host patch.
API documentation adds another 40 lines in one existing file. This review document
is separate from those implementation counts. No web-app or extension payload
is included in the counts. The new classes are kept separate by responsibility;
combining files would change file counts without reducing maintenance work.

| Responsibility | Added lines | Removed lines | Why native support remains |
| --- | ---: | ---: | --- |
| WebView, isolated origins, document tasks, activity/controller lifecycle and UI binding | 751 | 26 | Android owns windows, WebView and launcher integration. |
| Shared document grants, scoped file access, picker/import/export and interrupted-write recovery | 431 | 0 | Android owns document URIs and persisted grants. Record formats remain in the web SDK. |
| Selected-model access, cancellation and structured AI context | 168 | 2 | Existing native provider configuration and OAuth refresh must stay private. App prompts/policies remain in the extension. |
| Generic native extension UI primitives and button variants | 83 | 34 | Apps uses screens/list rows and standard Material actions, without app-specific native screens. |
| Runtime initialization and WebKit dependency | 5 | 0 | Registers generic services and the origin-checked WebView implementation. |
| **Code/config total** | **1,438** | **62** | |

## Cleanup performed

- Removed the private-development `hosted` build variant. Standard upstream build
  types remain; the new APK has not replaced the installed test build.
- Removed the Gradle/Python bundling dependency on `web-app-projects/apps`.
  Install the independent extension ZIP through the existing extension manager.
- Removed the old MainActivity shortcut action/extras and its conversion route.
  One explicit HostedAppActivity URI route identifies document tasks.
- Removed temporary startup/capability/document timing instrumentation.
- Removed model-ID/label matching for old preference formats. Model choices use
  configured Aether keys and an explicit current selection.
- Removed pre-Android-26 shortcut branches: upstream already requires API 26.
- Standalone extension packaging no longer stages APK assets or packages examples.

Retained checks enforce current invariants: origin/ID/path bounds, immutable asset
versions and hash verification, cancellation, file revision checks and interrupted
writes. These are not legacy migrations. Browser content cannot obtain provider
authentication or native installation authority.

## Validation

- `:shared:testDebugUnitTest`: 180 tests, no failures/skips.
- `:app:testDebugUnitTest`: 342 tests, no failures/skips.
- `:app:assembleDebug`: passed after cleanup; no test APK was installed.
- Pi Bridge TypeScript check: passed; bridge tests: 55/55 passed.
- Apps extension syntax, storage, scoping, model preferences and guidance tests:
  11/11 passed; standalone extension packaging passed.
- StoryLab: 23/23 checks, managed build, ZIP import and unchanged-build reuse passed
  from the public checkout. All five public app editions built.
- `git diff --check`: passed for the native implementation patch.

The bridge suite was run with the developer's personal Pi directory masked in a
process-local mount namespace and `/dev` bound normally. An initial unisolated run
picked up an unrelated installed Pi extension and changed a transcript assertion;
no product change was made to conceal that environment effect. Provider tests use
fixtures/local servers; they did not establish live-provider or device acceptance.

## Gaps and further optimization decisions

**This branch is a review draft, not ready to claim cross-platform parity.** The
Android implementation existed in the supplied working build. iOS has no matching
WKWebView/document/model adapters. Linux cannot compile/install the iOS app.
No Swift/shared implementation is fabricated here. Native API definitions and
ownership should be agreed before an iOS port or upstream submission.

The rebuilt native APK has not been installed or exercised on a device in this
review. Earlier StoryLab device checks used the installed host, before this native
cleanup. Fresh import, cold shortcut launch, multiple windows, cancellation, file
pickers and model selection need regression checks against this new binary.
Existing tests do not fully cover the new WebView/document lifecycle.

Most remaining code implements platform I/O, recovery and lifecycle. Further large
reductions would require dropping capabilities or a broader shared-host design.
The selected-model facade still uses the existing Aether view model's selection
state; moving it into a shared platform-neutral controller is a possible next
refactor, but doing so solely to shorten this patch would expand its scope.

## Upstream contribution requirements checked

- [README contribution section](https://github.com/Zhou-Shilin/Aether#-contributing)
  welcomes issues and pull requests. It also invites an extension-list PR for new
  extensions; no extension-list PR has been opened.
- No separate CONTRIBUTING file, PR template or CLA/DCO instruction was found in
  the current repository tree. This is an observation at the pinned revision,
  not a guarantee about future maintainer requirements.
- [PR Check workflow](https://github.com/Zhou-Shilin/Aether/blob/01b764a60634323403de89fbe3bb12467d833dee/.github/workflows/pr-check.yml)
  requires affected-area Node/extension API checks, shared/Android unit tests and
  Debug APK build, plus an iPhone simulator build for matching paths. Preserve
  existing workflows and run the full required matrix before requesting review.
- [AGENTS.md](https://github.com/Zhou-Shilin/Aether/blob/01b764a60634323403de89fbe3bb12467d833dee/AGENTS.md)
  calls for Android/iOS consistency and verification of actual platform entry
  points. The iOS gap above must be addressed or explicitly agreed with maintainers.
- Preserve upstream GPL-3.0 licensing and third-party notices. The web-app repo
  separately retains LearnHarness's AGPL licence and excludes private medical
  reference libraries and personal data.

The user authorized submission after reviewing the breakdown. Draft PR #120 is
open for architecture feedback; it has not been merged. iOS and device regression
validation remain outstanding.
