---
title: "Devlog #6: All-LiveKit — Deleting the Game Server"
description: "We moved chat and position sync to LiveKit data channels, ripped out every WebSocket dependency, and went fully serverless. One marathon day, 10 PRs."
postDate: 2026-02-01
author: Nick & the Adventures in Tech meetup
---

## January 29: the marathon

Ten PRs in one day. By the end, our custom WebSocket game server was gone — entirely deleted — and everything ran through LiveKit.

Here's the timeline:

```
09:32  PR #55   Wire ChatService to LiveKit data channels
09:56  PR #56   Position sync via LiveKit data channels
16:23  PR #57   Fix bot not appearing on subscribe
17:38  PR #58   Shared chat for all participants
18:08  PR #59   Upgrade placeholder bubbles to video
19:55  PR #60   ███ REMOVE WEBSOCKET SERVER ███
20:28  PR #61   Document the new data channel topics
20:40  PR #62   Auth menu with sign out
22:03  PR #63   Clear LiveKit state on sign out
22:47  PR #64   Tests for GameMap, Direction, AuthMenu
```

PR #60 was the satisfying one. Every import of `NetworkingService`, every reference to the WebSocket URL, every connection handler — deleted.

## The data channel protocol

LiveKit data channels work like lightweight WebSocket messages. Each message has a "topic" string and a binary payload. We standardised on four topics:

```
┌──────────────────────────────────────────────────┐
│              LiveKit Data Channels                │
├──────────┬───────────┬───────────────────────────┤
│ Topic    │ Direction │ Payload                   │
├──────────┼───────────┼───────────────────────────┤
│ position │ broadcast │ {x, y, mapId, identity}   │
│ chat     │ broadcast │ {message, sender}         │
│ chat-rsp │ broadcast │ {message, from: bot}      │
│ ping/pong│ targeted  │ {timestamp}               │
└──────────┴───────────┴───────────────────────────┘

broadcast = all participants receive it
targeted  = specific participant only
```

Position updates use **unreliable** delivery (UDP-semantics). If a position packet drops, the next one will correct it — we're sending 10 updates per second, so one lost packet is invisible.

Chat messages use **reliable** delivery (TCP-semantics). You don't want questions to Clawd disappearing into the void.

## Shared chat: a design choice

We made a deliberate decision: **all chat is shared**. When anyone asks Clawd a question, everyone in the room sees it and the answer.

```
┌──────────────────────────────────────────────┐
│ Chat                                     [─] │
├──────────────────────────────────────────────┤
│                                              │
│  Alice: How do I reverse a string in Dart?   │
│                                              │
│  🤖 Clawd: Great question! You can use      │
│  .split('').reversed.join('') or the more    │
│  efficient StringBuffer approach...          │
│                                              │
│  Bob: What about recursion?                  │
│                                              │
│  🤖 Clawd: Absolutely! Here's a recursive   │
│  approach...                                 │
│                                              │
├──────────────────────────────────────────────┤
│ [Type a message...]              [🎤] [Send] │
└──────────────────────────────────────────────┘
```

This mirrors how the physical meetup works. Someone asks a question, the whole table hears the answer and learns from it. Private DMs would come later ([Devlog #12](/posts/12-the-editor-grows-up/)), but the shared-by-default model creates a collaborative learning environment.

## Token-based agent dispatch

Here's a problem that only appears in production: the bot sometimes doesn't join the room.

LiveKit has "automatic agent dispatch" — when a new room is created, LiveKit tells registered agent workers to join. But our room (`tech-world`) has a **5-minute empty timeout**. If a user signs out and signs back in within 5 minutes, the room still exists. It's not "new." So automatic dispatch never fires. No bot.

```
User signs in  → room "tech-world" created → bot dispatched ✓

User signs out → 3 minutes pass → user signs in again
               → room "tech-world" still exists
               → NOT a new room
               → automatic dispatch does NOT fire
               → no bot ✗
```

**The fix: token-based dispatch.** Our Firebase Cloud Function embeds a `RoomAgentDispatch` in every user's access token:

```
Token generation (Firebase Cloud Function):
  1. User requests a LiveKit token
  2. Function creates token with RoomAgentDispatch config
  3. Token says: "when I join, dispatch bot-claude"
  4. User joins room with this token
  5. LiveKit reads dispatch config → sends job to bot worker
  6. Bot joins, regardless of room age
```

Zero extra API calls. Zero extra infrastructure. Just metadata in the authentication token.

## Sign-out / sign-in lifecycle

The all-LiveKit architecture made auth state transitions cleaner. The service locator pattern (`Locator`) means each sign-in gets completely fresh service instances:

```
Sign-in:
  1. Create LiveKitService  → Locator.add()
  2. Create ChatService     → Locator.add()
  3. Create ProximityService → Locator.add()
  4. Connect to LiveKit room

Sign-out:
  1. LiveKitService.dispose()  → Locator.remove()
  2. ChatService.dispose()     → Locator.remove()
  3. ProximityService.dispose() → Locator.remove()
  4. Clean slate. No stale state.
```

PR #63 fixed a bug where stale LiveKit room state prevented reconnection after sign-out. The fix: `dispose()` must fully tear down the LiveKit connection, not just stop sending. Otherwise the room object thinks it's still connected and refuses to reconnect.

## The numbers

```
BEFORE                          AFTER
──────                          ─────
Servers to maintain: 2          Servers to maintain: 1
  (WebSocket + bot)               (bot only)
Custom networking code: ~400 LOC  Custom networking code: 0
Data protocols: 2               Data protocols: 1
  (WebSocket + LiveKit)           (LiveKit only)
Monthly cost: ~$15              Monthly cost: ~$7
  (GCE WebSocket + GCE bot)      (GCE bot only)
```

## What we learned

1. **Data channels can replace a game server** at our scale. For ~50 concurrent users with position updates at 10Hz, LiveKit handles it without breaking a sweat.

2. **Token-based dispatch is more robust than automatic dispatch.** Automatic dispatch depends on room lifecycle. Token-based dispatch happens on every connection. Always prefer the approach that doesn't depend on external state.

3. **Dispose means dispose.** Half-disposing a real-time connection leaves you in a state where you can't reconnect and can't create fresh. Full teardown on sign-out, full creation on sign-in.

---

*Next: [Devlog #7: Clawd Gets a Voice](/posts/07-clawd-gets-a-voice/) — text-to-speech, speech-to-text, and the bot presence indicator.*
