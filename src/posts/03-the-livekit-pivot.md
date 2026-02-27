---
title: "Devlog #3: The LiveKit Pivot"
description: "We already had LiveKit for video chat. Then we realised it could carry player positions, chat messages, and bot commands too. Why maintain a game server?"
postDate: 2025-12-22
author: Nick & the Adventures in Tech meetup
---

## Another long pause

After the multiplayer sprint in July 2024, the repo went quiet again until October 2025. Fifteen months. The meetup kept running, but Tech World sat waiting.

When development resumed, it resumed *fast*.

## The realisation

We had LiveKit for video chat. LiveKit has **data channels** — arbitrary binary messages that ride on the same connection as video and audio. They're reliable (TCP-backed) or unreliable (UDP-backed). They can be broadcast to all participants or targeted to specific ones.

The question became obvious: **why are we running a separate WebSocket server just for player positions?**

```
BEFORE (two servers):
┌─────────┐     ┌──────────────────┐
│ Flutter  │────▶│ WebSocket Server │──── Player positions
│ Client   │     │ (Compute Engine) │
│          │────▶│ LiveKit Cloud    │──── Video, audio
└─────────┘     └──────────────────┘

AFTER (one platform):
┌─────────┐     ┌──────────────────┐
│ Flutter  │────▶│ LiveKit Cloud    │──── Video, audio,
│ Client   │     │                  │     positions, chat,
└─────────┘     └──────────────────┘     bot messages
```

October 2025 was the transition period. The front end switched from the Compute Engine WebSocket backend to LiveKit for everything. The camera started following the player. Barrier walls got extended to feel more solid.

## The gather.town moment

In early January 2026, it all came together. PR after PR landed in rapid succession:

- **Simplified LiveKit UI** — Stripped down from a full video conference layout to just camera/mic toggle buttons. The game canvas became the primary view.
- **Game world enabled by default** — No more separate "connect to room" page. Sign in → you're in the game.
- **WASM build** — Flutter web running as WebAssembly for better performance.

The UI went from "video conference with a game attached" to "game world where you happen to have video chat":

```
┌──────────────────────────────────────────────────────┐
│ ┌──────────────────────────────────────────────┐ [🎤][📷] │
│ │                                              │     │
│ │           ╔══════════════════╗                │     │
│ │           ║  🏠 pixel art    ║                │     │
│ │           ║     room         ║                │     │
│ │           ║                  ║                │     │
│ │           ║    🧑 ← you      ║                │     │
│ │           ║           🧑      ║                │     │
│ │           ║   click to walk  ║                │     │
│ │           ╚══════════════════╝                │     │
│ │                                              │     │
│ └──────────────────────────────────────────────┘     │
│ Game canvas fills the screen. Camera follows player. │
└──────────────────────────────────────────────────────┘
```

## Meet Clawd

December 2025 also introduced **Clawd** — the AI tutor. A Node.js service running on GCP Compute Engine using the `@livekit/agents` framework. It joins the LiveKit room as a participant called `bot-claude` and listens for chat messages.

At this point Clawd was simple: a chat bubble that appeared when you got close, wired to Claude's API through LiveKit data channels. But having an AI participant in the same room as human players — using the same communication protocol — opened up everything that followed.

## Social sign-in and player bubbles

**PR #30** added Google and Apple sign-in alongside email/password. **PR #31** added local player bubble visibility — when you walk near another player, your own video bubble appears too (so you know you're "in range"). **PR #32** added a proper loading progress indicator for the web build, which had been showing a blank white screen during the WASM download.

## The CI/CD sprint

With real features landing, we needed quality gates. In a single day:

- Testing pipeline with `flutter test --coverage`
- Static analysis with `flutter analyze --fatal-infos` (zero warnings tolerated)
- Pre-commit hooks running the analyzer
- GitHub Actions for automated builds and deploy
- Coverage exclusions for platform-specific FFI code
- Docs-only change detection (skip tests for `.md` updates)

This infrastructure has caught dozens of issues since. The `--fatal-infos` rule is strict — even an unused import blocks the commit — but it keeps the codebase clean.

## What we learned

1. **Your communication platform might be your game server.** LiveKit was already handling the hard part (real-time, low-latency, cross-platform). Adding game data to it was trivial compared to maintaining a custom WebSocket server.

2. **The UI transition matters.** Going from "video conference with game" to "game with video" was mostly a layout change, but it completely transformed how the app felt. The game canvas being primary made it feel like a *place*.

3. **CI/CD early, CI/CD strict.** Adding `--fatal-infos` after 30+ PRs meant fixing a backlog of warnings. Adding it earlier would have prevented them from accumulating.

---

*Next: [Devlog #4: Proximity Video Bubbles](/posts/04-proximity-video-bubbles/) — walk near someone and their webcam feed appears as a floating circle in the game world.*
