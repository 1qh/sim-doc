# substrate-publication

## Decision

Both repos public on GitHub. `sim` (substrate packages + product app + Convex backend + tools) and `simdocs` (full docs corpus) both ship as public-OSS repositories.

| Surface | Visibility |
|---|---|
| `packages/three-kit`, `hud`, `design-tokens`, `sim-engine`, `editor`, `bits`, `boolean` | Public OSS source |
| `apps/web` (product app) | Public OSS source |
| `apps/backend` (Convex schema + functions) | Public OSS source |
| `tools/*` | Public OSS source |
| `simdocs` repo | Public OSS source |

## License

- Substrate packages (`packages/*`): MIT.
- Product app (`apps/web` + `apps/backend`): MIT for substrate-extracted shapes; proprietary-shaped product code stays MIT as well unless an explicit alternative lands in an ADR amendment.
- `simdocs`: CC0-1.0 (docs license, public-domain-equivalent).
- `tools/*`: MIT.

## Repo host

GitHub under operator's account. Both `sim` and `simdocs` repos public from first push.

## Publishing as installable artifact

Substrate packages stay source-only inside this monorepo until a second consumer exists. Per `book/SUBSTRATE.md` "Premature publication anti-pattern" — extract-publish-on-second-use, not speculative future-consumer. `package.json` carries `"private": true` until extraction trigger fires.

## Discipline forcing function

Per `book/SUBSTRATE.md` "Substrate stays clean BECAUSE it is published" — visible publication is the forcing function. Every substrate commit reviewed against "is this generic enough to ship to strangers". A diff that fails the test belongs in `apps/web`, not in a package.

Product app being public also means: secrets never committed (per `book/HARD-RULES.md` single-secrets-root); env shape documented but values stay in operator-local secrets; no API keys / OAuth client secrets / Convex deploy keys / admin emails in tracked source.

## Caught by

- Substrate-boundary lint scans `packages/*` source for domain vocab (MIPS, K-map, datapath, kmap, opcode-specific names) — must be zero hits per `book/HARD-RULES.md` "Never reference our-codebase identifiers in docs" generalized to source.
- `package.json` `"private": true` enforced on substrate packages until publish-trigger fires (second-consumer evidence).
- Secrets-leak lint scans every commit for known secret-shape patterns; zero hits required.
