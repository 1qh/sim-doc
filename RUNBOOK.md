# RUNBOOK

Operator emergency + routine procedures. Concrete commands, not theory.

## Routine

### Deploy a new build

```bash
cd ~/sim
bun run check          # full local check, green required
bun run build          # turbo build
make deploy            # dokploy apply + smoke
```

Smoke fails → roll back via `make rollback` (re-applies previous tagged image).

### Add a new substrate package

```bash
cd ~/sim
bun scripts/new-package.ts <name>
```

Scaffolds `packages/<name>/` with `package.json`, `tsconfig.json`, `src/`, foundation-demo entry under `apps/web/learn/foundation/<name>/`. Per `adr/monorepo-package-split.md`.

### Add a new MIPS instruction to the encodable tier

```bash
# 1. Update simdocs/MIPS-ISA.md opcode + funct table
# 2. cd ~/sim
bun run gen.isa        # regenerates apps/web/src/features/datapath/generated/isa.ts
# 3. Add golden-trace fixture under tools/test/golden-traces/<mnemonic>.json
bun test golden        # asserts new fixture matches executor output
```

### Add a new K-map example exercise

```bash
# Edit apps/web/content/kmap-examples/<slug>.mdx
# Snapshot test auto-runs on commit
```

### Rotate the Cloudflare API token

```bash
# 1. Generate a new token in the Cloudflare dashboard (founder UI click)
# 2. Update operator-local secrets root:
$EDITOR <secrets-root>/cloudflare-api-token.env
# 3. Push to deploy env:
make env.sync
# 4. Smoke:
make smoke.deploy
```

### Add a share schema version (expand-contract)

```bash
# 1. Bump packages/sim-engine version handler (add new optional field)
# 2. Commit old-version fixture under tools/test/replay-fixtures/v<N>/
bun test replay        # asserts old fixtures still replay
```

## Emergency

### Roll back a bad deploy

```bash
cd ~/sim
make rollback          # redeploys previous tagged image via dokploy
make smoke.deploy      # verifies
```

### App container down

```bash
# 1. Check dokploy logs:
dokploy logs sim
# 2. Restart:
dokploy restart sim
# 3. If the image is bad, roll back:
make rollback
```

### Caddy / TLS issues

```bash
# 1. Check Caddy logs:
dokploy logs caddy
# 2. Renew certs manually:
dokploy exec caddy caddy reload
# 3. Verify TLS:
curl -vI https://<domain>
```

### Bootstrap a fresh operator machine

```bash
# 1. Restore secrets root from operator backup
unzip <operator-backup>.zip -d <secrets-root>

# 2. Clone repos
git clone <sim-repo-url> ~/sim
git clone <simdocs-repo-url> ~/simdocs

# 3. Bootstrap
cd ~/sim
sh tools/bootstrap-mac.sh

# 4. Verify
make verify.fresh
```

## Diagnostic

### Tail logs from deployed instance

```bash
dokploy logs <service-name> --follow
```

### Check CDN cache hit ratio

```bash
# Cloudflare dashboard or API:
curl "https://api.cloudflare.com/client/v4/zones/<zone-id>/analytics/dashboard" \
  -H "Authorization: Bearer $CF_API_TOKEN"
```

## Caught by

- Each procedure above has a corresponding tracked script under `tools/runbook/<procedure>.sh`
- Smoke tests cover the happy path of each routine procedure
- Emergency procedures exercised quarterly (game day)
