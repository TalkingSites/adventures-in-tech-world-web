---
title: "Devlog #2: Multiplayer Madness"
description: "Version 2 arrives with LiveKit, Firebase, and real multiplayer. Also: the stream subscription bug that haunted us for two weeks."
postDate: 2024-07-22
author: Nick & the Adventures in Tech meetup
---

## The eight-month gap

Between June 2023 and July 2024, the repo had exactly one meaningful commit. The meetup was running, but we were exploring other things. One lone commit in February 2024 — `Update to perception` — swapped the Astro state management framework for a new one called Perception. Same philosophy (custom-built, educational), different names (`percepts` instead of `missions`).

Then in July 2024, everything changed.

## Version 2: a total rewrite

PR #6 (`Move to version 2`) threw away nearly everything:

```
BEFORE                          AFTER
──────                          ─────
SDK >=2.18 <3.0                 SDK >=3.4.3 <4.0
Astro state management          → deleted
WebSocket to Cloud Run          LiveKit + Firebase Auth
10×10 grid, 64px cells          20×20 grid, 32px cells
2 sprite sheets                 3 sprite sheets (NPC11-13)
Custom monorepo deps            Standard pub packages
```

The big additions: **LiveKit** for real-time communication, **Firebase** for auth, and a proper game world with a background image.

Here's what it looked like now:

```
┌──────────────────────────────────────────────────────────┐
│  ┌─────────────────────┐  ┌────────────────────────────┐ │
│  │                     │  │ LiveKit Room               │ │
│  │   "Welcome to       │  │ ┌──────┐ ┌──────┐         │ │
│  │    Tech World"      │  │ │ 📹   │ │ 📹   │         │ │
│  │                     │  │ │You   │ │Them  │         │ │
│  │   (purple panel,    │  │ └──────┘ └──────┘         │ │
│  │    only on wide     │  │                            │ │
│  │    screens)         │  │ [Join Room] [Settings]     │ │
│  │                     │  │                            │ │
│  └─────────────────────┘  └────────────────────────────┘ │
│                                                           │
│  Layout: Row with Visibility(maxWidth >= 1200)            │
│  Left: welcome panel  |  Right: LiveKit connect page      │
└──────────────────────────────────────────────────────────┘
```

At this point the game canvas and the video conference were separate experiences. You'd join a LiveKit room and see video feeds in a traditional grid layout. The Flame game world existed but was wired in later.

## The multiplayer bugs

PR #8 (`Add multiplayer`) was the breakthrough — and the beginning of two weeks of bug hunting.

The concept: when your player moves, serialize the path and broadcast it via the `NetworkingService`. When you receive another player's path, create a `PlayerComponent` and move it along the path. Simple enough.

The reality:

**Bug #1: Missing player IDs (PR #10)**

Players would appear but we couldn't tell who was who. The user ID wasn't being set before the first position update arrived.

**Bug #2: Single-subscription streams (PR #12)**

This one was subtle and educational. Dart streams are single-subscription by default — only one listener allowed. Our `PlayerService` was exposing streams that multiple parts of the game needed to listen to (the game world, the UI, the networking layer). Second listener? Silent failure.

```dart
// BEFORE: Single subscription — only one listener works
final _controller = StreamController<PlayerPath>();

// AFTER: Broadcast — multiple listeners allowed
final _controller = StreamController<PlayerPath>.broadcast();
```

**Bug #3: Load order (PR #14, #16)**

The game world tried to subscribe to player streams in `onLoad()`, but the streams weren't ready yet. The `TechWorld` component needed to be created before `runApp()` was called, not during the widget build.

**Bug #4: Movement sync (PR #19)**

Players would teleport instead of walking smoothly. The path segments were being applied all at once instead of sequentially. Each `MoveEffect` needed an `onComplete` callback to trigger the next segment.

## The server shuffle

The backend went through three homes in two weeks:

1. **Cloud Run** — Quick to deploy, but the WebSocket connections kept dropping. Cloud Run's request timeout wasn't great for persistent connections.
2. **Compute Engine** — More control, stable WebSocket connections. We configured WSS with a custom domain (`adventures-in-tech.world`).
3. Eventually replaced entirely by LiveKit data channels (but that's [Devlog #6](/posts/06-all-livekit/)).

Each migration taught us something about connection lifecycle:

```
Cloud Run:   wss://websocket-server-cg5i52v35q-ts.a.run.app
Compute Engine: wss://adventures-in-tech.world
Production:     wss (port 443, custom domain with SSL)
Development:    ws://127.0.0.1:8080
```

## The result

By late July 2024: two people could open the web app, sign in with Firebase, join a LiveKit room for video chat, AND see each other's characters moving around a 20x20 pixel art room. The characters were still walking over solid-blue barrier rectangles, and the video chat was in a separate panel from the game, but the core was there.

The game world looked like this:

```
┌────────────────────────────────────────┐
│ ╔══════════════════════════════════╗   │
│ ║  🏠 single_room.png background  ║   │
│ ║                                  ║   │
│ ║   ████                           ║   │
│ ║   ████  ← blue barrier rects    ║   │
│ ║   ████    (hardcoded positions)  ║   │
│ ║   ████                           ║   │
│ ║         🧑 NPC11                 ║   │
│ ║                  🧑 NPC12        ║   │
│ ║              (32×64 sprites)     ║   │
│ ║                                  ║   │
│ ╚══════════════════════════════════╝   │
│ Grid: 20×20 cells, 32px each          │
│ Background: single_room.png           │
│ Barriers: blue rectangles (solid)     │
│ Players: NPC sprites with walk anims  │
└────────────────────────────────────────┘
```

## What we learned

1. **Broadcast streams aren't optional for multiplayer.** If N systems need to react to the same event (player position updates), you need `.broadcast()`. Single-subscription is a footgun.

2. **Initialisation order is the hardest bug.** In a system with Firebase, LiveKit, Flame, and networking all starting up concurrently, the order things become ready matters enormously. We'd revisit this lesson many times.

3. **Don't build your own state management.** Astro and Perception were great learning exercises, but v2 replaced them with simple `StreamController`s and a service locator. The simpler approach was easier to debug and easier for new meetup members to understand.

---

*Next: [Devlog #3: The LiveKit Pivot](/posts/03-the-livekit-pivot/) — the moment we realised our video chat platform could replace our game server.*
