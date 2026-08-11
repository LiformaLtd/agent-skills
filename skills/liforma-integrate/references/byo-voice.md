# Bring-your-own voice

Keep a third-party speech stack (mic / brain / TTS). Liforma owns avatar playback + lipsync.

**Docs:** https://docs.liforma.ai/avatar-experiences/bring-your-own-voice  
**Examples:** https://examples.liforma.ai (gallery) · `examples/*-embed` in https://github.com/LiformaLtd/examples.liforma.ai

## When to use

User already has ElevenLabs Agents, OpenAI Realtime, Deepgram Voice Agent, Gemini Live, LiveKit, or another PCM/file source — and wants a Liforma avatar to lip-sync to that audio.

## Session + Experience shape

1. Mint with capability **`externalSpeechAudio`** (usually also speech animation).
2. Mount Experience often as `mode="presenter"` with **`speechInputMode="off"`** so only the vendor owns the mic.
3. Wait until the player has **started** (audio unlocked), then connect the vendor helper.
4. Call **`bridge.end()`** when the conversation finishes.

## Prefer first-class helpers (do not hand-roll)

| Package | Helper | Host credentials |
|---------|--------|------------------|
| `@liforma/client/elevenlabs` | `connectElevenLabsAgent` | `signedUrl` (prefer) or demo `agentId` — peer `@elevenlabs/client` |
| `@liforma/client/openai` | `connectOpenAiRealtime` | ephemeral Realtime client secret |
| `@liforma/client/openai` | `connectOpenAiRealtimeWebRtc` | same (preferred browser media path) |
| `@liforma/client/deepgram` | `connectDeepgramAgent` | same-origin WebSocket **`proxyUrl`** (BFF) |
| `@liforma/client/google` | `connectGeminiLive` | same-origin WebSocket **`proxyUrl`** (BFF) |
| `@liforma/client/livekit` | `connectLiveKitAgent` | LiveKit `url` + participant `token` — peer `livekit-client` |

Shared primitives for custom vendors: `@liforma/client/byo` (`createTurnSession`, PCM/mic helpers, `BYO_PLAYBACK_GUIDANCE`).

Lower-level speech API (only if no helper fits): `experience.speech.createUtterance` / `play` / `interrupt`.

## Copy-into-product pattern

Runnable examples isolate the SDK call in **`helloByo.ts` / `helloByo.js`** (`startByoSpeech` → one `connect*`). DemoApp / `+page` / `app.js` are scaffolding only.

| Example slug | Local port |
|--------------|------------|
| `elevenlabs-embed` | 4006 |
| `openai-realtime-embed` | 4007 |
| `deepgram-embed` | 4008 |
| `livekit-embed` | 4009 |
| `gemini-live-embed` | 4010 |

## Provider docs

- https://docs.liforma.ai/avatar-experiences/bring-your-own-voice/elevenlabs  
- https://docs.liforma.ai/avatar-experiences/bring-your-own-voice/openai  
- https://docs.liforma.ai/avatar-experiences/bring-your-own-voice/deepgram  
- https://docs.liforma.ai/avatar-experiences/bring-your-own-voice/livekit  
- https://docs.liforma.ai/avatar-experiences/bring-your-own-voice/google  
- https://docs.liforma.ai/avatar-experiences/bring-your-own-voice/other-providers  

**Not the same product:** `@liforma/elevenlabs-compatible` migrates *away* from ElevenLabs Agents. BYO **keeps** the vendor and animates a Liforma avatar.
