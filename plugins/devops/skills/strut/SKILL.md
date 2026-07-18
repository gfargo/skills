---
name: strut
description: Operate and manage Docker Compose stacks on VPS infrastructure with the strut CLI. Use for any strut task — deploying and releasing services, database backup and restore, debugging production issues, detecting config drift, rotating secrets and keys, setting up monitoring, configuring domains and SSL, validating stacks, or auditing and migrating existing servers.
license: MIT
metadata:
  author: gfargo
  version: "1.0"
---

# strut — VPS Stack Management

strut is a Bash CLI for managing Docker Compose stacks on VPS infrastructure. Commands follow the shape:

```
strut <stack> <command> [--env <name>] [options]
```

`--env <name>` selects the environment file `.<name>.env` (e.g. `--env prod` reads `.prod.env`). Most commands run against a VPS over SSH; some run locally.

## Command Quick Reference

```bash
# Deploy & release
strut <stack> release --env prod          # update repo → migrate → deploy → verify (on VPS)
strut <stack> deploy  --env prod          # deploy containers (local, or VPS if VPS_HOST set)
strut <stack> rebuild --env prod          # build images on target + deploy
strut <stack> stop    --env prod          # stop containers
strut <stack> rollback --env prod         # restore previous deploy snapshot

# Inspect
strut <stack> status  --env prod          # container status
strut <stack> health  --env prod --json   # health checks
strut <stack> briefing --env prod         # one-call situation report: posture + prioritized actions
strut <stack> preflight --env prod        # deploy go/no-go verdict (GO/CAUTION/NO-GO) before releasing
strut <stack> logs <service> --follow --env prod
strut <stack> diff    --env prod          # preview pending changes vs VPS
strut fleet status                        # git sync state across all [hosts]

# Data
strut <stack> backup all --env prod       # backup all databases
strut <stack> restore <file> --dry-run    # rehearse a restore (non-destructive)
strut <stack> db:pull --env prod          # pull prod data to local

# Drift & secrets
strut <stack> drift detect --env prod     # config drift vs git
strut <stack> drift images --env prod     # stale container image digests
strut <stack> secrets push --env prod     # sync .env to VPS
strut <stack> keys db:rotate postgres --env prod

# Infrastructure
strut <stack> domain example.com admin@example.com --env prod
strut audit <vps-host> [user] [ssh-key]   # discover what's running on a VPS
strut migrate <vps-host>                  # interactive migration wizard
```

## When to Read Each Reference

Load the relevant reference file for detailed, step-by-step procedures:

| Task | Reference |
|------|-----------|
| Deploying, releasing, updating, stopping services | [references/deployment.md](references/deployment.md) |
| Diagnosing production issues, 502s, crashes, disk/DB problems | [references/debugging.md](references/debugging.md) |
| Backing up or restoring databases, pulling prod data | [references/backups.md](references/backups.md) |
| Detecting or fixing config drift, auto-fix, drift history | [references/drift.md](references/drift.md) |
| Rotating SSH keys, API keys, DB passwords, env secrets | [references/secrets.md](references/secrets.md) |
| Setting up Prometheus/Grafana/Alertmanager monitoring | [references/monitoring.md](references/monitoring.md) |
| Configuring custom domains and SSL/TLS certificates | [references/domains-ssl.md](references/domains-ssl.md) |
| Validating stack structure and config before deploy | [references/validation.md](references/validation.md) |
| Auditing an existing VPS and migrating to strut | [references/migration.md](references/migration.md) |

## Core Principles

1. **Always `--dry-run` first** for destructive commands (deploy, restore, drift fix, stop).
2. **Back up before major changes:** `strut <stack> backup all --env prod`.
3. **Use `release` for VPS** (runs remotely over SSH), not `deploy` (runs locally).
4. **Make changes in git, not on the VPS** — let deployments propagate; drift detection catches manual edits.
5. **Health checks gate success** — driven by `services.conf`; keep it current.
6. **Assess before you act** — run `briefing` to triage a stack in one call, and `preflight` for a go/no-go before any release. Both are read-only aggregations of the checks above (`--json` for machine parsing).

## Environment Files

Per-environment `.env` files live at the project root (`.prod.env`, `.staging.env`). They contain literal `KEY=VALUE` pairs — shell expansion (`$VAR`, `$(cmd)`) is **not** evaluated (strut reads them with a safe parser, not `source`). Files are written mode `0600`.
