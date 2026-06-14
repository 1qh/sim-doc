# state-management

## Decision

Layered state with strict per-layer ownership:

| Layer | Owns |
|---|---|
| URL fragment | Shareable + bookmarkable state (gzipped, base64url) |
| localStorage | Per-device preferences + local snapshot list |
| IndexedDB | Cached example/asset bundles |
| Zustand stores | Hot-loop sim state + ephemeral UI |
| React Query | Client-compute memoization cache (solver / minimization results) |
| React state | Local component state |
| sim-engine | Pure-function transformations (stateless) |

Per `STATE-MANAGEMENT.md`. No server layer — the app is pure client-side.

## Why zustand for hot-loop state

- Subscribe outside React render via `subscribeWithSelector` — frame-loop mutations don't thrash React
- Tiny bundle (~1 KB)
- No provider boilerplate
- Selector pattern matches the access discipline needed for `useFrame` loops
- Active maintenance + operator-stack precedent

## Rejected alternatives

- **Redux / Redux Toolkit** — heavier ceremony, no benefit over zustand for this scope
- **Jotai / Recoil** — atom-shaped state pattern doesn't fit sim-engine's whole-state shape
- **MobX** — proxy-based reactivity fights with R3F transient subscribes
- **Pure React (Context + useReducer)** — re-render thrash in hot loops

## Why React Query for client compute

- Heavy pure-function results (QM / Espresso minimization, datapath trace) memoized by canonical input key
- Cancellation + in-flight dedup for worker-offloaded compute via `AbortSignal`
- Stale-while-recompute UX without hand-rolled cache bookkeeping
- Cache is purely client-side compute memoization — no server queries exist

## Why URL state for sharing

- URL fragment is the natural permalink surface — the whole gzipped state lives in the hash per `adr/share-content-addressed.md`
- Browser back/forward works correctly
- Linkable / bookmarkable by definition
- Decodes fully client-side; payloads too large for a URL fragment are non-shareable (encoder tier `'oversize'`)

## Anti-patterns banned

- Persisting frame-loop state to localStorage (storage thrash)
- Storing URL-derivable state in zustand (duplication, drift)
- React state for cross-component sim state (lift to zustand)
- React Query holding ephemeral UI state (wrong layer — that is zustand)
- Caching impure / time-dependent values in React Query (key must be a canonical input)

## Caught by

- `tools/lint/state-layer.ts` enforces per-layer ownership patterns
- E2E: refresh-survives-URL-fragment, refresh-resets-zustand
- Smoke: localStorage writes blocked inside hot loop
