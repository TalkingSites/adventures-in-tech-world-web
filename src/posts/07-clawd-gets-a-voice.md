---
title: "Devlog #7: Clawd Gets a Voice"
description: "Teaching our AI tutor to speak and listen using the browser's built-in Speech APIs — plus the bot presence indicator that saved users from talking to nobody."
postDate: 2026-02-04
author: Nick & the Adventures in Tech meetup
---

## The awkward silence

Clawd could chat via text. But watching someone type a question and then *read* the AI's response felt... flat. In the physical meetup, you ask a question out loud and hear the answer. We wanted that.

## Text-to-speech: Clawd speaks

**PR #81** wired up the Web Speech API's `speechSynthesis`:

```dart
// tts_service_web.dart
void speak(String text) {
  final utterance = SpeechSynthesisUtterance(text);
  utterance.rate = 1.0;
  utterance.pitch = 1.0;
  window.speechSynthesis.speak(utterance);
}
```

That's nearly the entire implementation. The Web Speech API is remarkably simple — give it text, it speaks. The voice depends on the user's browser and OS (Chrome uses Google voices, Safari uses macOS voices, etc.).

The trick was wiring it into the right place: `ChatService` calls `TtsService.speak()` when a `chat-response` message arrives from `bot-claude`. Only bot responses are spoken — you don't want the TTS reading back your own messages.

## Speech-to-text: talking to Clawd

**PR #82** was the other direction. Click the mic button, speak your question, and it gets transcribed and sent as a chat message.

```dart
// stt_service_web.dart
void startListening(void Function(String) onResult) {
  final recognition = SpeechRecognition();
  recognition.continuous = false;
  recognition.interimResults = false;

  recognition.onresult = (SpeechRecognitionEvent event) {
    final transcript = event.results.first.first.transcript;
    onResult(transcript);
  };

  recognition.start();
}
```

Again, the Web Speech API does the heavy lifting. `SpeechRecognition` listens to the microphone, runs speech-to-text (using the browser's built-in model or a cloud service), and returns a transcript.

## The cross-platform pattern

Neither TTS nor STT work on native platforms (macOS app, iOS). We don't want to crash or show broken UI on those platforms. The solution: **conditional exports**.

```dart
// tts_service.dart
export 'tts_service_stub.dart'
    if (dart.library.js_interop) 'tts_service_web.dart';
```

Dart evaluates the condition at compile time:
- **Web build** (`dart.library.js_interop` is available) → imports `tts_service_web.dart` with the real implementation
- **Native build** → imports `tts_service_stub.dart` with a no-op

```
tts_service.dart (router)
    ├── tts_service_web.dart   ← Web: real speechSynthesis
    └── tts_service_stub.dart  ← Native: speak() does nothing
```

No `kIsWeb` checks at call sites. No runtime platform detection. The right implementation is selected at compile time.

## The bot presence problem

With voice and text chat working, we hit a UX issue: **users were talking to nobody**.

Clawd runs on a GCE instance. If the instance restarts, or the bot process crashes, or the LiveKit worker deregisters — Clawd isn't in the room. But the chat panel still shows the input field. Users type questions, hit send, and wait. And wait. The 30-second timeout eventually fires and shows a vague error.

**PR #114** fixed this with a bot presence indicator:

```
┌──────────────────────────────────────┐
│ Chat                                 │
├──────────────────────────────────────┤
│                                      │
│  ┌──────────────────────────────┐    │
│  │ ⚠️  Clawd is offline          │    │
│  │ The AI tutor isn't in the    │    │
│  │ room right now.              │    │
│  └──────────────────────────────┘    │
│                                      │
├──────────────────────────────────────┤
│ [Input disabled when bot absent ]    │
└──────────────────────────────────────┘

vs.

┌──────────────────────────────────────┐
│ Chat                            🟢   │
├──────────────────────────────────────┤
│                                      │
│  You: What's a Future in Dart?       │
│                                      │
│  🤖 Clawd: A Future represents a    │
│  value that will be available...     │
│                                      │
├──────────────────────────────────────┤
│ [Ask Clawd a question...]   [🎤][↵] │
└──────────────────────────────────────┘
```

`ChatService` tracks bot presence via LiveKit participant events:

```dart
BotStatus { absent, idle, thinking }

// When bot-claude joins the room:
participantJoined → botStatus = BotStatus.idle

// When bot-claude leaves:
participantLeft → botStatus = BotStatus.absent

// When user sends a message:
sendMessage() → botStatus = BotStatus.thinking

// When response arrives:
onChatResponse() → botStatus = BotStatus.idle
```

The fast guard in `sendMessage()` is the key UX improvement: if the bot is absent, immediately show a system message ("Clawd is offline") instead of waiting for the 30-second network timeout.

<!-- SCREENSHOT: Chat panel showing Clawd online (green dot) with a conversation in progress. Mic button visible. -->

## What we learned

1. **The Web Speech API is shockingly capable.** Both TTS and STT are one-liners. The hard part isn't the speech — it's integrating it cleanly with the rest of the app (when to speak, when to listen, how to handle errors).

2. **Conditional exports are the cleanest cross-platform pattern.** No runtime checks, no factory constructors, no `kIsWeb` scattered everywhere. The compiler picks the right file.

3. **Presence indicators prevent frustration.** Users will always find a way to interact with features that aren't available. Disabling the input and showing an explanation is much better than a 30-second timeout that ends with a cryptic error.

---

*Next: [Devlog #8: The In-Game Code Editor](/posts/08-the-code-editor/) — terminal stations, a Dart LSP server, and 23 coding challenges.*
