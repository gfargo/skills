# Debugging

Troubleshooting procedures for strut VPS deployments.

## Quick Diagnostics

```bash
strut my-stack health --env prod                      # Health check all services
strut my-stack status --env prod                      # Container status
strut my-stack logs my-service --tail 100 --env prod  # Recent logs
```

## Common Issues

### 502 Bad Gateway

nginx can't reach the backend after a container restart (new Docker IPs).

```bash
strut my-stack exec "docker compose exec nginx nginx -s reload" --env prod
```

### Port Already Allocated

```bash
strut my-stack stop --env prod                # Stop everything cleanly
strut my-stack deploy --env prod              # Redeploy
```

### Service Won't Start

```bash
strut my-stack logs my-service --tail 100 --env prod
strut my-stack exec "docker compose exec my-service env" --env prod
```

Common causes: missing env vars, DB connection issues, port conflicts, disk full.

### Database Connection Issues

```bash
strut my-stack exec "docker compose exec postgres pg_isready -U postgres" --env prod
strut my-stack exec "docker compose exec redis redis-cli ping" --env prod
```

### Disk Space

```bash
strut my-stack exec "df -h /" --env prod
strut my-stack exec "docker system df" --env prod
strut my-stack exec "docker image prune -f" --env prod
```

## Advanced

```bash
strut my-stack shell --env prod                            # Interactive SSH
strut my-stack exec "docker stats --no-stream" --env prod  # Resource usage
strut my-stack exec "docker inspect <container>" --env prod
```

## exec vs shell

- `exec` — single commands, automation, quick checks
- `shell` — interactive debugging, multiple commands
