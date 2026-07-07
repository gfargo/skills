# Monitoring Setup

Deploying and managing self-hosted monitoring (Prometheus, Grafana, Alertmanager) for strut stacks.

## Quick Reference

```bash
strut monitoring deploy                          # deploy monitoring stack
strut monitoring add-target my-stack             # add a stack to monitoring
strut monitoring remove-target my-stack          # remove a stack
strut monitoring alert-channel add email \
  --to alerts@example.com \
  --from monitoring@example.com \
  --resend-api-key re_xxx
strut monitoring alert-channel test email        # test alert delivery
strut monitoring status --json                   # monitoring status
```

> Available subcommands: `deploy`, `add-target`, `remove-target`, `alert-channel add|test`, `status`.

## Components

| Component | Purpose | Default Port |
|-----------|---------|-------------|
| Prometheus | Metrics collection, time-series DB, alert evaluation | 9090 |
| Grafana | Dashboards and visualization | 3000 |
| Alertmanager | Alert routing, grouping, notifications | 9093 |
| Node Exporter | System metrics (CPU, memory, disk, network) | 9100 |
| cAdvisor | Per-container resource metrics | 8080 |

## Installation

```bash
# 1. Deploy
strut monitoring deploy

# 2. Configure .monitoring-prod.env
#    RESEND_API_KEY=re_xxx
#    ALERT_EMAIL_TO=alerts@example.com
#    ALERT_EMAIL_FROM=monitoring@example.com
#    GRAFANA_ADMIN_USER=admin
#    GRAFANA_ADMIN_PASSWORD=<secure-password>

# 3. Add stacks
strut monitoring add-target my-stack
strut monitoring add-target another-stack

# 4. Access Grafana at http://<vps-ip>:3000
```

## Alert Channels

### Email (SMTP via Resend)

1. Create an API key at your SMTP provider (Resend free tier: 100 emails/day).
2. Configure:
   ```bash
   strut monitoring alert-channel add email \
     --to alerts@example.com --from monitoring@example.com --resend-api-key re_xxx
   ```
3. Test: `strut monitoring alert-channel test email`

### Slack

```bash
strut monitoring alert-channel add slack --webhook-url https://hooks.slack.com/services/xxx
strut monitoring alert-channel test slack
```

### Generic Webhook

```bash
strut monitoring alert-channel add webhook --url https://your-service.com/alerts --method POST
```

## Alert Severity

| Severity | Triggers | Examples |
|----------|----------|---------|
| Critical | Immediate action | Service down, DB unreachable, disk >95% |
| Warning | Attention soon | CPU >80% for 5min, memory >90%, disk >85% |
| Info | Informational | Backup completed, deploy successful, drift detected |

## Default Alert Rules

```yaml
alert: ServiceDown
expr: up == 0
for: 2m
severity: critical

alert: HighCPU
expr: rate(node_cpu_seconds_total[5m]) > 0.8
for: 5m
severity: warning

alert: DiskSpaceLow
expr: (node_filesystem_avail_bytes / node_filesystem_size_bytes) < 0.15
for: 5m
severity: warning
```

Custom rules: add YAML under `stacks/monitoring/prometheus/alerts/`.

## Cross-VPS Monitoring

Run Node Exporter + cAdvisor on remote hosts, then reach them from the monitoring VPS.

```bash
# On the remote VPS:
docker run -d --name node-exporter --restart unless-stopped -p 9100:9100 prom/node-exporter
docker run -d --name cadvisor --restart unless-stopped -p 8080:8080 \
  -v /:/rootfs:ro -v /var/run:/var/run:ro -v /sys:/sys:ro -v /var/lib/docker/:/var/lib/docker:ro \
  gcr.io/cadvisor/cadvisor

# Prefer an SSH tunnel over exposing ports publicly:
autossh -M 0 -f -N -o "ServerAliveInterval 30" \
  -L 9100:localhost:9100 ubuntu@<remote-vps>
```

## Troubleshooting

```bash
# Prometheus not scraping — check targets at http://<vps-ip>:9090/targets
curl http://localhost:9100/metrics       # Node Exporter reachable?
strut monitoring status --json

# Alerts not sending — check Alertmanager at http://<vps-ip>:9093
strut monitoring alert-channel test email
```

## Security

1. Use a strong Grafana admin password.
2. Don't expose metrics endpoints publicly — use SSH tunnels.
3. Restrict metrics ports to the monitoring VPS IP via firewall.
4. Keep webhook URLs and API keys secret.
