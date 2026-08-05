# Liforma Agent Skills

Procedural skills so coding agents integrate [Liforma](https://www.liforma.ai) correctly on the first attempt.

Works with Cursor, Claude Code, Codex, and other agents that support the [Agent Skills](https://agentskills.io/specification) format.

## Install

```bash
npx skills add LiformaLtd/agent-skills
```

Or a single skill:

```bash
npx skills add LiformaLtd/agent-skills --skill liforma-integrate
npx skills add LiformaLtd/agent-skills --skill liforma-debug
npx skills add LiformaLtd/agent-skills --skill liforma-demo
```

## Skills

| Skill | When to use |
|-------|-------------|
| `liforma-demo` | Try / clone a curated example before building from scratch |
| `liforma-integrate` | Embed an Experience: mint path, Svelte/React/Next/vanilla, Thumbnail/Widget |
| `liforma-debug` | Silent avatar, auth/CORS, audio unlock, wrong CDN URL, mint failures |

`liforma-integrate` loads detail from `skills/liforma-integrate/references/` (server mint, browser mint, surfaces) so the main skill stays short.

## Quick reference

| Task | Skill |
|------|-------|
| "Show me a Liforma demo" | `liforma-demo` |
| Corner chat widget | `liforma-demo` → experience-widget, or integrate → Widget |
| Lesson / tutor app | `liforma-demo` → spanish-tutor / guided-practice |
| Production embed with API key | `liforma-integrate` → server mint |
| Static site / allowlisted origin | `liforma-integrate` → browser mint |
| Avatar silent / 403 on mint | `liforma-debug` |

## Example prompts

```
"What Liforma examples can I run locally?"

"Add a Liforma avatar to my SvelteKit app"

"Embed exp_… with a server session endpoint in Next.js"

"My public-sessions mint returns 403"

"Clone the basic-embed example and point it at my experience id"
```

## Canonical docs (do not invent APIs)

- https://docs.liforma.ai/llms.txt
- https://docs.liforma.ai/getting-started/quick-start
- https://examples.liforma.ai · https://github.com/LiformaLtd/examples.liforma.ai
- Developer portal: https://app.liforma.ai
- npm: [`@liforma/client`](https://www.npmjs.com/package/@liforma/client)

```text
Experience (exp_…) → Session Manifest → @liforma/client
```

## License

MIT
