# Deployment

Procedures for deploying strut services to a VPS.

## Quick Deploy

```bash
strut my-stack release --env prod --dry-run   # Preview
strut my-stack release --env prod             # Full release
```

`release` runs on the VPS over SSH and automatically: updates the repo → runs migrations → deploys → verifies health.

## Manual Deploy Steps

```bash
strut my-stack update --env prod              # Sync strut repo on VPS
strut my-stack deploy --env prod              # Deploy containers
strut my-stack health --env prod --json       # Verify
```

## Stop Containers

```bash
strut my-stack stop --env prod                # Stop containers
strut my-stack stop --env prod --volumes      # Stop + remove volumes
strut my-stack stop --env prod --dry-run      # Preview
```

## Service Profiles

```bash
strut my-stack deploy --env prod                          # core (default)
strut my-stack deploy --env prod --services messaging     # + messaging
strut my-stack deploy --env prod --services full          # everything
```

## Deploy Modes

```bash
strut my-stack deploy --env prod --standard     # in-place (default)
strut my-stack deploy --env prod --blue-green    # stand up green, health-gate, swap, drain blue
```

## First-Time Setup

1. Create an env file from the template, fill in secrets (`VPS_HOST`, registry creds, DB passwords).
2. Configure `strut.conf` with registry type and org.
3. Deploy: `strut my-stack release --env prod`.

## Migrations

```bash
strut my-stack migrate postgres --status --env prod   # Check
strut my-stack migrate postgres --up --env prod       # Apply
strut my-stack migrate neo4j --down 1 --env prod      # Rollback one
```

## Rollback

```bash
strut my-stack rollback --env prod            # Restore previous deploy snapshot
```

## Local vs VPS Semantics

- `deploy` / `stop` — runs locally, or on the VPS if `VPS_HOST` is set
- `release` — always runs on the VPS via SSH
- `update` — syncs the strut repo on the VPS (no container restart)
- `shell` / `exec` — SSH to the VPS

## Best Practices

1. Always `--dry-run` first for destructive commands.
2. Back up before major changes: `strut my-stack backup all --env prod`.
3. Use `release` for VPS deploys (runs remotely), not `deploy` (runs locally).
4. Keep `services.conf` health checks current — they gate deploy success.
