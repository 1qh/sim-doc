# bun-first-discipline

Server-side, scripts, and tools rely on official Bun APIs as much as possible. Per operator instruction + pm4ai ALLOWED_STACK + `BUN_GLOBALS` map.

## Server-side + scripts (Bun runtime)

| Concern | Bun API |
|---|---|
| File read | `Bun.file(path).text()` / `.json()` / `.bytes()` / `.stream()` |
| File write | `Bun.write(path, data)` |
| Globbing | `Bun.Glob('**/*.mdx').scan({ cwd })` |
| Shell | `Bun.$\`command\`` (tagged template) |
| Spawn | `Bun.spawn(['cmd', 'arg'])` / `Bun.spawnSync` |
| Env | `Bun.env.VAR` (instead of `process.env`) |
| HTTP server | `Bun.serve({ fetch })` — n/a since Next handles, but used in tools |
| Fetch (server) | `Bun.fetch` — Bun's enhanced fetch with HTTP/3, decompression, etc. |
| Compression server | `Bun.zstdCompress(bytes)` / `Bun.zstdDecompress(bytes)` / `.zstdCompressSync` / `Bun.gzipSync` / `Bun.gunzipSync` / `Bun.deflateSync` / `Bun.inflateSync` |
| Archive | `Bun.Archive` (tar) — n/a unless we archive |
| Cookies | `Bun.Cookie` / `Bun.CookieMap` |
| Crypto hash (md5/sha1/sha256/sha512) | `Bun.CryptoHasher` / `Bun.SHA256` / `Bun.SHA512` etc. |
| Fast non-crypto hash | `Bun.hash(data)` (wyhash) |
| HTML escape | `Bun.escapeHTML` |
| UUIDs | `crypto.randomUUID()` (native) or `Bun.randomUUIDv7()` for time-ordered |
| JSON variants | `Bun.JSON5` / `Bun.JSONC` / `Bun.JSONL` |
| TOML / YAML | `Bun.TOML.parse` / `Bun.YAML.parse` |
| Markdown | `Bun.markdown(source)` — for server-side rendering when remark pipeline overkill |
| Module resolve | `Bun.resolve(spec, importer)` / `Bun.resolveSync` |
| Process exec | `Bun.which('command')` |
| Sleep | `Bun.sleep(ms)` / `Bun.sleepSync(ms)` (scripts only; never in product hot paths) |
| Cron | `Bun.cron({ ... })` — for any scheduled tooling |
| Inspect | `Bun.inspect(obj)` for debug output |
| Heap snapshot | `Bun.generateHeapSnapshot()` for memory diagnostics |
| Equality deep | `Bun.deepEquals(a, b)` |
| Match deep | `Bun.deepMatch(value, pattern)` |
| Semver compare | `Bun.semver.satisfies` / `.order` |
| Color in CLI | `Bun.color({ ... })` |
| Stream → text/bytes/JSON | `Bun.readableStreamToText/Bytes/JSON/Blob/...` |
| Terminal width helpers | `Bun.stringWidth` / `Bun.stripANSI` / `Bun.sliceAnsi` / `Bun.wrapAnsi` |
| Nanosecond timing | `Bun.nanoseconds()` |
| Plugin loader | `Bun.plugin({ name, setup })` |
| FFI | `Bun.FFI` — n/a |
| Test runner | `bun:test` — for everything testable |

## What Bun does NOT cover for this project

These stay on dedicated libs:

| Concern | Library |
|---|---|
| Boolean expression parsing | `chevrotain` |
| Fuzzy search | `mini-search` |
| Snapshot canonicalize | `safe-stable-stringify` |
| State diff | `microdiff` |
| GPU detection | `detect-gpu` |
| Color science (OKLCH palette derivation) | `culori` |
| Toposort | `toposort` |
| MDX code highlighting | `shiki` |
| 3D engine | `three`, `@react-three/fiber`, `@react-three/drei`, `@react-three/uikit`, `postprocessing` |
| Editor | `monaco-editor` |
| State | `zustand` |
| Validation | `zod` |
| Animation | `motion` |
| Components | `shadcn` (via cli sync) + `cnsync` |
| Icons | `lucide-react` |
| Tables / virtual / forms / dnd | `@tanstack/react-table` / `@tanstack/react-virtual` / `@tanstack/react-form` / `@dnd-kit` |
| Themes | `next-themes` |
| Logging | `consola` |
| Utilities | `es-toolkit` |

## Banned

- `process.env` in any server / script / tool — use `Bun.env`
- `fs-extra` / `graceful-fs` / `mkdirp` / `rimraf` — use `node:fs` or Bun.file/write
- `dotenv` — use Bun's native `.env` auto-loading
- `execa` / `shelljs` / `zx` — use `Bun.$`
- `cross-env` — use Bun's native env
- `node-fetch` / `axios` — use native or `Bun.fetch`
- `nanoid` / `uuid` — use `crypto.randomUUID()` / `Bun.randomUUIDv7`
- `chokidar` — use `node:fs.watch`
- `chalk` / `picocolors` / `kleur` — use `Bun.color`
- `glob` / `fast-glob` / `tinyglobby` — use `Bun.Glob`
- `js-yaml` / `yaml` — use `Bun.YAML`
- `@iarna/toml` — use `Bun.TOML`
- `js-sha256` / `md5` / `crypto-js` — use `Bun.CryptoHasher`

## Client-side caveat

Bun.* APIs are server/script-only. Client code uses native Web APIs:

- `fetch` (native) — not `Bun.fetch`
- `crypto.randomUUID()` (native, also available in Bun)
- `crypto.subtle` (WebCrypto) for AES / SHA on browser
- `CompressionStream` / `DecompressionStream` (native) for zstd / gzip
- `structuredClone` (native) for deep-copy
- `Intl.*` (native) for locale-aware formatting

Substrate packages target both runtimes — pure-TS code that runs on Bun or browser interchangeably (no Bun.* imports in `packages/*` source).

Product app code in `apps/web` server (RSC + Server Actions + Route Handlers) uses Bun.*; client islands use Web APIs.

## Caught by

- `tools/lint/bun-first.ts` greps server-side scripts (`apps/web/src/app/**/page.tsx` RSC + `apps/web/src/app/**/route.ts` + `apps/web/src/server/**` + `tools/**`) for banned-replaceable patterns:
  - `process.env.` (must be `Bun.env.`)
  - `import fs from 'fs-extra'` (banned)
  - `from 'execa'` / `from 'zx'` (banned)
  - `from 'glob'` / `fast-glob` (banned)
  - `new Date()` + `setTimeout` in non-trivial scheduling contexts (prefer `Bun.cron` / `Bun.sleep`)
  - `process.exit` without operator-approved exception (use thrown errors)
- pm4ai `pm4ai status` command surfaces banned-deps + Bun-replaceable hits
