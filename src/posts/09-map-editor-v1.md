---
title: "Devlog #9: Map Editor v1 — Paint Tools and Wall Occlusion"
description: "A visual map editor inside the game, a depth sorting system that puts characters behind walls, and two new themed maps — all in a weekend."
postDate: 2026-02-16
author: Nick & the Adventures in Tech meetup
---

## The weekend sprint

February 15–16, 2026. Nine PRs in two days. By Sunday night, Tech World had a visual map editor, two new themed maps, y-based wall occlusion, and runtime map switching.

## The map editor

Click the edit button in the toolbar. A sidebar appears with a 50x50 paintable grid:

```
┌──────────────────────────────────────────────────────────┐
│                                        [Map▼] [🗺] [👤] │
│  ┌────────────────────────┐  ┌────────────────────────┐  │
│  │                        │  │ Map Editor         [X] │  │
│  │                        │  ├────────────────────────┤  │
│  │     Game Canvas        │  │ [▓] [S] [T] [⌫]       │  │
│  │                        │  │  ↑    ↑   ↑   ↑       │  │
│  │  (live preview of      │  │ wall spwn trm erase   │  │
│  │   editor state via     │  ├────────────────────────┤  │
│  │   MapPreviewComponent) │  │ ┌──────────────────┐   │  │
│  │                        │  │ │▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│   │  │
│  │                        │  │ │▓             S  ▓│   │  │
│  │                        │  │ │▓   ▓▓▓▓        ▓│   │  │
│  │                        │  │ │▓   ▓    T      ▓│   │  │
│  │                        │  │ │▓   ▓▓▓▓        ▓│   │  │
│  │                        │  │ │▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓│   │  │
│  │                        │  │ └──────────────────┘   │  │
│  │                        │  │ ↑ paintable mini-grid  │  │
│  │                        │  ├────────────────────────┤  │
│  │                        │  │ [Import] [Export]      │  │
│  └────────────────────────┘  └────────────────────────┘  │
└──────────────────────────────────────────────────────────┘
```

Select a tool. Click or drag on the grid. Changes appear instantly on both the mini-grid and the game canvas.

The `MapEditorState` (extends `ChangeNotifier`) holds a 50x50 array of cell types. The sidebar listens for changes to repaint the mini-grid. The game canvas renders a `MapPreviewComponent` that also listens for changes.

## Performance: caching as Picture

The first implementation re-rendered the entire 50x50 grid on the game canvas every frame. At 60fps, that's 2,500 rectangle draws per frame. It worked, but it was wasteful.

**PR #90** cached the render as a `Picture` object:

```dart
// Only re-render when the grid actually changes
void _rebuildCache() {
  final recorder = PictureRecorder();
  final canvas = Canvas(recorder);

  // Draw all 2,500 cells once
  for (var y = 0; y < gridSize; y++) {
    for (var x = 0; x < gridSize; x++) {
      _drawCell(canvas, x, y);
    }
  }

  _cachedPicture = recorder.endRecording();
}

@override
void render(Canvas canvas) {
  canvas.drawPicture(_cachedPicture);  // One GPU call
}
```

After caching: one `drawPicture()` call per frame instead of 2,500 `drawRect()` calls. The cache invalidates only when a cell changes.

## Wall occlusion

**PR #92** was the visual leap. Before this, characters always rendered on top of walls. After: characters render *behind* walls when they're north of the wall.

The trick: **y-based depth sorting**. Every component's Flame `priority` equals its grid row:

```
             y=0  ┌───────────────────────────┐
             y=1  │  wall sprite (priority=3)  │
             y=2  │          ▓▓▓▓▓▓           │
             y=3  │          ▓▓▓▓▓▓           │  ← wall at row 3
             y=4  │     🧑                     │  ← player at row 4
             y=5  │  (player priority=4)       │
                  └───────────────────────────┘

  Player row (4) > Wall row (3) → player renders IN FRONT of wall ✓

             y=0  ┌───────────────────────────┐
             y=1  │     🧑                     │  ← player at row 2
             y=2  │  (player priority=2)       │
             y=3  │          ▓▓▓▓▓▓           │  ← wall at row 3
             y=4  │          ▓▓▓▓▓▓           │
             y=5  │  wall sprite (priority=3)  │
                  └───────────────────────────┘

  Player row (2) < Wall row (3) → player renders BEHIND wall ✓
```

Flame's `World` component sorts children by priority, so this just works. The player's priority updates every frame based on its current position:

```dart
@override
void update(double dt) {
  priority = position.y.round() ~/ gridSquareSize;
}
```

`WallOcclusionComponent` creates sprite overlays from the background PNG for walls. Each overlay extends 1 cell above a barrier. The effect: walk behind a wall and the wall smoothly covers your character.

<!-- SCREENSHOT: Player character half-hidden behind a wall on the L-Room map, demonstrating the occlusion effect. -->

## Themed maps

**PR #86** added two maps designed as ASCII art:

**The Library** — Bookshelves forming reading nooks with 4 terminals:
```
##################################################
#...........#....................................#
#...........#..........####..........####.........#
#...........#..........####..........####.........#
#...........#....................................#
#...........#....T...............T...............#
#...........#....................................#
#...........#....................................#
#.....S.....#..........####..........####.........#
#...........#..........####..........####.........#
#...........#....................................#
#...........#....T...............T...............#
#...........#....................................#
##################################################
```

**The Workshop** — A maker-space with 2 terminals:
```
. = open floor
# = workbench / barrier
S = player spawn
T = coding terminal
```

The ASCII format is simple and human-readable. The parser (`map_parser.dart`) converts each character to a cell type and builds a `GameMap` with barriers, spawn point, and terminal positions.

## Runtime map switching

**PR #95** added a `MapSelector` dropdown to the toolbar. Choose a map and `TechWorld.loadMap()` tears down the current components and creates new ones:

```dart
void loadMap(GameMap map) {
  // Remove old components
  _barriers.forEach(remove);
  _terminals.forEach(remove);
  _occlusionComponents.forEach(remove);

  // Create new components for the new map
  _currentMap = map;
  _createBarriers();
  _createTerminals();
  _createOcclusion();
  _movePlayerToSpawn();
}
```

Auto-exits editor mode on map switch (**PR #96**). No stale state, no mixed maps.

## What we learned

1. **`Picture` caching is dramatic.** Going from 2,500 draw calls to 1 made the editor feel weightless. Any time you're drawing the same thing every frame, cache it.

2. **Y-based depth sorting is elegant.** No z-buffer, no render order lists, no manual management. Just `priority = row`. Flame handles the rest.

3. **ASCII map format is underrated.** It's version-controllable, diffable, human-readable, and trivial to parse. For a 50x50 grid, it's the perfect authoring format.

---

*Next: [Devlog #10: Avatars, Tilesets & Rooms](/posts/10-avatars-tilesets-rooms/) — from coloured squares to pixel art, and the lobby system.*
