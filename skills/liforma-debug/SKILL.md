---
name: liforma-debug
description: Troubleshoot Liforma Experience embeds — silent avatar, no audio, CORS/origin errors, 401/403 mint failures, wrong experience id, iframe not loading, speak before audio unlock, wrong CDN URL.
---

# Liforma debug

Symptom → check → fix. Prefer shipped docs over inventing APIs.

**Index:** https://docs.liforma.ai/llms.txt  
**Errors:** https://docs.liforma.ai/api-reference/errors  
**Portal:** https://app.liforma.ai

## Silent avatar / blank iframe

1. Confirm `experienceId` is a valid `exp_…` the project can mint.  
2. Open Network: mint call to `/v1/public-sessions` or your `sessionEndpoint` — status and JSON body.  
3. Confirm player iframe loads `player.liforma.ai` (or local player in stack=local).  
4. Check browser console for postMessage / CSP errors.

## 401 / 403 on mint

| Path | Likely cause |
|------|----------------|
| Browser mint (`/v1/public-sessions`) | Origin not allowlisted for the project/experience |
| Server mint (`/v1/sessions`) | Missing/invalid API key; key not sent from **server** route |
| `sessionEndpoint` | Route not same-origin; returns HTML/error instead of manifest JSON |

Fix: portal Origins or server API key; never expose the API key to the browser.

Docs: [Browser embeds](https://docs.liforma.ai/avatar-experiences/browser-embeds) · [Server sessions](https://docs.liforma.ai/avatar-experiences/server-sessions)

## CORS / Origin errors

- Browser mint requires the page’s **exact origin** in the developer portal allowlist (scheme + host + port).  
- Localhost ports must be listed individually (e.g. `http://localhost:4003`).  
- Do not expect R2/CDN operator CORS docs to be something integrators configure — that is Liforma infrastructure.

## No audio / speak() does nothing

1. Browser autoplay policy: user must unlock audio (usually via player start / gesture).  
2. Call `speak()` only after the experience has started / audio unlocked (e.g. `onStarted` / started callback).  
3. Confirm mode supports speech (`presenter` / conversation as documented).

Docs: [Experience API](https://docs.liforma.ai/avatar-experiences/experience-api) · [Events](https://docs.liforma.ai/avatar-experiences/events)

## Wrong or ancient CDN script

- Use **`https://cdn.liforma.ai/sdk/v2/client.js`** only.  
- If the page still loads `/sdk/v1/` or undocumented URLs, replace with v2.  
- Prefer `npm install @liforma/client` for framework apps: https://www.npmjs.com/package/@liforma/client

## Invented API / wrong events

- No `Liforma.textToSpeech()` — use `Experience` + modes / `speak()`.  
- `modeChange` is a bare string: `'listening' | 'speaking' | 'thinking'`.  
- `ConversationMessage.status === 'final'` (not `final: boolean`).  
- `close` / `onStateUpdate` from attach / component props.

## Widget / thumbnail issues

- Thumbnail: no session — preview/plates only.  
- Widget: collapsed is light; session starts on expand (see docs). Props may still iterate — check current docs pages, not outdated blogs.

[ExperienceThumbnail](https://docs.liforma.ai/avatar-experiences/experience-thumbnail) · [ExperienceWidget](https://docs.liforma.ai/avatar-experiences/experience-widget)

## Still stuck

1. Reproduce with [examples.liforma.ai](https://examples.liforma.ai) / [GitHub examples](https://github.com/LiformaLtd/examples.liforma.ai).  
2. Compare against [Quick start](https://docs.liforma.ai/getting-started/quick-start).  
3. Capture mint request/response (redact API keys) and iframe URL before changing random config.
