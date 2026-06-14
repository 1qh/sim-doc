# offline-pwa

## Decision

Service worker registered on first visit. Caches app shell + sim assets + visited example bundles. The app is pure client-side — every sim, every share decode, and the local snapshot list run entirely in the browser, so the whole app is fully offline-capable after first load.

## Why

- Learning tool — users sometimes work offline (plane, transit, low-connectivity environment)
- Sim is fundamentally client-side (3D + state machine), every fact computed in-browser
- Asymmetric UX win for low cost
- Aligns with "world-class endpoint" — premium web apps work offline

## Service worker shape

- Tool: **Serwist** (`@serwist/next`) — `next-pwa` is dead (last meaningful release 2022). Serwist is Workbox-spirit with first-class App Router + Next 16 support, TypeScript service workers, and revision-keyed precache injection.
- Caching strategy:
  - **App shell** (HTML, JS, CSS): `StaleWhileRevalidate`
  - **3D assets** (none today, but if any HDRI / fonts): `CacheFirst`, immutable
  - **Examples** (`/api/examples/[slug]`): `StaleWhileRevalidate`
  - **Fonts** (Latin subset): `CacheFirst`, immutable
- Versioning: build-time hash for cache keys; new build invalidates old
- Update flow: subtle "new version available, reload" toast (single, dismissible)

## Fully offline by construction

The app holds zero server dependency at runtime:
- Shares decode from the URL fragment in-browser (gzip via native `DecompressionStream`, base64url decode) — no roundtrip ever.
- `/me` reads the local snapshot list straight from `localStorage` — works offline.
- Sims (3D + state machine) compute every value client-side.

## What requires online

- Initial load of the app shell + an unseen example bundle not yet cached.

Everything else continues functioning offline.

## PWA manifest

`manifest.json` with:
- Name + description (domain language, no school refs)
- Icons (procedural SVG → PNG variants at build time)
- Theme color matching design tokens
- Display: `standalone`
- Start URL: `/`
- Scope: `/`

Installable to home screen on supported browsers.

## Banned

- Workbox precaching that exceeds 50 MB total (cache budget)
- Aggressive prefetch of unvisited routes
- Notifications API (no push notifications in floor)

## Pitfall

- Background Sync API is Chromium-only — the primary offline-replay path is the `online` event + boot-time replay; SW background-sync is a bonus where available.

## Caught by

- Service-worker smoke: page works offline after first online visit
- Cache-size budget test
- Offline-share smoke: share link decodes from URL fragment with network disabled
- Offline-`/me` smoke: snapshot list renders from localStorage with network disabled
- Update-flow smoke: deploy new version → toast appears → reload → new version active
