---
title: "Devlog #4: Proximity Video Bubbles"
description: "The feature that makes Tech World feel like a place: walk near someone and their webcam appears as a circle floating above their character. Here's how we piped LiveKit video frames into Flame's render pipeline."
postDate: 2026-01-17
author: Nick & the Adventures in Tech meetup
---

## The idea

In Gather.town, you walk near someone and their video feed appears in a panel. Nice, but it feels like a UI overlay — the video is separate from the world.

We wanted something different: the video feed rendered **inside the game**, as a circular bubble floating above each character. It should participate in depth sorting (render behind walls when the character is behind a wall). It should fade in and out based on proximity.

The problem: there's no standard Flutter API for "give me raw video frame pixels from a LiveKit track."

## The proximity detection

The easy part came first. `ProximityService` checks every player pair using **Chebyshev distance** — the maximum of the horizontal and vertical grid distance. This accounts for diagonal proximity (you're "near" someone diagonally, not just cardinally).

```dart
// Chebyshev distance: max of |dx| and |dy|
// Threshold: 3 grid squares
// If you're within a 7×7 area centered on another player, you're "near"

  ┌───────────────────────────┐
  │ . . . . . . . . . . . .  │
  │ . . ┌─────────────┐ . .  │
  │ . . │ near  near  │ . .  │
  │ . . │ near 🧑 near │ . .  │
  │ . . │ near  near  │ . .  │
  │ . . └─────────────┘ . .  │
  │ . . . . . . . . . . . .  │
  └───────────────────────────┘
  3 squares in every direction = 7×7 detection zone
```

The service emits `ProximityEnter` and `ProximityExit` events on a stream. Other systems subscribe and react.

## The video frame pipeline

This was the hard part. The architecture on macOS:

```
LiveKit VideoTrack
    │
    ▼
Native RTCVideoRenderer (Objective-C)
    │  receives raw BGRA frames in a callback
    ▼
Shared memory buffer (no copy)
    │  pointer exposed to Dart
    ▼
Dart FFI reads pixel data
    │  ui.decodeImageFromPixels()
    ▼
dart:ui Image
    │
    ▼
Flame Canvas (drawImageRect with circular clip)
```

The key file: `macos/Runner/VideoFrameCapture.h/.m` — a native Objective-C class that registers as an `RTCVideoRenderer`. When LiveKit delivers a frame, it writes the pixel data to a buffer. Dart reads that buffer via FFI. No intermediate encoding, no PNG/JPEG conversion, no texture upload. Just raw pointer access to BGRA pixel data.

On web, it's different — we grab the HTML `<video>` element that LiveKit creates, call `createImageBitmap()` on it, and convert to `dart:ui Image`. Not zero-copy, but fast enough.

On platforms without either approach (iOS before we added FFI support), you get a placeholder: a coloured circle with the player's initial.

## The 10-PR debugging saga

**January 17, 2026** was the most intense day of the project. The initial FFI implementation landed as PR #35. Then reality hit. PRs #36 through #43 landed the same day, each fixing something the previous one broke:

| PR | What broke | The fix |
|----|-----------|---------|
| #35 | Initial FFI implementation | — |
| #36 | Test video bubbles left in code | Cleanup (embarrassing) |
| #37 | Local player video not appearing | `VideoBubbleComponent` was created before the video track was ready |
| #38 | Component won't mount | `isLoaded` getter override was preventing Flame's lifecycle |
| #41 | Pathfinding too slow on bigger maps | Replaced A\* with Jump Point Search (JPS) |
| #43 | Web video capture not working | Separate web implementation using `ImageBitmap` |

Then two weeks later, another wave:

| PR | What broke | The fix |
|----|-----------|---------|
| #71 | Web video broken in release mode | `dart2js` tree-shook our JS interop code as "unused" |
| #72 | Remote video broken in release | Same tree-shaking, different code path |
| #73 | iOS crash on launch | FFI symbols don't exist on iOS — needed platform guard |
| #76 | Video elements vanish | LiveKit disposed `<video>` elements we were reading from |
| #77 | Web remote video still broken | Race between element creation and frame capture |
| #80 | Mobile positioning wrong | Off-screen `<video>` elements needed explicit positioning |

The release-mode bugs (#71, #72) were particularly nasty. Dart's JS compiler (`dart2js`) aggressively removes code it thinks is unused. Our JavaScript interop bindings were only called dynamically at runtime, so the compiler couldn't see the references and deleted them. The fix: add explicit references that survive tree-shaking.

<!-- SCREENSHOT: Side-by-side of two players near each other with video bubbles visible above their characters. Ideally on the single_room.png background map. -->

## The result

Walk near another player. Their webcam smoothly fades in as a circular bubble floating above their character. The bubble is a real Flame component — it participates in depth sorting, so when a player walks behind a wall, their bubble renders behind the wall too. Walk away and it fades out.

At 30fps with 640x480 video frames, it processes ~36MB/s of pixel data with no perceptible lag beyond network delay.

This is the feature that makes Tech World feel like a *place* instead of a video call.

## What we learned

1. **Release mode is a different universe.** Debug builds worked perfectly. Release builds silently deleted our interop code. Always test release builds for platform-specific features.

2. **Zero-copy FFI is worth the pain.** The debugging took days, but the result is genuine zero-copy video frame access. No intermediate encoding, no copying pixel buffers. Just a pointer.

3. **Platform-specific code needs platform guards.** iOS doesn't have macOS FFI symbols. Web doesn't have FFI at all. Every platform needs its own code path and its own tests.

---

*Next: [Devlog #5: The Video Bug Diaries](/posts/05-video-bug-diaries/) — a deeper look at the weirdest bugs from the video capture implementation, with actual error messages and stack traces.*
