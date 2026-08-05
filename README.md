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
```

## Skills

| Skill | When to use |
|-------|-------------|
| `liforma-integrate` | Embed an avatar experience, choose mint path, pick Svelte / React / Next / vanilla |
| `liforma-debug` | Silent avatar, auth/CORS errors, audio unlock, wrong CDN URL, mint failures |

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
