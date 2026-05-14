# health-check

## Decision

Two endpoints: `/api/healthz` (liveness) + `/api/readyz` (readiness).

## Liveness — `/api/healthz`

Asserts the Next process is up. Returns 200 with minimal body.

```ts
export async function GET() {
  return new Response('ok', {
    status: 200,
    headers: { 'cache-control': 'no-store', 'content-type': 'text/plain' },
  });
}
```

Dokploy uses this for container liveness. Restarts container if endpoint returns non-2xx or times out.

## Readiness — `/api/readyz`

Asserts dependencies reachable: Convex backend, file storage, any other operator zoo primitives.

```ts
export async function GET() {
  const checks = {
    convex: await pingConvex(),     // typed timeout via AbortSignal.timeout(2000)
    storage: await pingStorage(),
    db: await pingDb(),
  };
  const allOk = Object.values(checks).every(Boolean);
  return Response.json(checks, {
    status: allOk ? 200 : 503,
    headers: { 'cache-control': 'no-store' },
  });
}
```

Dokploy uses this for routing decisions: traffic flows only when readyz is 200.

## Why two endpoints

- **Liveness** restarts the process. Cheap, always-true unless process is wedged.
- **Readiness** controls routing. Returns 503 during dep outage so load balancer routes around.

Mixing them = wrong-class restart on dep outage (won't fix dep, kills serving capacity).

## Timeouts

Per `book/HARD-RULES.md` "Every wait loop has a deadline":
- Convex ping: 2 s
- Storage ping: 2 s
- DB ping: 2 s

Endpoint overall budget: 3 s. If any check times out → that check `false` → 503.

## Smoke

`make smoke.deploy` includes a healthz + readyz hit. Both must be 200 for deploy to be considered green.

## Caught by

- Dokploy probe configuration references these paths
- Deploy smoke: both endpoints 200
- Readiness regression: simulate Convex down → readyz returns 503, healthz still 200
