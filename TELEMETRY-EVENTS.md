# TELEMETRY-EVENTS

Exact list of Plausible events fired. Client-side only, aggregate-only, no identifiers, no per-user funnel.

Per `adr/observability-policy.md`.

## Page views

Plausible default — every navigation fires a `pageview` event. URL recorded, no query string with identifiers. The share fragment (after `#`) is never sent — Plausible records the path only, and the hash stays in the browser.

Excluded from tracking:
- `/s` shared-snapshot views (would inflate counts with viral spread, not signal)
- `/me` (local snapshot list)

## Custom events

| Event name | Trigger | Props (aggregate dimensions only) |
|---|---|---|
| `sim_step_taken` | User steps datapath or pipeline | `{ feature: 'datapath' \| 'pipeline' }` |
| `sim_instruction_loaded` | User picks an instruction in `/datapath` | `{ mnemonic: '<mnemonic>' }` — opcode names not identifiers |
| `kmap_grouped` | User completes a grouping in K-map | `{ vars: 2..6, mode: '2d' \| '3d' }` |
| `kmap_solver_revealed` | User toggles solver-reveal | `{ vars: 2..6 }` |
| `snapshot_saved` | Save to local snapshot list completes | `{ feature: 'datapath' \| 'kmap' \| 'pipeline' \| 'compare' }` |
| `snapshot_shared` | Share button copied a permalink | `{ feature: 'datapath' \| 'kmap' \| 'pipeline' \| 'compare', tier: 'fragment' \| 'oversize' }` |
| `snapshot_share_oversize` | Share attempted on a state too large for a URL fragment | `{ feature: 'datapath' \| 'kmap' \| 'pipeline' \| 'compare' }` |
| `snapshot_loaded_from_permalink` | A shared link decoded and rendered in the browser | `{ feature, outcome: 'ok' \| 'rejected' }` |
| `example_loaded` | User loaded a preset example | `{ slug: '<example-slug>' }` |
| `learn_page_viewed` | User scrolled past 80% of a learn page | `{ slug: '<page-slug>' }` |
| `cross_link_triggered` | User clicked "load from datapath" or "show in datapath" cross-link | `{ direction: 'kmap-to-datapath' \| 'datapath-to-kmap' }` |
| `command_palette_action` | User selected a command palette result | `{ kind: 'route' \| 'learn' \| 'example' \| 'instruction' \| 'component' }` |
| `error_surfaced` | Client surfaced an error to the user | `{ errCode: '<code-from-ERROR-CATALOG>' }` |
| `pwa_install_prompted` | Browser fired beforeinstallprompt | `{}` |
| `pwa_installed` | User installed PWA | `{}` |
| `theme_palette_switched` | User changed color-blind palette | `{ palette: 'default' \| 'deuteranopia' \| 'protanopia' \| 'tritanopia' }` |
| `reduced_motion_overridden` | User toggled reduced-motion in settings | `{ to: 'on' \| 'off' \| 'system' }` |

## Banned event shapes

- Anything containing a user identifier (IP, session token) — none exist anyway
- Anything containing free-form user input (asm source, K-map truth table values, snapshot bodies, share-fragment payloads)
- Anything per-snapshot identifying (content hashes, even truncated)
- Anything time-of-day-specific that could correlate with single-user activity
- Anything that would let aggregate usage be reverse-engineered to identify individuals

## DNT compliance

Every event check honors `navigator.doNotTrack === '1'` AND `Sec-GPC: 1` header. Event fired only when both pass.

## Plausible config

- Cookieless instance (per `adr/observability-policy.md`)
- Goals configured for: `snapshot_saved`, `snapshot_shared`, `learn_page_viewed`
- Funnels: none (no per-user funnel allowed)

## Caught by

- `tools/lint/telemetry-events.ts` greps event-emit sites; every event name must appear in this catalog with matching props shape
- Privacy smoke: emit each event in test, assert no PII or share-payload appears in payload
- DNT smoke: requests with DNT=1 produce zero events
