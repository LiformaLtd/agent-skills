---
name: liforma-integrate
description: >
  Build a Liforma avatar Experience end-to-end. Assess the host stack, pick server vs browser
  mint, choose Svelte/React/Next/vanilla (or Thumbnail/Widget), and implement without inventing
  APIs. Use when: (1) embedding a talking avatar or Experience, (2) adding @liforma/client,
  (3) minting sessions / sessionEndpoint / public-sessions, (4) ExperienceThumbnail or
  ExperienceWidget, (5) user mentions Liforma, Session Manifest, exp_ ids, or lip-synced avatar
  in a web app, (6) migrating from a CDN script tag to the npm package.
license: MIT
metadata:
  author: liforma
  version: "1.1.0"
---

# Liforma integration

Liforma is an **Avatar Experience Platform**. Integrators ship Experiences — not raw STT/LLM/TTS APIs.

```text
Experience (exp_…) → Session Manifest → @liforma/client
```

**Canonical index:** https://docs.liforma.ai/llms.txt  
**Do not invent APIs** (no `Liforma.textToSpeech()`, no alternate top-level avatar helpers). Use `Experience` + modes / `speak()`.

## Step 1 — Discover what the user already has

Scan the codebase **before** asking. Do not ask questions the repo already answers.

| Signal | Where to look | Meaning |
|--------|---------------|---------|
| `svelte` / `@sveltejs/kit` | `package.json` | Prefer `@liforma/client/svelte` |
| `react` / `next` | `package.json` | Prefer `/react` or `/next` |
| Pure `index.html` / no server | file tree | CDN web component or browser mint |
| Existing API routes / BFF | `src/routes`, `app/api`, `server/` | Can hold API key → prefer server mint |
| Auth session / cookies | auth libs, middleware | Wire `authorize()` on session route |
| `@liforma/client` or `cdn.liforma.ai/sdk` | deps, scripts | Existing integration — extend or debug, don't restart |
| `LIFORMA_` / API keys in `.env` | env files | Never move secrets into client bundles |
| Lesson / catalogue pattern | multiple `experienceId`s | Close session before switching experiences |

### Questions only if still unknown

Ask as **one short checklist**, not one-at-a-time:

```
1. Goal? (support widget, lesson tutor, marketing embed, presenter / speak API, …)
2. Do you have a backend that can hold an API key?
3. Framework preference if the repo is ambiguous?
```

## Step 2 — Pick ONE pathway (simplest that works)

Do not dump every option. Choose and explain in 2–3 sentences.

```
Has a backend that can keep secrets?
  → SERVER MINT  (POST /v1/sessions + same-origin sessionEndpoint)

Client-only / static / quick demo?
  → BROWSER MINT  (origin allowlist + POST /v1/public-sessions; path name is historical)

Gallery card only (no conversation yet)?
  → ExperienceThumbnail  (no session)

Corner launcher → expand to talk?
  → ExperienceWidget
```

Then pick the SDK surface for the host framework (see [references/surfaces.md](references/surfaces.md)).

| Pathway | Read next |
|---------|-----------|
| Server mint | [references/server-mint.md](references/server-mint.md) |
| Browser mint | [references/browser-mint.md](references/browser-mint.md) |
| Surfaces / hello world | [references/surfaces.md](references/surfaces.md) |

## Step 3 — Present the recommendation

Example:

> You already have a SvelteKit app with API routes, so we'll use **server mint** and `<Experience sessionEndpoint="…" />` from `@liforma/client/svelte`. The API key stays on the server; the browser only talks to your same-origin route.

Then implement using the matching reference — do not improvise mint HTTP shapes.

## Step 4 — Implement

Follow the reference guide. Shared principles for **every** path:

1. **Secrets stay on the server.** API keys never ship to the browser. If you see a key in client code, stop and restructure.
2. **Start from a known-good experience id** for demos: `exp_01EXAMPLES_COFFEE_BARISTA` (browser mint + allowlisted origin), or a project `exp_…` from [app.liforma.ai](https://app.liforma.ai).
3. **CDN is v2 only:** `https://cdn.liforma.ai/sdk/v2/client.js` — never v1.
4. **Prefer examples over greenfield structure:** https://github.com/LiformaLtd/examples.liforma.ai · gallery https://examples.liforma.ai  
   Preserve TypeScript, normal CSS (no Tailwind), and experience ids on app data.
5. **Audio unlock before `speak()`.** Call speak after start / user gesture (`onStarted` or equivalent).
6. **Align with shipped docs** for events and payloads — see Hard rules below.

### Portal checklist (before claiming done)

1. Project in [app.liforma.ai](https://app.liforma.ai)  
2. API key (server mint) **or** exact origin allowlisted (browser mint: scheme + host + port)  
3. Real `exp_…` the project can mint  

## Hard rules

- Prefer server mint when a backend exists; `/public-sessions` name is historical.
- Do not invent top-level TTS/avatar helpers outside `Experience`.
- Public speech is **`experience.speech.*` only** (`speak` / `play` / `createUtterance` / `interrupt`) — never invent `experience.speak` or provider-specific audio helpers.
- BYO voice (ElevenLabs, OpenAI, Deepgram, …): mint with `externalSpeechAudio`; stream PCM via `createUtterance` or play encoded/URL/MediaStreamTrack — see https://docs.liforma.ai/avatar-experiences/bring-your-own-voice
- `modeChange` payload is a bare string: `'listening' | 'speaking' | 'thinking'`.
- `ConversationMessage` uses `status: 'final'`, not `final: boolean`.
- Player `close` / `onStateUpdate` come from `attach()` / component props, not only `experience.on()`.
- Thumbnail = no session; Widget = light until expand — props may still iterate; trust current docs pages.

## What to consult

- https://docs.liforma.ai/llms.txt  
- https://docs.liforma.ai/getting-started/quick-start  
- https://docs.liforma.ai/avatar-experiences/bring-your-own-voice  
- https://docs.liforma.ai/avatar-experiences/experience-api  
- https://www.npmjs.com/package/@liforma/client  
- [references/server-mint.md](references/server-mint.md)  
- [references/browser-mint.md](references/browser-mint.md)  
- [references/surfaces.md](references/surfaces.md)  
- Skill `liforma-demo` if they want a runnable example first  
- Skill `liforma-debug` if mint / audio / iframe fails  
