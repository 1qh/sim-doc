# observability-policy

## Decision

Self-host aggregate-only privacy-respecting:
- Plausible Analytics for page-view + aggregate event counts (self-host operator)
- Client RUM (Real User Monitoring) for anonymous performance metrics
- Structured JSON logs to stdout, captured by Dokploy log pipeline
- Optional platform-managed error reporter or error-boundary surfaces only (deferred unless triggered) — pm4ai bans `@sentry`, `@bugsnag`, `@honeybadger`, `rollbar`
- Optional self-host Grafana + Prometheus (deferred unless triggered)
- Optional OpenTelemetry tracing (deferred unless triggered)

## Why not third-party hosted

- No persistent visitor identity at any third party
- Per `book/PHILOSOPHY.md` self-host first + bearer-mode-or-self-host invariant
- Aggregate-only respects visitor privacy without consent banners

## Plausible config

- Self-host Plausible container in operator zoo
- Single site = our domain
- Cookieless (default Plausible behavior)
- DNT-respecting
- Events tracked: page views, share-link created (count only), example loaded
- No custom dimensions; visitors are anonymous, no identity to tie to

## Client RUM

- Anonymous performance signals only: Core Web Vitals, bundle load time, share-encode duration
- No visitor identifier; aggregate buckets only
- Sampled, batched, emitted to the self-host pipeline

## Log format

Structured JSON to stdout. Caddy log → Dokploy → operator's log pipeline.

Fields:
- `ts`, `level`, `service`, `op`, `duration_ms`, `outcome`, `errCode`, `redacted_fields`

PII redaction at boundary: any request body or header that could carry personal data → hash or redact before log emit. Visitors are anonymous; logs carry no identity.

## Banned

- Google Analytics, Mixpanel, Segment, Amplitude, Hotjar, FullStory, Heap, Plausible-Cloud (use self-host)
- Per-visitor behavioral tracking
- Session recording
- A/B testing infrastructure that targets individuals
- Cross-domain tracking

## Triggers to add deferred layers

- Platform-managed error reporter (not Sentry/Bugsnag/Honeybadger/Rollbar per pm4ai): error rate exceeds operator-bearable diagnostic threshold via logs alone
- Self-host Grafana: operator needs visual dashboards
- OpenTelemetry: distributed tracing needed (low-priority for current scope)

Each trigger lands as ADR amendment, never speculative.

## Pitfall

- Speculation-rules prerendered pages run JS, so analytics events fire before the user navigates — gate every analytics call on `document.prerendering === false` and listen for `prerenderingchange` to flush queued events on activation.

## Caught by

- `tools/lint/no-third-party-trackers.ts` greps for known tracker SDKs / script URLs
- Log scrub-test: PII tokens not present in log output
- DNT-header smoke: requests with `DNT: 1` produce no Plausible event
- Self-host Plausible compose service runs in `verify.local`
