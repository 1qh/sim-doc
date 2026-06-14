# API-CONVENTIONS

Conventions for every server surface — Server Actions for pure compute (assemble / solve), Route Handlers (`/api/og`, `/api/rum`, `/api/healthz`, sitemap), `ImageResponse`. No identity, no server-side writes, no persistence — the server is stateless pure-compute + static delivery only.

## Server Actions

Pure compute only (assemble a program, run a solver). Stateless: input in, typed result out, nothing persisted.

| Aspect | Rule |
|---|---|
| Naming | Verb-first, kebab-case file: `assemble-program.ts`, `solve-kmap.ts` |
| Validation | Zod v4 schema on every input at function boundary. Invalid input throws typed `BadInputError`. Per `book/HARD-RULES.md` "Zero fallback" |
| Error shape | `{ ok: false, code: string, message: string }` returned to caller (never raw exceptions to client) |
| Success shape | `{ ok: true, data: <typed> }` |
| Rate limit | Wrapped in `withRateLimit(action, { perIp: number, perWindow: number })` — protects compute cost only, not any write |
| Logging | Auto-wrapped via OBSERVABILITY logger — op name, duration, outcome |

## Route Handlers

| Aspect | Rule |
|---|---|
| Naming | RESTful, kebab-case: `/api/og`, `/api/rum`, `/api/healthz` |
| Methods | Explicit HTTP method handlers; never accept other methods |
| Status | Use HTTP status correctly: 200, 204, 400, 429, 500 |
| Content-Type | Always `application/json` unless serving a specific binary type |
| Cache-Control | Explicit per route, never default. Public + immutable for static-addressed assets |
| Validation | Zod v4 schema on every input (query params, body, headers) |
| Error shape | Same as Server Actions |

## ImageResponse (OG cards)

Per `OG-IMAGES.md`. Dynamic OG image generation via Next 16 `ImageResponse`.

Cache-Control: `public, immutable, max-age=31536000` on static-content cards; `public, max-age=3600` on dynamic-content cards.

## Error code vocabulary

Typed enum used across server surface:

| Code | Meaning |
|---|---|
| `BAD_INPUT` | Validation failure |
| `NOT_FOUND` | Resource absent |
| `RATE_LIMITED` | Rate limit hit |
| `INTERNAL` | Server error (always logged, never PII in response) |

Per `book/HARD-RULES.md` "Errors quoted exact" — dev mode returns full message + stack; prod returns code + generic message.

## Headers

| Header | Default |
|---|---|
| `Content-Security-Policy` | `default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data: https:; connect-src 'self'; font-src 'self' data:` — tightened per route |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains; preload` |
| `X-Content-Type-Options` | `nosniff` |
| `X-Frame-Options` | `DENY` (except `/embed/*` if ever shipped) |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | `interest-cohort=()` (no FLoC), camera/microphone/geolocation all off |

## Idempotency

Server surfaces are pure compute: same input → same output, naturally idempotent with no state to track. No `Idempotency-Key` machinery needed.

## Caught by

- `tools/lint/api-conventions.ts` greps for Server Action / Route Handler declarations; asserts naming + validation wrapping + error shape
- Smoke test per endpoint: valid input → success shape; invalid input → error code
- CSP test: production response includes required headers
