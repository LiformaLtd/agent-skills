---
name: liforma-demo
description: >
  Spin up or adapt a curated Liforma example — basic embed, experience widget, Spanish Tutor,
  guided practice, or speak playground. Use when the user wants to try Liforma, see a demo,
  clone an example, get something running before writing a custom integration, or asks what
  examples exist. Prefer this before greenfield when they only need a working reference.
license: MIT
metadata:
  author: liforma
  version: "1.0.0"
---

# Liforma demos / examples

Curated runnable apps live in **https://github.com/LiformaLtd/examples.liforma.ai**  
Gallery: **https://examples.liforma.ai**  
Live Meet avatars: **https://www.liforma.ai/meet**

This skill chooses an example and drives clone/adapt. Deep integration rules still come from `liforma-integrate` and https://docs.liforma.ai/llms.txt.

## What you cannot verify alone — say this up front

| Moment | Why it's human-only |
|--------|---------------------|
| Portal API key / origin allowlist | Account secrets and dashboard clicks |
| Browser mic permission | OS/browser prompt |
| Confirming the avatar spoke / lip-synced | Needs human eyes and ears |
| Production deploy auth (Vercel, etc.) | Interactive login |

Do the mechanical work (clone, install, point at the right folder). Do not claim the avatar "works" until the user confirms in a browser.

## Step 1 — Pick an example

Show this table unless they already named one:

| # | Example | Kind | Best when | Frameworks (repo) | Local port |
|---|---------|------|-----------|-------------------|------------|
| 1 | **basic-embed** | embed | Fastest hello world | SvelteKit, vanilla | 4001 |
| 2 | **experience-widget** | widget | Corner launcher / support-style UI | vanilla | 4002 |
| 3 | **spanish-tutor** | lessons | Multi-lesson app, close-before-switch | SvelteKit, vanilla | 4003 |
| 4 | **guided-practice** | presenter | `speak()` + manual listen / tutor lines | SvelteKit, vanilla, Next, React Vite | 4004 |
| 5 | **speak-playground** | presenter | Experiment with speak API | (see repo gallery) | 4005 |

Routing:

- "Just show me anything" → **1**  
- "Floating chat widget" → **2**  
- "Lessons / education app" → **3**  
- "Scripted tutor turns / speak API" → **4** or **5**  
- Evaluating Meet only (no code) → send them to https://www.liforma.ai/meet and stop

## Step 2 — Get credentials / origins early

For **browser mint** examples they need their origin allowlisted (or use documented demo origins).  
For **server mint** adaptations they need an API key in **server** env only.

Ask once:

> For a local run, allowlist `http://localhost:<port>` in https://app.liforma.ai (Origins), or tell me if you already have a server mint route / API key.

Never write a fake API key into a client env file.

## Step 3 — Clone and open the right folder

```bash
git clone https://github.com/LiformaLtd/examples.liforma.ai.git
cd examples.liforma.ai
```

Work in `examples/<slug>/` for the chosen stack (e.g. `examples/basic-embed/sveltekit`). Follow that folder's README / `spec.md`.

Repo root `./start` can run the gallery + examples — see examples `README.md` / `LOCAL_DEV.md`.

## Step 4 — Preserve vs adapt

**Preserve:**

- `@liforma/client` / `<Experience />` (or documented web component)
- TypeScript + normal CSS (no Tailwind)
- Experience ids on lesson/app data (not secrets in `.env` unless deploy truly needs it)
- Close-before-switch for multi-experience UIs

**Adapt:** branding, copy, experience ids, surrounding chrome.

If they outgrow the example, hand off to skill **`liforma-integrate`** for mint-path choice and production hardening.

## Step 5 — Done criteria

- App starts on the expected local port without secret leakage  
- User confirmed avatar audio/video in the browser  
- Origins or server mint configured for their environment  

If mint/audio fails → **`liforma-debug`**.
