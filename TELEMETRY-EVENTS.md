# TELEMETRY-EVENTS

Exact list of Plausible events fired. Aggregate-only, no identifiers, no per-user funnel.

Per `adr/observability-policy.md`.

## Page views

Plausible default — every navigation fires a `pageview` event. URL recorded, no query string with identifiers.

Excluded from tracking:
- `/api/*`
- `/s/*` (shared snapshots — would inflate counts with viral spread, not signal)
- `/admin/*`
- `/me`

## Custom events

| Event name | Trigger | Props (aggregate dimensions only) |
|---|---|---|
| `sim_step_taken` | User steps datapath or pipeline | `{ feature: 'datapath' \| 'pipeline' }` |
| `sim_instruction_loaded` | User picks an instruction in `/datapath` | `{ mnemonic: '<mnemonic>' }` — opcode names not identifiers |
| `kmap_grouped` | User completes a grouping in K-map | `{ vars: 2..6, mode: '2d' \| '3d' }` |
| `kmap_solver_revealed` | User toggles solver-reveal | `{ vars: 2..6 }` |
| `snapshot_saved` | Save action completes | `{ feature: 'datapath' \| 'kmap' \| 'pipeline' \| 'compare', tier: 1 \| 2 }` |
| `snapshot_shared` | Share button copied a permalink | same shape |
| `snapshot_loaded_from_permalink` | `/s/[hash]` rendered (server-side count, not client) | `{ contentType, tier }` |
| `signin_completed` | User completed Google OAuth flow | `{}` — count only |
| `anon_claimed` | `claimAnonSnapshots` mutation succeeded | `{ claimedCount: 1 \| 2..5 \| 6..10 \| 10+ }` — bucketed |
| `example_loaded` | User loaded a preset example | `{ slug: '<example-slug>' }` |
| `learn_page_viewed` | User scrolled past 80% of a learn page | `{ slug: '<page-slug>' }` |
| `cross_link_triggered` | User clicked "load from datapath" or "show in datapath" cross-link | `{ direction: 'kmap-to-datapath' \| 'datapath-to-kmap' }` |
| `command_palette_action` | User selected a command palette result | `{ kind: 'route' \| 'learn' \| 'example' \| 'instruction' \| 'component' }` |
| `error_surfaced` | Client surfaced an error to the user | `{ errCode: '<code-from-ERROR-CATALOG>' }` |
| `pwa_install_prompted` | Browser fired beforeinstallprompt | `{}` |
| `pwa_installed` | User installed PWA | `{}` |
| `offline_save_queued` | Save attempted offline | `{}` |
| `offline_save_replayed` | Queued save persisted on reconnect | `{}` |
| `theme_palette_switched` | User changed color-blind palette | `{ palette: 'default' \| 'deuteranopia' \| 'protanopia' \| 'tritanopia' }` |
| `reduced_motion_overridden` | User toggled reduced-motion in settings | `{ to: 'on' \| 'off' \| 'system' }` |

## Banned event shapes

- Anything containing user identifier (email, userId, IP, session token)
- Anything containing free-form user input (asm source, K-map truth table values, snapshot bodies)
- Anything per-snapshot identifying (hash values, even truncated)
- Anything time-of-day-specific that could correlate with single-user activity
- Anything that would let aggregate usage be reverse-engineered to identify individuals

## DNT compliance

Every event check honors `navigator.doNotTrack === '1'` AND `Sec-GPC: 1` header. Event fired only when both pass.

## Plausible config

- Self-host instance (per `adr/observability-policy.md`)
- Goals configured for: `snapshot_saved`, `signin_completed`, `learn_page_viewed`
- Funnels: none (no per-user funnel allowed)

## Caught by

- `tools/lint/telemetry-events.ts` greps event-emit sites; every event name must appear in this catalog with matching props shape
- Privacy smoke: emit each event in test, assert no PII appears in payload
- DNT smoke: requests with DNT=1 produce zero events
