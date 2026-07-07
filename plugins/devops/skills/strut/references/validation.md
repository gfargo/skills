# Stack Validation

Validating, checking, and auditing strut stacks — whether creating a new one or verifying an existing one.

## Quick Validation (Existing Stack)

```bash
strut my-stack health --env prod            # runtime health
strut my-stack drift detect --env prod      # config drift vs git
strut my-stack db:schema verify --env prod  # Postgres schema matches sql/init/*.sql
strut my-stack keys env:validate --env prod # env vars against template
strut my-stack backup health --env prod     # backup health
```

## New Stack Checklist

### 1. Required Files

Every stack lives under `stacks/<stack-name>/`:

```
stacks/<stack-name>/
├── docker-compose.yml          # required
├── docker-compose.local.yml    # required
├── .env.template               # required
├── .drift-ignore               # recommended
├── backup.conf                 # recommended
├── nginx/                      # if using reverse proxy
├── sql/init/                   # Postgres DDL (auto-applied on first start)
└── config/                     # runtime config files
```

### 2. docker-compose.yml Validation

```bash
docker compose -f stacks/<stack-name>/docker-compose.yml config --quiet
```

Verify manually:
- All services have `restart: unless-stopped` and a `healthcheck`.
- DB-dependent services use `depends_on` with `condition: service_healthy`.
- Optional services use `profiles:`.
- No hardcoded secrets — use `${VAR}` substitution.

### 3. Environment Variables

```bash
# Compare template vs actual key names
diff \
  <(grep -E '^[A-Z_]+=?' stacks/<stack-name>/.env.template | cut -d= -f1 | sort) \
  <(grep -E '^[A-Z_]+=?' .prod.env | cut -d= -f1 | sort)

# Check for unfilled placeholders
grep -E '(your-|change-me|xxxx|placeholder|TODO)' .prod.env
```

Common required variables:

| Variable | Purpose |
|---|---|
| `VPS_HOST` | SSH target |
| `VPS_USER` | SSH user |
| `VPS_DEPLOY_DIR` | strut path on VPS |
| `COMPOSE_PROJECT_NAME` | Docker project name |

### 4. Postgres Schema

```bash
strut <stack-name> db:schema verify --env prod
ls -la stacks/<stack-name>/sql/init/    # 01_*.sql, 02_*.sql, ...
```

### 5. Drift & Backup Config

Ensure `.drift-ignore` excludes runtime-generated files, and `backup.conf` has sensible defaults:

```bash
BACKUP_SCHEDULE_POSTGRES="0 2 * * *"
BACKUP_RETAIN_DAYS=30
BACKUP_RETAIN_COUNT=10
BACKUP_POSTGRES=true
```

Test: `strut <stack-name> backup all --env prod`.

## Common Validation Failures

- **compose syntax error** — tabs instead of spaces, missing quotes, undefined `${VAR}` references.
- **missing required env vars** — use the diff command above.
- **service fails healthcheck** — `strut <stack> logs <service> --tail 50 --env prod`.
- **drift detected after first deploy** — add runtime files to `.drift-ignore`, or fix real changes with `drift fix`.
