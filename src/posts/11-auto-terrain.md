---
title: "Devlog #11: Auto-Terrain — The Bitmask Algorithm"
description: "Paint 'water' and the editor automatically picks the correct edge, corner, and transition tiles. The 8-bit bitmask lookup that powers every good tile editor."
postDate: 2026-02-23
author: Nick & the Adventures in Tech meetup
---

## The problem

You're painting a map. You select "water" and paint a lake. But water isn't just one tile — it's a center tile, four edge tiles, four outer corner tiles, four inner corner tiles, and various transition pieces. Manually placing each one is tedious and error-prone.

Every good tile editor solves this with **auto-terrain**: paint the terrain type, and the editor figures out which specific sprite variant to use based on what's around it.

## The algorithm

For each cell, we look at all 8 neighbors (the **Moore neighborhood**) and encode them as an 8-bit bitmask:

```
  Bit layout (clockwise from top-left):

  ┌─────┬─────┬─────┐
  │ 128 │  1  │  2  │    bit 0 (1)   = N  (north)
  │ NW  │  N  │ NE  │    bit 1 (2)   = NE (northeast)
  ├─────┼─────┼─────┤    bit 2 (4)   = E  (east)
  │ 64  │  ·  │  4  │    bit 3 (8)   = SE (southeast)
  │  W  │cell │  E  │    bit 4 (16)  = S  (south)
  ├─────┼─────┼─────┤    bit 5 (32)  = SW (southwest)
  │ 32  │ 16  │  8  │    bit 6 (64)  = W  (west)
  │ SW  │  S  │ SE  │    bit 7 (128) = NW (northwest)
  └─────┴─────┴─────┘

  1 = same terrain    0 = different terrain or empty
```

Each cell's neighborhood encodes to a number from 0 to 255. Look up that number in a `bitmaskToTileIndex` map to get the correct sprite.

## The 47-tile blob pattern

With 8 bits, there are 256 possible neighbor configurations. But most map to the same visual tile. Diagonal neighbors (NE, SE, SW, NW) only matter when *both adjacent cardinal neighbors* also match. If the cell to the north is grass and the cell to the east is grass, then the NE corner matters (is it an inner corner?). But if the north cell is water, NE doesn't matter — the edge tile already handles it.

After collapsing irrelevant diagonals, there are **47 unique visual tiles** in the full "blob" tileset:

```
The 47 tiles (conceptual layout):

  ┌─────────────────────────────────────┐
  │ Outer corners (4):                  │
  │   ╔  ╗  ╚  ╝                       │
  │                                     │
  │ Edges (4):                          │
  │   ═  ║  ═  ║  (top, right, bot, L) │
  │                                     │
  │ Inner corners (4):                  │
  │   ╣  ╠  ╩  ╦                       │
  │                                     │
  │ Center (1):                         │
  │   ▓  (all 8 neighbors match)       │
  │                                     │
  │ T-junctions, peninsulas,            │
  │ single cell, etc. (34 more)         │
  └─────────────────────────────────────┘
```

The LimeZu `ext_terrains.png` tileset we're using follows this pattern for water tiles. Each tile is 16x16 pixels.

## The cascade update

When you paint a cell, it's not enough to update just that cell's tile. All 8 neighbors might need new tiles too — painting water next to an existing water edge changes that edge to a different tile.

```
Before painting (x=5, y=5):        After painting (x=5, y=5):

  . . . . . . .                     . . . . . . .
  . . . . . . .                     . . . . . . .
  . . ≈ ≈ . . .  ← water           . . ≈ ≈ ╗ . .  ← (5,4) updated
  . . ≈ ≈ . . .                     . . ≈ ≈ ║ . .  ← (5,5) NEW water
  . . . . . . .                     . . . . . . .
  . . . . . . .                     . . . . . . .

  Painting (5,5) → re-evaluate (5,5) + all 8 neighbors
  (4,4) (5,4) (6,4) (4,5) (6,5) (4,6) (5,6) (6,6)
```

In code:

```dart
void paintTerrain(int x, int y, TerrainType terrain) {
  grid[y][x] = terrain;

  // Re-evaluate this cell + all 8 neighbors
  for (var dy = -1; dy <= 1; dy++) {
    for (var dx = -1; dx <= 1; dx++) {
      _updateTile(x + dx, y + dy);
    }
  }
}

void _updateTile(int x, int y) {
  if (!inBounds(x, y)) return;
  final terrain = grid[y][x];
  if (terrain == null) return;

  final bitmask = _computeBitmask(x, y, terrain);
  final tileIndex = terrain.bitmaskToTileIndex[bitmask];
  setTile(x, y, TileRef(terrain.tilesetId, tileIndex));
}
```

## Animated water tiles

**PR #153** (which landed just before auto-terrain) added animated tile rendering. Water tiles shimmer using shared `AnimationTicker`s:

```
Frame 0     Frame 1     Frame 2     Frame 3
≈ ≈ ≈ ≈    ≋ ≋ ≋ ≋    ≈ ≈ ≈ ≈    ≋ ≋ ≋ ≋
≈ ≈ ≈ ≈    ≋ ≋ ≋ ≋    ≈ ≈ ≈ ≈    ≋ ≋ ≋ ≋

All water tiles share ONE AnimationTicker
  → perfectly synchronized animation
  → 100 water tiles = 1 timer, not 100 timers
```

Static tiles stay in a cached `Picture` (just like the map editor preview). Only animated tiles update per frame. This means a map with 2,000 static tiles and 100 animated tiles only redraws the 100 animated ones each frame.

## Auto-terrain + animated tiles together

Paint water with auto-terrain and the edges animate. The bitmask lookup picks the right *animated* tile variant for each edge/corner, and all variants share the same animation ticker. The visual result: paint a blob of water and it shimmers seamlessly, with every edge and corner looking correct.

<!-- SCREENSHOT: GIF of painting water in the map editor, showing auto-terrain selecting correct edge tiles and the water animating. -->

## What we learned

1. **The bitmask approach is O(1) per cell.** No scanning, no pattern matching, no iteration over tile atlas regions. Encode 8 neighbors as a byte, look up in a 256-entry table, done.

2. **Shared animation tickers are essential.** If every water tile had its own timer, they'd drift out of sync over time. One ticker per terrain type keeps everything locked.

3. **Cascade updates (cell + 8 neighbors) are the secret sauce.** Without them, painting a new cell would leave the adjacent cells showing stale edge sprites. The 9-cell update region is small enough to be instantaneous.

---

*Next: [Devlog #12: The Editor Grows Up](/posts/12-the-editor-grows-up/) — automapping rules, TMX import, persistent custom maps, and player-to-player DMs.*
