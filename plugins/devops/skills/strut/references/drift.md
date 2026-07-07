# Drift Detection & Auto-Fix

Detecting, reporting, and fixing configuration drift between git-tracked files and VPS runtime state.

## Quick Reference

```bash
strut my-stack drift detect --env prod        # detect config-file drift
strut my-stack drift report --env prod         # detailed diff report
strut my-stack drift report --env prod --json  # machine-readable
strut my-stack drift images --env prod         # stale container image digests
strut my-stack drift fix --env prod            # apply git-tracked config to VPS
strut my-stack drift fix --dry-run --env prod  # preview fix
strut my-stack drift history --env prod        # detection history
strut my-stack drift auto-fix enable --env prod
strut my-stack drift auto-fix disable --env prod
strut my-stack drift auto-fix status --env prod
```

## What Is Drift?

Configuration drift occurs when VPS runtime config differs from git-tracked config. Causes: manual VPS edits, failed partial deployments, un-reverted experiments, uncommitted emergency hotfixes.

Two kinds strut detects:
- **Config drift** (`drift detect`) — tracked files differ between git and the VPS.
- **Image drift** (`drift images`) — a running container's image digest differs from what its tag now resolves to on the registry (tag silently moved).

## Fixing Drift

```bash
strut my-stack drift fix --env prod
```

Fix process: backs up current VPS config → applies git-tracked config → runs health checks → restores backup if checks fail → logs and notifies.

Preview with `--dry-run` to see files to update, services to restart, estimated downtime.

## Auto-Fix

```bash
strut my-stack drift auto-fix enable --env prod
```

When enabled: drift detected on a schedule → git config applied automatically → backup before each fix → health checks after → rolls back on failure → alerts. Disable before intentional experiments.

## .drift-ignore

Some files legitimately differ at runtime. Add glob patterns to `stacks/<stack>/.drift-ignore`:

```
*.log
*.pid
*.sock
.env
.env.local
docker-compose.override.yml
*.backup
nginx/conf.d/ssl.conf
tmp/*
cache/*
```

Supports `*`, `?`, `**`, `[abc]`.

## Common Workflows

### Emergency Hotfix (Intentional Drift)

```bash
# 1. Make emergency change on VPS, then commit it back to git:
git add config && git commit -m "Emergency hotfix: <description>" && git push
# 2. Drift resolves on the next detection cycle.
```

### Configuration Experiment

```bash
strut my-stack drift auto-fix disable --env prod
# ... experiment on VPS, test ...
# If successful, commit to git, then:
strut my-stack drift auto-fix enable --env prod
```

## Troubleshooting

### False Positives (line endings)

```bash
file stacks/<stack>/docker-compose.yml   # check CRLF vs LF
dos2unix stacks/<stack>/docker-compose.yml
```

### Drift Not Detected

```bash
crontab -l | grep drift          # is the cron installed?
strut my-stack drift detect --env prod
strut my-stack shell --env prod   # SSH reachable?
```

## Best Practices

1. Use `.drift-ignore` for runtime-generated files.
2. Enable auto-fix on production stacks.
3. Review drift history periodically.
4. Always commit emergency hotfixes back to git.
5. Disable auto-fix before intentional experiments.
6. Make changes in git, not on the VPS.
