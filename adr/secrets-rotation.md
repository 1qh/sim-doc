# secrets-rotation

## Decision

Every long-lived secret rotates on a documented cadence. Rotation procedures scripted.

## Inventory + cadence

| Secret | Cadence | Trigger | Procedure |
|---|---|---|---|
| Cloudflare API token | Annual | Calendar + incident | Generate new token, update operator secrets root, deploy |
| Error reporter DSN (when enabled, non-Sentry) | n/a | Public, embedded in client; rotates on project rotation only | Generate new project, update env, redeploy |
| Plausible API key (when self-host) | Annual | Calendar + incident | Plausible admin UI rotate |

## Rotation discipline

- Generate new before revoking old (expand-contract per `book/HARD-RULES.md`)
- Test new in staging before promoting
- Revoke old only after prod has rolled to new for ≥24h
- Calendar reminder per secret-cadence pair; operator's calendar

## Incident rotation

Triggered when a secret is suspected leaked:
1. Generate new immediately
2. Push to staging + prod simultaneously (no expand-contract for compromise)
3. Revoke old immediately
4. Audit logs for usage of old between leak and revocation
5. Postmortem captured at the topic-owner doc

## Automation candidates (deferred)

- Renovate-style auto-rotate for credentials with API support — deferred until first rotation pain.

## Caught by

- Calendar reminders (operator-side)
- Audit log retained per `OBSERVABILITY.md` structured logs
- Smoke after each rotation: deploy works, app loads
- Secret-expiry-date warning in dashboard (when observability dashboard exists)
