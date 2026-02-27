---
title: "Devlog #10: Avatars, Tilesets & Rooms"
description: "Choosing your character, rendering real pixel art instead of coloured rectangles, and the lobby system that let different groups have different worlds."
postDate: 2026-02-21
author: Nick & the Adventures in Tech meetup
---

## From rectangles to pixel art

For months, barriers were blue rectangles. The game world had a nice background image (`single_room.png`), but everything on top of it was programmer art — solid-colour blocks marking where you couldn't walk.

**PR #109** changed that. The tileset infrastructure:

```
BEFORE                              AFTER
──────                              ─────
Blue rectangles for walls           LimeZu pixel art tilesets
One background PNG                  Tile layers with depth sorting
All barriers look the same          Bookshelves, desks, computers, walls

┌────────────────────┐              ┌────────────────────┐
│  ████              │              │  📚📚              │
│  ████  ← blue rect │              │  📚📚  ← bookshelf│
│        🧑          │              │        🧑          │
│  ████              │              │  🖥️💺              │
│  ████              │              │  🖥️💺  ← desk set │
└────────────────────┘              └────────────────────┘
```

The key classes:

```
TilesetRegistry
    │  Loads and stores tileset sprite sheets
    │
    ├── TileRef { tilesetId, tileIndex }
    │   Points to a specific sprite in a specific sheet
    │
    ├── TileLayerData
    │   Sparse 50×50 grid of TileRefs (one visual layer)
    │
    └── TileObjectLayerComponent
        Flame component that renders a tile layer
        Injects sprites into parent World (not as children)
        so they participate in global y-based depth sorting
```

That last point is important. If tile sprites were children of the layer component, they'd all share the layer's priority. By injecting them directly into the World, each sprite gets its own y-based priority. A desk at row 5 sorts independently from a bookshelf at row 3.

## Avatar selection

**PR #106** — pick your character:

```
┌──────────────────────────────────────┐
│  Choose your avatar:                 │
│                                      │
│  🧑  👩  🧔  👱  👨  🧑‍🦱             │
│  ↑                                   │
│  NPC11 NPC12 NPC13 ...              │
│                                      │
│  Each = spritesheet with             │
│    4 directions × 3 walk frames      │
│  Size: 32×64 pixels                  │
│  LimeZu Modern Interiors pack        │
└──────────────────────────────────────┘
```

Your avatar selection persists across sessions (stored in `SharedPreferences`). Other players see your chosen character walking around the map. Each sprite sheet has walk animations for all four cardinal directions.

## Bot embodiment

With the room system (**PR #133**), Clawd got a visible body in the game world. `BotCharacterComponent` renders a special sprite that follows a simple patrol path. Walk near the bot's character to trigger proximity chat.

```
Before: Clawd = invisible chat participant
         (text-only, in the sidebar)

After:  Clawd = visible character walking around
         (sprite in the world + chat in sidebar)

  ┌─────────────────────────────────┐
  │                                 │
  │    🤖 ← bot-claude sprite      │
  │     (patrolling)                │
  │                                 │
  │              🧑 ← you           │
  │              (walk close to     │
  │               start chatting)   │
  │                                 │
  └─────────────────────────────────┘
```

This small change had a big impact on feel. Players approach the bot like they'd approach a person. The AI tutor feels like part of the world, not a text interface hiding behind a button.

## The room system

**PR #133** was the biggest architectural change since the LiveKit pivot: a **room and lobby system**.

Before: everyone joins the same LiveKit room (`tech-world`), sees the same map, shares one chat. Fine for demos, but it doesn't scale to a meetup with multiple groups working on different things.

After:

```
┌──────────────────────────────────────────────────────┐
│  Lobby                                               │
│                                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐           │
│  │ Room A   │  │ Room B   │  │ Room C   │           │
│  │ 🗺 maze  │  │ 🗺 lib   │  │ 🗺 open  │           │
│  │ 👥 3/10  │  │ 👥 2/10  │  │ 👥 0/10  │           │
│  │ [Join]   │  │ [Join]   │  │ [Join]   │           │
│  └──────────┘  └──────────┘  └──────────┘           │
│                                                      │
│  Each room = separate LiveKit room                   │
│            + its own map                             │
│            + its own chat                            │
│            + its own Clawd instance                  │
└──────────────────────────────────────────────────────┘
```

Rooms are stored in Firestore. Each room has:
- A name
- A map configuration (which predefined or custom map to use)
- Participant count
- A generated seed for procedural elements

The bot uses token-based dispatch per-room. When you join Room A, your token dispatches `bot-claude` to Room A's LiveKit room. Different rooms get different bot instances.

## Procedural map generation

Each room can have a unique layout generated from a seed:

```dart
final random = Random(room.seed);

// Place barriers deterministically from the seed
for (var i = 0; i < barrierCount; i++) {
  final x = random.nextInt(gridSize);
  final y = random.nextInt(gridSize);
  grid[y][x] = CellType.barrier;
}
```

Same seed = same map, every time. Different rooms get different seeds. The generation uses extracted named constants for tile indices, so the procedural layouts use real tileset sprites, not coloured rectangles.

<!-- SCREENSHOT: Lobby screen showing 2-3 room cards with map previews and player counts. -->

## Challenge progression

**PR #109** also added challenge tracking. Completed challenges persist per-player:

```
Terminal states:
  ✨ = unsolved (glowing, inviting)
  ✅ = completed (still accessible for review)

Terminal icon updates on:
  - Challenge submission + positive Clawd review
  - Auth state change (reload progress for signed-in user)
```

PR #112 fixed a bug where terminal states didn't refresh when a different user signed in. The progress is per-user, so switching accounts needs to reload the completion data.

## What we learned

1. **Inject tile sprites into the World, not into a layer component.** Child components inherit their parent's priority. Direct world children get their own priority. For depth sorting to work, each sprite needs independent priority.

2. **Bot embodiment changes interaction patterns.** When the AI has a visible position in the world, players approach it spatially. It creates a more natural interaction model than a chat button.

3. **Rooms are multiplayer's scaling unit.** One shared space works for 5 people. For 50, you need rooms. The LiveKit-per-room model means each room's video/audio/data is completely isolated.

---

*Next: [Devlog #11: Auto-Terrain](/posts/11-auto-terrain/) — the bitmask algorithm that automatically picks the right water-to-grass edge tile.*
