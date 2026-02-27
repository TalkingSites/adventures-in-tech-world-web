---
title: "Devlog #5: The Video Bug Diaries"
description: "Ten PRs. Three platforms. One week. A detailed look at the strangest bugs we hit piping live webcam frames into a game engine — with real code and real fixes."
postDate: 2026-01-31
author: Nick & the Adventures in Tech meetup
---

## Why a whole post about bugs?

Because the best devlogs tell you what went wrong, not just what went right. And the video capture system produced some of the most educational bugs we've ever seen. Each one taught us something fundamental about how Flutter, Dart, JavaScript interop, and native platforms actually work under the hood.

## Bug #1: The tree-shaker (PRs #71–72)

**Symptom:** Video bubbles work perfectly in debug mode. Deploy to production (release mode). Video bubbles show nothing. No errors in the console.

**Root cause:** Dart compiles to JavaScript for the web using `dart2js`. In release mode, it aggressively tree-shakes (removes) code it thinks is never called. Our JavaScript interop for capturing video frames used dynamic dispatch — we called JS functions through `dart:js_interop` at runtime. The compiler couldn't trace those call paths statically, so it deleted the bindings.

**The "aha" moment:** The code *existed in our source*, but the compiler decided it was dead code and removed it from the output. Debug mode doesn't tree-shake, so it worked fine there.

**The fix:** Add explicit static references to the interop functions so the compiler knows they're used:

```dart
// Force the compiler to keep these bindings alive
// by referencing them in a way that survives tree-shaking
void _ensureBindingsRetained() {
  // These calls are never executed at runtime, but their
  // existence prevents dart2js from removing the symbols
  assert(() {
    captureVideoFrame;  // reference the function
    return true;
  }());
}
```

**Lesson:** If your code works in debug but not release, suspect the tree-shaker. Especially for interop code.

## Bug #2: The phantom video element (PR #76)

**Symptom:** Remote participant video works for 5-10 seconds, then goes black. Local video is fine.

**Root cause:** LiveKit manages `<video>` HTML elements internally. When a track is briefly muted or the connection hiccups, LiveKit might dispose the old `<video>` element and create a new one. We were holding a reference to the old element and trying to capture frames from it — but it was already garbage collected.

```
Timeline:
  t=0s   LiveKit creates <video id="abc">
  t=0s   We grab a reference to <video id="abc">
  t=5s   LiveKit mutes/unmutes the track
  t=5s   LiveKit disposes <video id="abc">, creates <video id="def">
  t=5s   We're still reading from <video id="abc"> → black frames
```

**The fix:** Listen for LiveKit's `TrackSubscribedEvent` and `TrackUnsubscribedEvent`. When a track event fires, re-acquire the `<video>` element reference. Never cache it long-term.

**Lesson:** When you depend on another library's DOM elements, you're at the mercy of that library's lifecycle management. Subscribe to events, don't cache references.

## Bug #3: The iOS crash (PR #73)

**Symptom:** App crashes on iOS at startup. Works on macOS. Works on web.

**Root cause:** The FFI video capture was written for macOS. The native Objective-C code (`VideoFrameCapture.h/.m`) was compiled into the macOS runner but doesn't exist in the iOS runner. When Dart tried to look up the FFI symbols at startup, it crashed with a missing symbol error.

```
dyld: Symbol not found: _VideoFrameCapture_create
  Referenced from: tech_world.framework
```

**The fix:** Platform guard before any FFI call:

```dart
import 'dart:io' show Platform;

bool get _supportsFFICapture =>
    !kIsWeb && Platform.isMacOS;

ui.Image? captureFrame() {
    if (!_supportsFFICapture) return null;
    // ... FFI code only runs on macOS
}
```

**Lesson:** FFI code is inherently platform-specific. Guard it before the first symbol lookup, not after. The crash happens at *link time*, not at call time.

## Bug #4: The off-screen positioning (PR #80)

**Symptom:** On mobile web browsers, the `<video>` elements used for frame capture are visible — they appear as tiny rectangles at the top of the screen, overlapping the game.

**Root cause:** To capture frames from a `<video>` element, the element needs to be in the DOM and not `display: none` (otherwise the browser won't decode frames). We were using `opacity: 0` to hide them, but mobile browsers ignored that in some cases and showed them anyway.

**The fix:** Position them off-screen instead of invisible:

```dart
videoElement.style
  ..position = 'fixed'
  ..left = '-9999px'
  ..top = '-9999px'
  ..width = '1px'
  ..height = '1px';
```

Off-screen elements are still "visible" to the browser's video decoder (frames keep flowing) but are physically outside the viewport. Works on every browser.

**Lesson:** "Hidden" means different things to different platforms. `display: none` stops video decoding. `opacity: 0` isn't reliable on mobile. Off-screen positioning is the most portable hack.

## The timeline

Here's how the full debugging saga played out over two weeks:

```
Jan 17  ████████  PRs #35-38, #41, #43 — Initial impl + first bugs
Jan 18  ██        Stabilization
        ...
Jan 31  ████████  PRs #71-73 — Release mode + iOS crash
        ████      PRs #74-76 — Docs, CI, element lifecycle
Feb 3   ████████  PRs #77-80 — Final web + mobile fixes
```

Ten PRs. Three platforms. Every fix revealed something we didn't understand about the platform we were building on.

## What we learned

The meta-lesson: **cross-platform means cross-bugs.** Each platform has its own rendering pipeline, its own lifecycle rules, and its own gotchas. A single feature (video frame capture) required three completely different implementations:

- **macOS:** Objective-C + FFI (zero-copy, fast, fragile)
- **Web:** JavaScript interop + ImageBitmap (portable, quirky lifecycle)
- **Other:** Placeholder fallback (safe, boring)

And each implementation had its own unique failure modes. The only way through was testing on every platform, in both debug and release mode.

---

*Next: [Devlog #6: All-LiveKit](/posts/06-all-livekit/) — the day we deleted the WebSocket server and went fully serverless.*
