# share-content-addressed

## Decision

Fragment-only share. State canonicalized + compressed + base64url-encoded into the URL hash. The whole shareable state rides in the URL; there is no backend and no server round-trip. Payloads too large for a URL fragment are non-shareable by design — the encoder labels them tier `'oversize'`.

## Tiers

| Compressed payload | Tier | Result |
|---|---|---|
| Fits a URL fragment | `'fragment'` | `/s#<base64url-payload>` — fully self-contained link |
| Exceeds URL-fragment budget | `'oversize'` | not shareable; UI surfaces "too large to share by link" |

- Zero backend round-trip for save or load
- Permalink works offline
- No infrastructure cost at any scale
- No server fallback for oversize — the link is the only carrier

## Encode

```
state → sim-engine.canonicalize → CompressionStream("deflate-raw") → base64url → /s#<payload>
```

## Canonicalization

Per `packages/sim-engine` codec:
- Object keys sorted deterministically via `safe-stable-stringify`
- Numbers in shortest exact representation (integer state preferred; banned: `NaN`, `Infinity`, `-0`)
- Arrays untouched (order is part of the state)
- `Set` / `Map` / `BigInt` banned in canonical-path Zod schemas (safe-stable-stringify silently drops Sets/Maps; BigInt lossily coerces)
- Schema-version byte prefix (cross-version replay-safe)
- Canonical bytes are the codec output; compression is transport-only

## Compression for transport

- `CompressionStream("deflate-raw")` — no header / no checksum / no mtime → byte-identical across Chrome / Safari / Firefox / Node 21+ / Bun. Universal browser support.
- `fzstd` / `pako` / `fflate` / `lz-string` all banned per pm4ai compression policy.

## Oversize tier

The encoder measures the base64url length against the URL-fragment budget. Over budget → tier `'oversize'`, the encoder returns no link, and the UI explains the state is too large to share by URL. There is no server to fall back to.

## Schema versioning

Snapshot body carries `{ v: <int>, ... }`. `sim-engine` keeps a deserializer for every published `v`. Old links forever-replay-able. New `v` lands via expand-contract — add new field, keep old shape working through one version, drop old in next major.

## Read path

```
GET /s#<payload>
  → app loads (static, CDN-cached bundle)
  → base64url-decode → DecompressionStream → sim-engine.deserialize → state
```

The fragment never reaches the network — it is decoded entirely client-side after the static app loads.

## Determinism guarantee

Same `(MIPS program, K-map exercise, sim state)` → identical canonical bytes → identical fragment. Substrate codec is the SSOT for this property; property-test enforces idempotence.

## Pitfall

- A permalink carries no random ID — the URL fragment IS the whole state, canonicalized then encoded, so identical state always yields an identical permalink and any random ID would break round-trip equality; states too large to encode are non-shareable by design (`'oversize'` tier).

## Caught by

- Codec round-trip property test (`fast-check`) — `deserialize(serialize(state)) === state` for any state.
- Codec stability test — encoded fragment of fixed test inputs is committed; any change to canonicalization that shifts the output fails CI.
- Tier-threshold property test — states bordering the fragment budget exercise both `'fragment'` and `'oversize'`.
- Schema-version migration test — old fixtures deserialize correctly under every published `v`.
