# disaster-recovery

## Decision

RPO (Recovery Point Objective): 0. The app holds no server-side state; nothing to lose. All shareable state lives in URL fragments held by visitors.
RTO (Recovery Time Objective): 1 hour. From total infrastructure loss to serving.

Redeploy-from-git, restore operator secrets, tested-restore protocol.

## Backup strategy

| Asset | Frequency | Retention | Destination |
|---|---|---|---|
| Caddy state (TLS certs, cache cert keys) | Weekly | 30 days | Self-host MinIO or operator's backup destination (per operator's reference deploy project pattern in memory) |
| Operator-original secrets root | Manual after every change | Operator's personal backup | Per `book/HARD-RULES.md` "Single secrets root" |
| Repo state | Push to remote | Forever | Git remote (host per `adr/repo-layout.md`) |

The app is a static client bundle; its only source of truth is git. There is no database, no user data, no server-side persistence to back up.

## Restore protocol (tested quarterly)

```bash
# 1. Fresh VM provisioned (dokploy or new VM)
# 2. Restore secrets root from operator backup
unzip <operator-backup>.zip -d <secrets-root>

# 3. Clone + bootstrap
git clone <sim-repo-url> /opt/sim
git clone <simdocs-repo-url> /opt/simdocs
cd /opt/sim
sh tools/bootstrap-mac.sh   # works on Linux too

# 4. Smoke
make smoke.deploy
```

## Tested restore (game day)

Quarterly. Operator simulates total infrastructure loss:
1. Spin up a new VM
2. Execute restore protocol from operator backup + git remote only
3. Time to serving measured against RTO

## Failure scenarios + responses

| Scenario | Response |
|---|---|
| Next container crash | Dokploy auto-restart; if persistent, `dokploy logs sim` → fix |
| VM hardware failure | Provision new VM, execute restore protocol |
| Cloudflare account compromise | Re-point DNS to direct origin; revoke API token; restore once new DNS provider in place |
| Git remote loss | Push to alternative remote; substrate repo is OSS-mirrored so canonical history survives |
| Operator-original secrets loss | Re-establish Cloudflare token + Plausible credentials; not recoverable from system, only from operator's personal backup |

## Banned

- Backup destination same as production location
- Untested-for-≥6-months restore procedure
- Backup retention shorter than 30 days
- Plaintext credentials in backup archives without encryption

## Caught by

- Quarterly tested-restore game day on operator's calendar
- Restore-time measurement landed in operations ledger per `book/PHILOSOPHY.md` "Ledger consultation"
- Deploy smoke asserts the app serves after restore
