---
name: liforma-publish
description: >
  Create and publish Liforma Experiences from a server with @liforma/publisher.
  Use when: (1) programmatic authoring or CMS publish, (2) creating characters,
  clothes, hair, sets, or backdrops via API, (3) the user mentions Publisher SDK,
  lfm_live_ keys, /v1/projects, or "publish an experience from code", (4) updating
  first-party Publisher examples. Not for Session Launch embeds — use liforma-integrate.
license: MIT
metadata:
  author: liforma
  version: "1.0.0"
---

# Liforma Publisher (alpha)

Server-only authoring. Live project API key (`lfm_live_…`). Never import `@liforma/publisher` in a browser bundle.

```text
upload plates → backdrops / clothes / hair jobs → set + character → experience.publish
```

**Canonical:** https://docs.liforma.ai/_alpha/publisher-sdk  
**REST walkthrough:** https://docs.liforma.ai/_alpha/programmatic-experience-creation  
**OpenAPI:** https://docs.liforma.ai/_alpha/openapi/publisher.json  
**Example:** https://github.com/LiformaLtd/examples.liforma.ai/tree/main/_alpha/examples/programmatic-publish  
**Do not invent APIs.** `@liforma/publisher@0.5` is **namespaced**. There are no flat `createCharacter` / `createClothes` / `listAvatars` methods and no Place or Location SDK clients.

## Step 1 — Confirm this is authoring, not embed

| They want | Use |
|-----------|-----|
| Play / embed an existing `exp_…` | Skill `liforma-integrate` |
| Create wardrobe, set, character, then publish | This skill |
| Mint failed / silent avatar | Skill `liforma-debug` |

Need a project id + live API key from https://app.liforma.ai. Server env only: `LIFORMA_PROJECT_ID`, `LIFORMA_API_KEY`.

## Step 2 — Copy the hotel-check-in shape

Prefer the shipped example over a greenfield script:

```ts
import { createPublisher } from '@liforma/publisher';

const publisher = createPublisher(process.env.LIFORMA_PROJECT_ID!, {
  apiKey: process.env.LIFORMA_API_KEY!
});

const avatars = await publisher.avatars.list();
const avatar = avatars[0]!;

const background = await publisher.uploadImage(lobbyPng, { contentType: 'image/png' });
const backdrop = await publisher.backdrops.create({ name: 'Hotel lobby', image: background });
const set = await publisher.sets.create({ name: 'Hotel lobby', backdropId: backdrop.id });

const [clothes, hair] = await Promise.all([
  publisher.clothes.create({ avatarId: avatar.id, image: clothesImage, backgroundMode: 'remove' }),
  publisher.hair.create({ avatarId: avatar.id, image: hairImage, backgroundMode: 'remove' })
]);

const character = await publisher.characters.create({
  avatarId: avatar.id,
  name: 'Front desk',
  voice: avatar.defaultVoiceId,
  sttLang: avatar.defaultSttLang,
  clothesId: clothes.id,
  hairId: hair.id
});

const experience = await publisher.experiences.create({
  title: 'Hotel check-in',
  characterId: character.id,
  setId: set.id,
  publish: true
});
```

Namespaces:

```text
publisher.avatars.list()
publisher.backdrops.startCreate / create / get / archive / restore / delete
publisher.sets.*
publisher.clothes.*
publisher.hair.*
publisher.characters.*
publisher.experiences.create / get / update / archive / restore / delete / publish
publisher.jobs.get / wait / watch / retry
```

## Step 3 — Jobs and options

Job-backed resources (backdrops, clothes, hair): `create()` = `startCreate()` → `jobs.wait` → `resource.get(targetId)`.

- `jobs.wait()` returns the **job**, not the resource.
- `jobs.watch()` is an async iterator. It does not take `onProgress`.
- Authored input is the first argument. `{ signal, timeoutMs, onProgress }` is the second (`onProgress` only on `wait` / `create`).
- Abort stops local waiting. It does **not** cancel a durable job.
- Retry only when the server said `error.retryable === true`. Do not infer retryability from codes.
- `forceNew: true` skips content-addressed reuse.

```ts
const started = await publisher.backdrops.startCreate({ name: 'Hotel lobby', image: background });
const completed = await publisher.jobs.wait(started.id);
const backdrop = await publisher.backdrops.get(completed.targetId);
```

## Hard rules

- REST stays project-scoped: `/v1/projects/{projectId}/…`.
- Experience create/update input uses **`setId`** (not `placeId`). Backdrop ids are `bdrop_…`.
- Clothes + hair are composite layers. Do not invent `publisher.costumes` or `costumeId` until docs ship them.
- `experiences.update` writes the draft. `experiences.publish` snapshots a revision. `publish: true` on create is allowed.
- Prefer `status`, `hasPublishedRevision`, `hasUnpublishedChanges`. Treat `published` as a deprecated 0.x alias.
- `delete()` is after archive only and returns `{ deleted: true, id }`.
- Job errors are `{ code, category, retryable, message }`.
- Align snippets with docs `publisherHotelCheckIn` / `publisherJobs` — do not reintroduce flat SDK methods.

## What to consult

- https://docs.liforma.ai/llms.txt  
- https://docs.liforma.ai/_alpha/publisher-sdk  
- https://docs.liforma.ai/_alpha/programmatic-experience-creation  
- https://docs.liforma.ai/_alpha/openapi/publisher.json  
- npm: [`@liforma/publisher`](https://www.npmjs.com/package/@liforma/publisher) — use the namespaced **0.5** surface in docs, even if an older tag is still cached locally
- Skill `liforma-integrate` once they have an `exp_…` to embed  
