# SECURITY

Threat model + defense layers for a pure-client web app. No accounts, no server-side persistence, no backend to compromise.

## Surface

The app is a static client. There is no login, no session, no user data store, and no server-write path. The real attack surface is the browser: XSS, content-security-policy gaps, dependency supply-chain, and untrusted share-fragment input that the client decodes locally.

## Threat model

| Threat | Mitigation |
|---|---|
| Untrusted share fragment (state injected via URL hash) | Fragment is client-decoded untrusted input: base64url → `DecompressionStream("deflate-raw")` → bytes → `sim-engine` codec. Validated against Zod schemas on decode; malformed or schema-violating payloads are rejected, never executed. No `eval`, no HTML, no inline scripts. |
| XSS via decoded state content | Decoded state is typed binary deserialized to typed fields; UI renders only typed fields through React/JSX. No `dangerouslySetInnerHTML` anywhere in product. |
| XSS in user-authored Asm / Boolean expr | Editor uses Monaco's typed model; rendering uses React, no `dangerouslySetInnerHTML`. |
| Oversize share payload | A state too large for a URL fragment is labeled tier `'oversize'` by the encoder and is not shareable. There is no server fallback, so no upload surface exists to attack. |
| Supply-chain (compromised npm dep) | Renovate auto-bumps with minimum-release-age gate (per `book/HARD-RULES.md`), CVE gate, manifest-diff gate. |
| Malicious script injection at runtime | Strict Content-Security-Policy: no inline scripts, no `eval`, locked script-src to self + the analytics origin. |

## Defense in depth (per `book/HARD-RULES.md`)

Every safeguard has at least two independent enforcement points:

| Invariant | Layer 1 | Layer 2 |
|---|---|---|
| No raw HTML in user content | Editor's typed model rejects raw HTML at parse time | React rendering uses JSX only, no `dangerouslySetInnerHTML` |
| Decoded share fragment is well-formed | Codec deserializer rejects bytes that fail the schema-version prefix check | Zod schema validation rejects any field violating the typed shape |
| No script injection | CSP `script-src 'self'` blocks inline + remote scripts | `no-eval` lint asserts no `eval` / `Function(` constructors in source |

## Secrets

No server secrets are needed for app operation — there is no backend, no auth provider, no deploy key. The only client-side configuration is the public analytics domain (a non-secret) injected at build.

## Logging + redaction

- No PII is collected, so none can be logged.
- Client error reporting carries no identifiers and scrubs decoded state bodies before any report leaves the browser.
- Errors quoted exact in dev; redacted to error-class only in prod.

## TLS

- Caddy auto-provisions Let's Encrypt certs
- HSTS on at Cloudflare
- Always-HTTPS redirect

## Responsible disclosure

Substrate packages are public OSS. Vulnerability disclosures are accepted via private email to the maintainer (address provided in the substrate repo's `SECURITY.md`, never public issue).

Triage:
- Acknowledge within 72h
- Fix or patched-mitigation within 14 days for high-severity
- Public disclosure after fix lands, with credit to reporter unless they opt out
- CVE assignment for confirmed vulnerabilities affecting OSS substrate

## Caught by

- `tools/lint/no-dangerously-set-inner-html.ts` greps for `dangerouslySetInnerHTML` anywhere
- `tools/lint/no-eval.ts` greps for `eval`, `Function(` constructors
- CSP smoke: response carries a `script-src 'self'` policy; inline-script fixture is blocked
- Share-decode fuzz: random + adversarial fragments decode to either a valid typed state or a clean rejection, never an execution path
