---
title: "Devlog #8: The In-Game Code Editor"
description: "Walk up to a glowing terminal. Tap it. A code editor appears with a Dart challenge and real LSP code completion. Here's how we built an IDE inside a game."
postDate: 2026-02-06
author: Nick & the Adventures in Tech meetup
---

## The vision

An educational game needs a place to write code. But we didn't want to redirect players to an external editor or embed a boring text field. We wanted **terminal stations in the game world** — walk up to one, interact with it, and get a coding challenge right there.

## Terminal stations

Each map can have `T` cells marking terminal positions. In the game, these render as glowing pixel art computer terminals. Walk within 2 grid squares and tap:

```
   ┌─────────────────────────────────────────┐
   │                                         │
   │         ████                             │
   │         █▓▓█  ← terminal sprite         │
   │         ████    (glowing when in range)  │
   │                                         │
   │              🧑 ← player                │
   │              (within 2 squares = can     │
   │               interact)                  │
   │                                         │
   └─────────────────────────────────────────┘

   Tap terminal → TechWorld.activeChallenge fires →
   main.dart swaps ChatPanel for CodeEditorPanel
```

The proximity check is simple: if the Manhattan distance between player and terminal is <= 2, the terminal "activates" (glows). Tap it to open the editor.

## 23 challenges

**PR #87** added the challenge library. Each challenge has a title, difficulty, description, starter code, and test cases:

```
BEGINNER (10)              INTERMEDIATE (7)
────────────               ────────────────
Hello Dart                 Binary Search
Sum a List                 Fibonacci Sequence
FizzBuzz                   Caesar Cipher
String Reversal            Anagram Checker
Even Numbers               Flatten List
Palindrome Check           Matrix Sum
Word Counter               Bracket Matching
Temperature Converter
Find Maximum               ADVANCED (6)
Remove Duplicates          ──────────
                           Merge Sort
                           Stack Implementation
                           Roman Numerals
                           Run Length Encoding
                           Longest Common Subsequence
                           Async Data Pipeline
```

Terminals cycle through challenges: `allChallenges[terminalIndex % allChallenges.length]`. Different terminals on the same map show different problems, so exploring the map means finding new challenges.

## The LSP connection

The *really* ambitious part: **real code completion and hover documentation** powered by the Dart Language Server Protocol (LSP).

The architecture:

```
Browser (Flutter web)
    │
    │  WebSocket (WSS)
    ▼
nginx (SSL termination)                   ← GCE instance
    │  limit_conn 5 per IP
    ▼
lsp-ws-proxy (localhost:9999)             ← bridges WS ↔ stdio
    │
    ▼
dart language-server --protocol=lsp       ← one per connection
    │
    ▼
/opt/lsp-workspace/                       ← shared pubspec.yaml
    pubspec.yaml                            + analysis_options.yaml
    analysis_options.yaml
```

Each WebSocket connection gets its own `dart language-server` process. The proxy (`lsp-ws-proxy`) bridges WebSocket messages to/from the language server's stdio. nginx handles SSL and connection limiting (5 per IP to prevent abuse).

The result: type `str` in the editor and get a dropdown showing `String`, `StringBuffer`, `StringSink`. Hover over a function name and see its documentation. It's a real IDE experience, inside a game, in the browser.

## The side panel UX

The editor replaces the chat panel in the sidebar (only one panel shows at a time):

```
┌──────────────────────────────────────────────────────────┐
│                                        [Map▼] [🗺] [👤] │
│  ┌────────────────────────┐  ┌────────────────────────┐  │
│  │                        │  │ ★ FizzBuzz             │  │
│  │                        │  │ Difficulty: Beginner   │  │
│  │     Game Canvas        │  ├────────────────────────┤  │
│  │                        │  │ void main() {          │  │
│  │     (player near       │  │   for (var i = 1;      │  │
│  │      terminal)         │  │       i <= 100; i++) { │  │
│  │                        │  │     // your code here  │  │
│  │                        │  │   }                    │  │
│  │                        │  │ }                      │  │
│  │                        │  │                        │  │
│  │                        │  ├────────────────────────┤  │
│  │                        │  │ [Submit to Clawd]      │  │
│  └────────────────────────┘  └────────────────────────┘  │
│                                                           │
│  Priority: map editor > code editor > chat panel          │
└──────────────────────────────────────────────────────────┘
```

Submit your solution and it's sent to Clawd via `ChatService`. Clawd reviews the code and responds with feedback. The editor closes and the chat panel returns to show the conversation.

**PR #105** later changed this to a centered modal dialog instead of a sidebar — better use of screen space and more focus on the code.

## Graceful degradation

The LSP server runs on our GCE instance. If it's down, the editor still works — you just don't get code completion or hover docs. It degrades from "IDE" to "text editor." The WebSocket connection failure is caught silently, and no error UI is shown.

```
LSP server up:    Full IDE — completion, hover, signatures
LSP server down:  Plain text editor — still fully functional
```

This was a deliberate choice. The editor's primary value is the *challenge* and the *Clawd review*, not the code completion. LSP is a nice bonus, not a requirement.

<!-- SCREENSHOT: Code editor panel open with a challenge, showing LSP code completion dropdown active. -->

## What we learned

1. **LSP is remarkably portable.** The Language Server Protocol was designed for editors like VS Code, but it works over any transport — including WebSocket from a browser. The same `dart language-server` binary that powers VS Code powers our in-game editor.

2. **Memory is the constraint.** Each `dart language-server` process uses ~200MB. On an e2-small (2GB), that limits us to 3-5 concurrent sessions. Scaling means upgrading the instance or running multiple instances behind a load balancer.

3. **Graceful degradation > error messages.** If a nice-to-have feature fails, make it invisible. Don't show an error dialog for "code completion unavailable." Just... don't show completions.

---

*Next: [Devlog #9: Map Editor v1](/posts/09-map-editor-v1/) — building a visual level editor with paint tools, wall occlusion, and live preview.*
