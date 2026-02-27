---
title: "Devlog #12: The Editor Grows Up"
description: "Automapping rules that generate shadows and trim, TMX file import from Tiled, persistent custom maps in Firestore, player DMs, and user profiles. The February 2026 finale."
postDate: 2026-02-27
author: Nick & the Adventures in Tech meetup
---

## Where we are

147 commits. 173 merged PRs. A project that started as red squares on a 10x10 grid now has proximity video chat, an AI tutor with a body, 23 coding challenges, animated pixel art tilesets, and a visual map editor with auto-terrain.

This final devlog (for now) covers the features that landed in the last week.

## Automapping rules engine (#152, #163)

Auto-terrain picks the right *sprite variant* for each cell. **Automapping** generates *additional content* based on rules:

```
Rule: "shadow below wall"
  IF cell(x, y) is a wall tile
  THEN place shadow tile at cell(x, y+1)

Rule: "trim at floor-wall boundary"
  IF cell(x, y) is floor AND cell(x, y-1) is wall
  THEN place trim tile at cell(x, y)

Rule: "user paint overrides"
  IF cell was painted by user AND cell has automapped content
  THEN prune the automapped content
```

The rules engine scans the map after each edit. It's conservative — user-painted tiles always win over automapped tiles (**PR #165**). If you paint a floor tile where automapping placed a shadow, the shadow gets pruned.

```
Before automapping:           After automapping:

  ▓▓▓▓▓▓▓▓                    ▓▓▓▓▓▓▓▓
  ........                     ░░░░░░░░  ← shadow generated
  ........                     ........
  ........                     ........

  ▓ = wall    . = floor    ░ = auto-generated shadow
```

## Auto-barriers (#164, #166)

Each tileset defines which tiles are impassable via a `barrierTileIndices` set in its metadata. When you paint a wall tile, the corresponding grid cell automatically becomes a collision barrier. No need to manually paint barriers on top of visual tiles.

This also fixed wall occlusion for object tiles. Previously, occlusion only worked with the background PNG. Now it works with tileset sprites too — any tile marked as a barrier gets an occlusion overlay, so characters correctly render behind pixel-art furniture and walls.

## TMX import (#171)

For complex maps, our in-game editor is convenient but limited. **Tiled** (the popular open-source map editor) is the industry standard for a reason — layers panel, tile stamp tool, terrain brushes, undo history, export formats.

PR #171 added TMX file import. Drop a `.tmx` file exported from Tiled and the editor parses it into our native format:

```
Tiled (.tmx XML)
    │
    ▼
TMX Parser
    │  reads layers, tilesets, tile GIDs
    ▼
TileLayerData + TileRef mapping
    │
    ▼
MapEditorState
    │  same format as hand-painted maps
    ▼
Save to Firestore (same as any custom map)
```

This bridges two workflows: quick edits in our in-game editor, detailed work in Tiled's full desktop application. Both produce the same `GameMap` data structure.

## Layer filtering (#168)

With multiple tile layers (ground, objects, decorations), the tileset palette was showing hundreds of tiles at once. PR #168 added layer filtering — select a layer and the palette shows only tiles relevant to that layer.

```
┌─────────────────────────────┐
│  Layer: [Ground ▼]          │
│                             │
│  ┌─────────────────────┐   │
│  │ grass dirt water     │   │  ← only ground tiles
│  │ sand  stone path     │   │
│  └─────────────────────┘   │
│                             │
│  vs.                        │
│                             │
│  Layer: [Objects ▼]         │
│                             │
│  ┌─────────────────────┐   │
│  │ desk  chair lamp     │   │  ← only object tiles
│  │ shelf book  plant    │   │
│  └─────────────────────┘   │
└─────────────────────────────┘
```

## Save, load, delete custom maps (#173)

The capstone feature: **persistent custom maps**. Design a map in the editor, save it to Firestore, and it appears in the map selector for all players.

```
Editor workflow:
  1. Enter editor mode
  2. Paint terrain (auto-terrain handles edges)
  3. Place objects (auto-barriers handles collision)
  4. Set spawn point and terminals
  5. Automapping adds shadows and trim
  6. [Save] → name it → stored in Firestore
  7. Any player → [Map selector] → sees your map → loads it
```

Maps are stored as JSON in Firestore: grid data, tile layer data, spawn point, terminal positions, and metadata (name, author, creation date). Loading is fast — the JSON is small (sparse grid representation, only non-empty cells stored).

## Player-to-player DMs (#156–159)

Not map-related, but a significant social feature: **direct messages** between players, backed by Firestore.

```
┌──────────────────────────────────────┐
│ Chat        [Room] [DMs]             │
├──────────────────────────────────────┤
│                                      │
│ Conversations:                       │
│  ┌────────────────────────────────┐  │
│  │ 💬 Alice         2m ago       │  │
│  │ "thanks for the help!"        │  │
│  ├────────────────────────────────┤  │
│  │ 💬 Bob           1h ago       │  │
│  │ "want to pair on the cipher?" │  │
│  └────────────────────────────────┘  │
│                                      │
│ [+ New Message]                      │
│                                      │
├──────────────────────────────────────┤
│ [Type a message...]          [Send]  │
└──────────────────────────────────────┘
```

Chat history persists across sessions — sign out, sign back in, and your conversations are still there. The "New Message" button (**PR #159**) lets you start a conversation with any player currently in the room.

PRs #157 and #158 fixed a bug where loading chat history blocked room loading. The fix: a single overall timeout for `loadHistory()` so slow Firestore reads don't prevent the player from entering the game.

## User profiles (#172)

Display name and profile picture upload. Your identity in the game world is now more than an email address.

## The state of things

Here's what Tech World looks like today:

```
┌──────────────────────────────────────────────────────────┐
│ Lobby: pick a room                    [👤 Nick ▼]       │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐               │
│  │ Study     │  │ Workshop │  │ My Map   │               │
│  │ 🗺 library│  │ 🗺 custom│  │ 🗺 custom│               │
│  │ 👥 2      │  │ 👥 0     │  │ 👥 1     │               │
│  │ [Join]    │  │ [Join]   │  │ [Join]   │               │
│  └──────────┘  └──────────┘  └──────────┘               │
│                                                          │
│ After joining:                                           │
│ ┌────────────────────────────────┐ ┌──────────────────┐  │
│ │   📚  📚  📚                   │ │ Chat        [DMs]│  │
│ │   📚  📚  📚  animated water  │ │                  │  │
│ │        ≈≈≈≈≈   tiles          │ │ You: How do I... │  │
│ │   🤖   ≈≈≈≈≈                  │ │                  │  │
│ │  Clawd ≈≈≈≈≈    📹 ← video   │ │ 🤖 Clawd:       │  │
│ │         🧑←you   bubble       │ │ Great question!  │  │
│ │          🧑←alice (nearby)    │ │ ...              │  │
│ │   🖥️  ← terminal (glowing)   │ │                  │  │
│ │                               │ │ [Ask Clawd...] 🎤│  │
│ └────────────────────────────────┘ └──────────────────┘  │
│                               [Map▼] [🗺 Editor] [👤]   │
└──────────────────────────────────────────────────────────┘
```

<!-- SCREENSHOT: The actual game running with all features visible — a populated room with tiled map, at least one other player nearby with video bubble, Clawd visible, chat panel open. This is the hero screenshot for the devlog series. -->

## The full journey

```
Jun 2023   10×10 grid, red squares, 2 sprites
           ↓
Jul 2024   20×20 grid, background PNG, multiplayer via WebSocket
           ↓  (15 months quiet)
Oct 2025   LiveKit migration begins
           ↓
Dec 2025   Proximity video bubbles, Clawd bot, CI/CD
           ↓
Jan 2026   Zero-copy FFI video, all-LiveKit architecture
           ↓
Feb 2026   Voice, code editor, map editor, tilesets, rooms,
           auto-terrain, automapping, DMs, user profiles
           ↓
Today      147 commits, 173 PRs, 12 devlogs
```

## What's next

- **More auto-terrain types** beyond water (grass-to-dirt, stone paths)
- **Challenge evaluation** — actually running submitted code in a sandbox
- **Spectator mode** — watch someone else work on a challenge
- **Mobile native** — the game runs on iOS and Android but needs touch UI polish
- **Self-hosted LiveKit** — moving off the free tier for production use

Come to a meetup. Build something with us.

*Follow the development on [GitHub](https://github.com/enspyrco/tech_world).*

---

*This is the first batch of devlogs for Adventures in Tech World. Future posts will cover individual features as they're built — shorter, more focused, more GIFs. Stay tuned.*
