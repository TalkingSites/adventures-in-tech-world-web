---
title: "Devlog #1: The 10x10 Grid"
description: "Red squares, blue lines, and a 640-pixel world. How Tech World started as a monorepo experiment with the tiniest possible game."
postDate: 2023-06-19
author: Nick & the Adventures in Tech meetup
---

## What it looked like

Imagine opening a web app and seeing this:

```
┌────────────────────────────────────────────────┐
│  ██████████████████████████████████████████████ │
│  ██  B ║    ║          ║                    ██ │
│  ██────║    ║          ║                    ██ │
│  ██    ║    ║          ║                    ██ │
│  ██    ║    ║          ║                    ██ │
│  ██    ║    ║    🧑     ║                   ██ │
│  ██    ║    ║          ║        ████        ██ │
│  ██    ║    ║          ║        ████        ██ │
│  ██    ║    ║          ║  ████████          ██ │
│  ██    ║    ║          ║                    ██ │
│  ██████████████████████████████████████████████ │
│                                                 │
│  ║ = blue grid lines     🧑 = 32x32 sprite     │
│  ████ = red barrier       B = "bald.png"       │
│  Background: solid #222222 (dark grey)          │
└────────────────────────────────────────────────┘
```

That's it. A 10x10 grid. 64 pixels per cell. 640x640 total. Red squares for walls. Blue lines for the grid. One tiny animated character from a free sprite sheet called `bald.png`.

It was beautiful.

## The backstory

Tech World was extracted from a monorepo belonging to the **Adventures in Dart, Flutter, Firebase** meetup group — a Melbourne-based group that had been building small projects together for years (slide puzzles, design patterns, coding exercises). The idea was ambitious: a shared virtual world where meetup members could hang out and learn together.

The first commit (`first commit, moved from monorepo`) landed on **June 19, 2023**. Here's the entire file tree that mattered:

```
lib/
  game/
    tech_world_game.dart    ← FlameGame subclass
    components/
      map_component.dart    ← grid + barriers + pathfinding
      player_component.dart ← animated sprite
    background/
      barriers.dart         ← hardcoded list of wall positions
  networking/
    services/
      networking_service.dart ← WebSocket to Cloud Run
assets/
  images/
    bald.png                ← your character
    beard.png               ← the other person
```

Two sprite sheets. One WebSocket server. One grid.

## How the rendering actually worked

The `MapComponent` drew everything by hand on a raw Canvas — no tile system, no sprite-based walls, just `canvas.drawRect()` calls:

```dart
// Barriers = red rectangles
for (final barrierRect in _barrierRects) {
  canvas.drawRect(barrierRect, Paint()..color = Colors.red);
}

// Grid = blue lines
for (double i = 0; i <= gridHeight; i += squareSize) {
  canvas.drawLine(Offset(0, i), Offset(gridWidth, i), _linePaint);
}

// Path = blue squares, destination = green square
canvas.drawRect(
  _pathLocations.last.toRect64(), Paint()..color = Colors.lightGreen);
```

Click anywhere and A\* pathfinding would calculate a route around the red squares, draw the path in blue, and (in theory) move your character along it. The movement was mostly commented out in this first commit — the path would appear but the character wouldn't actually walk it yet.

## The custom state management rabbit hole

Here's a fun detail: the app didn't use `setState`, `Provider`, or `Riverpod`. It used a custom state management framework called **Astro** — also built by the meetup group, also in the monorepo. It had its own concepts: "missions" (state changes), "percepts" (state selectors), even its own locator pattern.

```dart
// This is how you dispatched a state change in Astro
context.land(
  const StartChallenge(challengeType: ChallengeEnum.fixRepo),
);
```

Astro would eventually be replaced entirely. But it shows the meetup's philosophy: build everything from scratch to understand how it works. Even the state management.

## What we shipped (and what we didn't)

**Shipped:**
- A 10x10 grid with hardcoded barriers
- A sprite character that could face 4 directions
- A\* pathfinding (draw the path on click)
- WebSocket connection to a Cloud Run server
- A "challenges" system (UI for code exercises, wired up but incomplete)

**Didn't ship:**
- Actual movement along the path (commented out)
- Multiplayer (the WebSocket sent paths but didn't apply them)
- Any gameplay beyond walking around

Then the project went quiet for 8 months.

## What we learned

The most important thing this first version established: **Flame engine is the right choice.** It's Flutter-native, it handles sprites and animations well, and it composes with Flutter widgets (so we could overlay UI on top of the game later). That decision stuck through every rewrite that followed.

The second lesson: **custom frameworks are fun to build but expensive to maintain.** Astro, astro_types, ws_game_server_types — each one was a learning exercise, but each one was also a dependency that only the meetup group understood. Version 2 would replace all of them.

---

*Next: [Devlog #2: Multiplayer Madness](/posts/02-multiplayer-madness/) — adding a second player and discovering everything that breaks.*
