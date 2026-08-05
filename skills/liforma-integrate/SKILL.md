---
name: liforma-integrate
description: Integrate Liforma avatar experiences into a web app. Use when embedding Experience, minting sessions, choosing Svelte/React/Next/vanilla, ExperienceThumbnail/Widget, or building from Liforma examples.
---

# Liforma integration

Liforma is an **Avatar Experience Platform**. Integrators compose Experiences — not raw STT/LLM/TTS APIs.

```text
Experience (exp_…) → Session Manifest → @liforma/client
```

**Canonical index:** https://docs.liforma.ai/llms.txt  
**Do not invent APIs** (e.g. no `Liforma.textToSpeech()`). Use `Experience` + modes / `speak()`.

## Step 1 — Discover the host stack

Scan the codebase before asking the user:

- Framework: SvelteKit / Svelte, React, Next.js App Router, or vanilla HTML
- Existing auth / API routes (session mint BFF?)
- Whether a backend can hold an API key securely

## Step 2 — Recommend mint path

| Situation | Path |
|-----------|------|
| Backend available | Prefer **server mint**: `POST /v1/sessions` with API key → same-origin `sessionEndpoint` on `<Experience>` |
| Client-only / demos | **Browser mint**: allowlist origin in [app.liforma.ai](https://app.liforma.ai) → SDK calls `POST /v1/public-sessions` (name is historical) |

Never put API keys in browser code.

Docs: [Server sessions](https://docs.liforma.ai/avatar-experiences/server-sessions) · [Browser embeds](https://docs.liforma.ai/avatar-experiences/browser-embeds)

## Step 3 — Choose SDK surface

| Host | Install / import |
|------|------------------|
| Svelte / SvelteKit | `npm install @liforma/client` → `@liforma/client/svelte` |
| React | `npm install @liforma/client` → `@liforma/client/react` |
| Next.js | `npm install @liforma/client` → `@liforma/client/next` (+ `createLiformaSessionRouteHandler`) |
| Vanilla / no bundler | CDN `https://cdn.liforma.ai/sdk/v2/client.js` + `<liforma-experience>` |

Also available (props may still iterate — follow current docs):

- `<ExperienceThumbnail />` — gallery cards, no session
- `<ExperienceWidget />` / `<liforma-experience-widget>` — corner launcher

npm: https://www.npmjs.com/package/@liforma/client  
CDN must be **v2** (`/sdk/v2/client.js`), never v1.

## Step 4 — Hello world

Public demo experience id (browser mint + allowlisted origin):

`exp_01EXAMPLES_COFFEE_BARISTA`

### Svelte

```svelte
<script>
  import { Experience } from '@liforma/client/svelte';
</script>

<Experience experienceId="exp_01EXAMPLES_COFFEE_BARISTA" />
```

### React

```tsx
import { Experience } from '@liforma/client/react';

export function Demo() {
  return <Experience experienceId="exp_01EXAMPLES_COFFEE_BARISTA" />;
}
```

### Vanilla

```html
<script src="https://cdn.liforma.ai/sdk/v2/client.js" defer></script>
<liforma-experience experience-id="exp_01EXAMPLES_COFFEE_BARISTA"></liforma-experience>
```

Authenticated apps add `sessionEndpoint="/api/liforma-session"` (or Next helper). See [Quick start](https://docs.liforma.ai/getting-started/quick-start).

## Step 5 — Prefer a matching example

Fork / adapt from https://github.com/LiformaLtd/examples.liforma.ai rather than inventing structure. Gallery: https://examples.liforma.ai

Preserve: Experience embed, TypeScript, normal CSS (no Tailwind), experience ids on app data (not secrets in `.env` unless deployment requires it).

## Step 6 — Portal checklist

1. Create/select project at https://app.liforma.ai  
2. API key for server mint **or** allowlisted origins for browser mint  
3. Use a real `exp_…` id the project can mint  

## Hard rules

- Prefer server mint when a backend exists.
- Do not invent top-level TTS/avatar helpers outside `Experience`.
- `modeChange` payload is a bare string: `'listening' | 'speaking' | 'thinking'`.
- `ConversationMessage` uses `status: 'final'`, not `final: boolean`.
- Player `close` / `onStateUpdate` come from `attach()` / component props, not only `experience.on()`.
- Deep detail: docs URLs above — keep generated code aligned with shipped docs, not guesses.
