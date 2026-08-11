---
name: liforma-debug
description: >
  Troubleshoot Liforma Experience embeds. Use when the avatar is silent, the iframe is blank,
  audio does not play, speak() does nothing, BYO connect* lipsync fails, sessions fail to start,
  mint returns 401/403, CORS/Origin errors appear, events look wrong, the wrong CDN script is
  loaded, or Widget/Thumbnail misbehaves. Also use for "Liforma not working", "Session Launch error",
  or "browser-sessions failed".
license: MIT
metadata:
  author: liforma
  version: "1.2.0"
---

# Liforma debugging

Symptom → most likely cause → fix. Prefer shipped docs over inventing APIs.

**Index:** https://docs.liforma.ai/llms.txt  
**Errors:** https://docs.liforma.ai/api-reference/errors  
**Portal:** https://app.liforma.ai

## General approach

1. Reproduce with a known-good path: [Quick start](https://docs.liforma.ai/getting-started/quick-start) or [examples.liforma.ai](https://examples.liforma.ai).
2. Capture the **mint** request/response (redact API keys) and the **iframe** URL.
3. Confirm you are not mixing browser-mint expectations with a broken `sessionEndpoint` (or vice versa).
4. Fix secrets / origins before tweaking random component props.

## Silent avatar / blank iframe

**Most likely: mint failed or never ran.**

1. Valid `experienceId` (`exp_…`) the project can mint?  
2. Network: `/v1/browser-sessions` or your `sessionEndpoint` — status + JSON body (`{ session, launch }`).  
3. Iframe loads `player.liforma.ai` (or local player when `stack=local`)?  
4. Console: postMessage / CSP / mixed-content errors?

**Also:** experience unpublished, wrong project, or ad blockers interfering with the iframe.

## 401 / 403 on mint

| Path | Most likely cause |
|------|-------------------|
| Browser mint (`/v1/browser-sessions`) | Origin not allowlisted (exact scheme + host + port) |
| Server mint (`/v1/sessions`) | Missing/invalid API key; key used from the **browser** |
| `sessionEndpoint` | Not same-origin; returns HTML/login instead of Session Launch JSON |

**Fix:** portal Origins or server-side API key. Never paste the API key into frontend code to “bypass” 403.

## CORS / Origin errors

- Browser mint cares about the **page Origin**, not R2 operator CORS (integrators do not configure Liforma CDN CORS).
- List each local port separately (`localhost:4001` vs `5173`).
- `https://app.example.com` ≠ `http://app.example.com`.

## No audio / `speak()` does nothing

**Most likely: autoplay / audio not unlocked yet.**

1. User gesture / player start must unlock audio.  
2. Call `speak()` / `connect*` only after started (`onStarted` / equivalent).  
3. Mode must support speech (presenter / conversation per docs).  
4. OS/browser mic mute only matters for **listening**, not for playback of `speak()`.

Docs: [Experience API](https://docs.liforma.ai/avatar-experiences/experience-api) · [Events](https://docs.liforma.ai/avatar-experiences/events)

## BYO voice connected but mouth/audio wrong

| Symptom | Check |
|---------|--------|
| No mouth / silent avatar | Session has `externalSpeechAudio`? Helper called **after** audio unlock? |
| Double voice | Vendor `<audio>` / LiveKit attach still playing — mute vendor speaker; only Liforma plays |
| Deepgram / Gemini never connects | Same-origin **WS proxy** running? Key stays on server |
| Weak lipsync | Prefer helper + transcript path; sample rate must be known before PCM write |

Prefer shipped `connect*` helpers and the example `helloByo.ts` — see integrate [byo-voice.md](../liforma-integrate/references/byo-voice.md).

## Wrong or ancient CDN script

- Required: `https://cdn.liforma.ai/sdk/v2/client.js`  
- If you see `/sdk/v1/` or random forks, replace with v2 or switch to `npm install @liforma/client`.

## Invented API / wrong events

Silent logic bugs — no HTTP error:

- No `Liforma.textToSpeech()` — use `Experience` + modes / `speak()`, or BYO `connect*` helpers.  
- Do not invent custom vendor bridges when `@liforma/client/{elevenlabs,openai,…}` exists.  
- Prefer `activityChange` envelope (`data`: `'idle' | 'listening' | 'thinking' | 'speaking'`).  
- `ConversationMessage.status === 'final'` (not `final: boolean`).  
- `close` / `onPlayerStatusChange` from attach / component props.

## Widget / thumbnail

- **Thumbnail:** no session — if they expect conversation, they need `Experience` or Widget expand.  
- **Widget:** collapsed is light; heavy work on expand. Check current docs; props may still iterate.

[ExperienceThumbnail](https://docs.liforma.ai/avatar-experiences/experience-thumbnail) · [ExperienceWidget](https://docs.liforma.ai/avatar-experiences/experience-widget)

## Lesson / multi-experience apps

Switching `experienceId` mid-flight without closing the session causes confusing UI. Close / destroy the active session before opening another (Spanish Tutor pattern in examples).

## Still stuck

1. Diff against https://github.com/LiformaLtd/examples.liforma.ai for the same framework.  
2. Try Meet: https://www.liforma.ai/meet  
3. Use skill `liforma-integrate` references (`server-mint`, `browser-mint`, `byo-voice`) to re-validate the pathway.  
