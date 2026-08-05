# SDK surfaces and hello world

npm: https://www.npmjs.com/package/@liforma/client  

| Host | Package entry | Notes |
|------|---------------|--------|
| Svelte / SvelteKit | `@liforma/client/svelte` | `<Experience />`, `<ExperienceThumbnail />`, `<ExperienceWidget />` |
| React | `@liforma/client/react` | Same product surfaces |
| Next.js App Router | `@liforma/client/next` | Client components + `createLiformaSessionRouteHandler` |
| JS API | `@liforma/client` | `Experience.startSession`, `attach`, types |
| Vanilla / no bundler | CDN IIFE | `https://cdn.liforma.ai/sdk/v2/client.js` → `<liforma-experience>` |

Optional peers: `svelte` ^5; `react` / `react-dom` ^18 \|\| ^19.

## Hello world (browser mint)

```svelte
<script>
  import { Experience } from '@liforma/client/svelte';
</script>

<Experience experienceId="exp_01EXAMPLES_COFFEE_BARISTA" />
```

```tsx
import { Experience } from '@liforma/client/react';

export function Demo() {
  return <Experience experienceId="exp_01EXAMPLES_COFFEE_BARISTA" />;
}
```

```html
<script src="https://cdn.liforma.ai/sdk/v2/client.js" defer></script>
<liforma-experience experience-id="exp_01EXAMPLES_COFFEE_BARISTA"></liforma-experience>
```

## Related components

| Component | Role |
|-----------|------|
| `Experience` / `<liforma-experience>` | Full conversation / presenter session |
| `ExperienceThumbnail` | Gallery / card preview — **no session** |
| `ExperienceWidget` / `<liforma-experience-widget>` | Corner launcher; light until expand |

Docs:

- https://docs.liforma.ai/avatar-experiences/svelte  
- https://docs.liforma.ai/avatar-experiences/react  
- https://docs.liforma.ai/avatar-experiences/nextjs  
- https://docs.liforma.ai/avatar-experiences/experience-thumbnail  
- https://docs.liforma.ai/avatar-experiences/experience-widget  
- https://docs.liforma.ai/avatar-experiences/experience-api  
