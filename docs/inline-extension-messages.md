# Inline Pi extension messages

This branch builds on the existing web-app host proposal. It adds generic persisted
Pi custom-message delivery, rather than putting either visual extension into the app.
Interactive UI and Anatomy remain independently installable from
[aether-visuals](https://github.com/josephm1/aether-visuals).

## Native change breakdown

Compared with the existing host-support branch: 13 runtime source files,
108 added lines and three removed lines. Two test files add 30 lines. Documentation
is counted separately.

- **Pi bridge:** after a prompt settles, collect only newly committed display
  messages from the active branch and include them in the completion payload.
  This handles Pi's deferred `sendMessage(..., {triggerTurn:false})` behavior.
- **Shared Kotlin:** one small parser handles message identity, visible text and
  bounded JSON details. The shared client consumes the same completion payload.
- **Android:** persist custom messages in their originating session, expose tagged
  messages to the extension host, and render registered extension views inside both
  chat message layouts. Both existing Android host-context entry points are wired.
- **iOS:** append the same parsed messages to the originating session, using its
  existing extension-message renderer. No second visual renderer is added.
- **Tests:** a real Pi TypeScript extension queues a deferred display message against
  a scripted local provider; Android tests check tagged context and parsing bounds.

No Anatomy toolkit, generated HTML, provider credentials, sample-specific routing,
legacy migration or compatibility loader is added to native code. The generic view
renderer is reused. Document storage and controls belong to the extensions.

## Validation

On Linux, the Android hosted APK built successfully and all five focused
`ExtensionHostStateTest` tests passed. The bridge's deferred-message regression
passed against the real Pi SDK with a local scripted provider. The public branch
contains only the focused source patch; the APK was built from matching sources
in the local application workspace.

The extension workspace separately passed 12 core tests, eight browser interaction
checks and a packaged-host test loading both extensions together. Browser captures
and [manual prompts](https://github.com/josephm1/aether-visuals/blob/main/docs/TESTING.md)
state their actual environment. These do not establish native device correctness.
Final physical-device interaction is assigned to the user. iOS source was updated,
but Xcode compilation, signing and device verification require a Mac and remain
pending. No upstream PR for this feature has been opened or updated.

## Context refresh fix

Repeated Android `start(context)` calls now return before changing the remembered
context. Previously they overwrote it before `updateContext` compared the new
value, so new custom-message content could fail to refresh and show only its text
fallback. iOS already refreshes on conversation/message-count changes.
The Android hosted APK rebuilt and the five focused unit tests passed; final
physical verification of this fix is pending.
