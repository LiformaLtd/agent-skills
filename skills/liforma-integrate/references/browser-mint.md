# Browser mint (origin allowlist)

**When:** Client-only apps, static sites, or quick demos with no secret-holding backend.

**Flow:**

```text
Browser (@liforma/client)
  → POST https://api.liforma.ai/v1/browser-sessions
  → Session Launch { session, launch } (if Origin is allowlisted)
  → player.liforma.ai/embed
```

`launch` is opaque — do not parse it. Prefer [server-mint.md](./server-mint.md) (`sessionEndpoint`) for authenticated products.

## Prerequisites

- Exact page origin allowlisted in [app.liforma.ai](https://app.liforma.ai) → project → Origins  
  Include scheme + host + port (`http://localhost:4001` ≠ `http://localhost:5173`).
- An `experienceId` the project can mint for that origin.

## Demo experience

For hello-world against public examples (when your origin is allowlisted for demos):

`exp_01EXAMPLES_COFFEE_BARISTA`

Meet gallery: https://www.liforma.ai/meet

## Implementation outline

Framework:

```svelte
<script>
  import { Experience } from '@liforma/client/svelte';
</script>

<Experience experienceId="exp_01EXAMPLES_COFFEE_BARISTA" />
```

Vanilla:

```html
<script src="https://cdn.liforma.ai/sdk/v2/client.js" defer></script>
<liforma-experience experience-id="exp_01EXAMPLES_COFFEE_BARISTA"></liforma-experience>
```

## Gotchas

- 403 on mint → origin missing or mismatched (trailing slash / http vs https / wrong port).
- Never “fix” browser mint by pasting an API key into frontend code — switch to [server-mint.md](./server-mint.md).
- CDN script must be **v2**: `https://cdn.liforma.ai/sdk/v2/client.js`.

## Docs

- https://docs.liforma.ai/avatar-experiences/browser-embeds  
- https://docs.liforma.ai/api-reference/browser-sessions  
- https://docs.liforma.ai/getting-started/quick-start  
