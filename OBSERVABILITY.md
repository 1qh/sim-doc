# OBSERVABILITY

Self-host, aggregate-only, privacy-respecting. Client-side telemetry only — there is no backend to observe. Per `adr/observability-policy.md`.

## Server logs

The only server is static client delivery. Caddy + Dokploy capture access logs for the static origin:

```json
{
  "ts": "...",
  "level": "info | warn | error",
  "service": "caddy",
  "op": "asset-serve | route-serve",
  "duration_ms": 12,
  "outcome": "ok | error"
}
```

No PII is ever logged — none is collected. No emails, no user IDs, no identifiers of any kind exist to log.

## Metrics

Cookieless Plausible Analytics (privacy-respecting, no cookies, DNT-respecting):
- Page view per route (aggregate counts only)
- Custom interaction events (see `TELEMETRY-EVENTS.md`)

No per-user funnel. No conversion tracking. No retargeting pixels. No third-party scripts.

## Client error tracking

Error-boundary discipline (per `adr/observability-policy.md`). pm4ai bans `@sentry`, `@bugsnag`, `@honeybadger`, `rollbar`.

When enabled:
- Source maps uploaded at build (private, not served to browser)
- No identifiers attached; decoded state bodies scrubbed at the boundary before any report leaves the browser
- Rate-limited to prevent spam
- Connected to a Plausible event for "error class X occurred"

## Real-user performance

Web Vitals (LCP, INP, CLS, FCP, TTFB) collected in the browser with no identifiers attached, reported as aggregate Plausible events. No request carries any per-user data.

## Internal dashboards

Self-host Grafana + Prometheus OR none. When enabled, exposed only on internal network (Caddy basic-auth or VPN-only).

Dashboards:
- Page view + sim usage aggregate
- Client error rate per route
- Web Vitals distribution (p50/p95/p99)
- CDN cache-hit ratio at Cloudflare

## Banned

- Third-party analytics with non-bearer persistence (Google Analytics, Mixpanel, Segment, Amplitude, Hotjar, FullStory, Heap)
- Per-user behavioral tracking
- Session recording
- Cookie-based tracking (Plausible is cookieless)
- Cross-domain tracking
- Marketing pixels
- A/B testing infrastructure that targets individuals

## Caught by

- `tools/lint/no-third-party-trackers.ts` greps for known tracker script URLs + SDK package names; zero hits
- DNT-header smoke: requests with `DNT: 1` produce no Plausible event
- Privacy budget: scrub-test asserts no decoded-state or identifier tokens slip through the error redactor
