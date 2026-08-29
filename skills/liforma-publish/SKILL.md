---
name: liforma-publish
description: >
  Create and publish Liforma Experiences from a server with @liforma/publisher.
  Use when: (1) programmatic authoring or CMS publish, (2) creating characters,
  costumes, clothes, hair, sets, or backdrops via API, (3) the user mentions
  Publisher SDK, lfm_live_ keys, /v1/projects, or "publish an experience from
  code", (4) updating first-party Publisher examples. Not for Session Launch
  embeds — use liforma-integrate.
license: MIT
metadata:
  author: liforma
  version: "1.2.0"
---

# Liforma Publisher (alpha)

Server-only authoring. Live project API key (`lfm_live_…`). Never import `@liforma/publisher` in a browser bundle.

```text
prefer: experiences.createFrom(images + copy)
or step-by-step: upload plates → backdrops / costumes|clothes+hair → set + character → experience.publish
```

**Canonical:** https://docs.liforma.ai/_alpha/publisher-sdk  
**REST walkthrough:** https://docs.liforma.ai/_alpha/programmatic-experience-creation  
**OpenAPI:** https://docs.liforma.ai/_alpha/openapi/publisher.json  
**Example:** https://github.com/LiformaLtd/examples.liforma.ai/tree/main/_alpha/examples/programmatic-publish  
**Do not invent APIs.** `@liforma/publisher@0.7` is **namespaced**. Prefer `experiences.createFrom` for one-shot composition. There are no flat `createCharacter` / `createClothes` / `listAvatars` methods and no Place or Location SDK clients.

## Step 1 — Confirm this is authoring, not embed

| They want | Use |
|-----------|-----|
| Play / embed an existing `exp_…` | Skill `liforma-integrate` |
| Create wardrobe, set, character, then publish | This skill |
| Mint failed / silent avatar | Skill `liforma-debug` |

Need a project id + live API key from https://app.liforma.ai. Server env only: `LIFORMA_PROJECT_ID`, `LIFORMA_API_KEY`.

## Step 2 — Prefer createFrom

```ts
import { readFileSync } from 'node:fs';
import { createPublisher } from '@liforma/publisher';

const publisher = createPublisher(process.env.LIFORMA_PROJECT_ID!, {
  apiKey: process.env.LIFORMA_API_KEY!
});

const avatars = await publisher.avatars.list();
const avatar = avatars[0]!;

const result = await publisher.experiences.createFrom({
  title: 'Hotel check-in',
  backdrop: {
    image: { bytes: readFileSync('./lobby.png'), contentType: 'image/png' }
  },
  character: {
    avatarId: avatar.id,
    name: 'Front desk',
    voice: avatar.defaultVoiceId,
    sttLang: avatar.defaultSttLang,
    personality: 'Friendly hotel receptionist'
  },
  startingMessage: 'Good evening. How may I help you?',
  systemInstructions: 'The customer is checking in.',
  publish: true
});

console.log(result.experience.id, result.created);
```

`ImageSource`: `Blob` | `{ bytes, contentType }` | `{ url }` (copy into Liforma — never a permanent URL dependency) | `{ uploadId }`. Reuse with `{ id }` on backdrop/character/plates. Errors keep the underlying `code` and add `context.step`.

## Step 3 — Or copy a low-level shipped shape

Prefer shipped examples over a greenfield script. Use **either** a whole costume **or** composite clothes + hair — never both on one character.

### Whole look (`costumeId`)

One full-body plate under `costumes/whole/`:

```ts
import { createPublisher } from '@liforma/publisher';

const publisher = createPublisher(process.env.LIFORMA_PROJECT_ID!, {
  apiKey: process.env.LIFORMA_API_KEY!
});

const avatars = await publisher.avatars.list();
const avatar = avatars[0]!;
// avatars.list() returns costumes[], clothes[], and hair[] separately (0.6+)

const background = await publisher.uploadImage(lobbyPng, { contentType: 'image/png' });
const backdrop = await publisher.backdrops.create({ name: 'Exam room', image: background });
const set = await publisher.sets.create({ name: 'Exam room', backdropId: backdrop.id });

const costume = await publisher.costumes.create({
  avatarId: avatar.id,
  image: wholeLookImage,
  backgroundMode: 'remove'
});

const character = await publisher.characters.create({
  avatarId: avatar.id,
  name: 'Examiner',
  voice: avatar.defaultVoiceId,
  sttLang: avatar.defaultSttLang,
  costumeId: costume.id
});

const experience = await publisher.experiences.create({
  title: 'Language exam',
  characterId: character.id,
  setId: set.id,
  publish: true
});
```

### Composite look (`clothesId` + `hairId`)

Layer plates under `costumes/composite/…`:

```ts
const [clothes, hair] = await Promise.all([
  publisher.clothes.create({
    avatarId: avatar.id,
    image: clothesImage,
    backgroundMode: 'remove'
  }),
  publisher.hair.create({
    avatarId: avatar.id,
    image: hairImage,
    backgroundMode: 'remove'
  })
]);

const character = await publisher.characters.create({
  avatarId: avatar.id,
  name: 'Front desk',
  voice: avatar.defaultVoiceId,
  sttLang: avatar.defaultSttLang,
  clothesId: clothes.id,
  hairId: hair.id
});
```

Namespaces:

```text
publisher.avatars.list()
publisher.backdrops.startCreate / create / get / archive / restore / delete
publisher.sets.*
publisher.costumes.*
publisher.clothes.*
publisher.hair.*
publisher.characters.*
publisher.experiences.create / get / update / archive / restore / delete / publish
publisher.jobs.get / wait / watch / retry
```

## Step 3 — Jobs and options

Job-backed resources (backdrops, costumes, clothes, hair): `create()` = `startCreate()` → `jobs.wait` → `resource.get(targetId)`.

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
- Whole looks use `publisher.costumes` + character `costumeId`. Composite looks use `publisher.clothes` + `publisher.hair` with `clothesId` / `hairId`. Do not combine `costumeId` with layer ids.
- `experiences.update` writes the draft. `experiences.publish` snapshots a revision. `publish: true` on create is allowed.
- Prefer `status`, `hasPublishedRevision`, `hasUnpublishedChanges`. Treat `published` as a deprecated 0.x alias.
- `delete()` is after archive only and returns `{ deleted: true, id }`.
- Job errors are `{ code, category, retryable, message }`.
- Align snippets with docs `publisherHotelCheckIn` / `publisherHotelExaminer` / `publisherJobs` — do not reintroduce flat SDK methods.
- Install `@liforma/publisher@^0.6.0` for costumes. Stay on 0.5 only against an API that does not emit `avatars.list().costumes`.

## What to consult

- https://docs.liforma.ai/llms.txt  
- https://docs.liforma.ai/_alpha/publisher-sdk  
- https://docs.liforma.ai/_alpha/programmatic-experience-creation  
- https://docs.liforma.ai/_alpha/openapi/publisher.json  
- npm: [`@liforma/publisher`](https://www.npmjs.com/package/@liforma/publisher) — namespaced **0.6** surface (`publisher.costumes`, character `costumeId`)
- Skill `liforma-integrate` once they have an `exp_…` to embed  
