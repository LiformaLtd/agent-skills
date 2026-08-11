# Server mint (API key)

**When:** The host app has a backend that can hold secrets.

**Flow:**

```text
Browser  →  same-origin sessionEndpoint (POST)
         →  your server mints with API key
         →  POST https://api.liforma.ai/v1/sessions
         →  Session Launch { session, launch } back to SDK
         →  player.liforma.ai/embed
```

`launch` is opaque — forward it; do not parse it.

## Prerequisites

- API key from [app.liforma.ai](https://app.liforma.ai) (server-only env var)
- An `experienceId` (`exp_…`) owned/allowed for that project
- Same-origin route the browser can `POST` to (no CORS fight)

## Implementation outline

1. Install `npm install @liforma/client`.
2. Add a server route that accepts `{ experienceId, language? }`, calls `POST /v1/sessions` with the API key, returns the Session Launch JSON.
3. Next.js App Router: prefer `createLiformaSessionRouteHandler` from `@liforma/client/next` (add real `authorize()` in production).
4. Client:

```svelte
<Experience
  experienceId="exp_…"
  sessionEndpoint="/api/liforma-session"
/>
```

(or React / imperative `Experience.startSession` + `attach` — same contract)

## Gotchas

- Returning HTML (login page) or an error string from `sessionEndpoint` breaks the SDK — return Session Launch JSON or a clear JSON error.
- Do not put the API key in `PUBLIC_*` / `VITE_*` / client bundles.
- Authenticated products should reject unauthenticated mint in production (`allowUnauthenticated` is for demos only).

## Docs

- https://docs.liforma.ai/avatar-experiences/server-sessions  
- https://docs.liforma.ai/avatar-experiences/nextjs  
- https://docs.liforma.ai/api-reference/sessions  
- OpenAPI: https://docs.liforma.ai/openapi/sessions.json  
