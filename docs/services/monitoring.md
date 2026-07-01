# Uptime Kuma

*Last updated: May 11, 2026*

## Overview

Lightweight service monitor that checks if homelab services are up or down and sends Pushover alerts when something goes offline.

| Setting | Value |
|---|---|
| URL | https://monitor.mystikal.dev |
| LXC | CT 103 (homepage) |
| IP | 192.168.1.71:3001 |
| Docker image | louislam/uptime-kuma:latest |
| Data | /opt/homepage/uptime-kuma/ |

Runs alongside Homepage in the same Docker Compose stack on CT 103.

## Monitors

| Service | URL | Notes |
|---|---|---|
| OPNsense | https://192.168.1.1 | Ignore TLS |
| Proxmox | https://proxmox.mystikal.dev | Ignore TLS |
| Vaultwarden | https://vault.mystikal.dev | — |
| UniFi | https://unifi.mystikal.dev | Ignore TLS |
| Homepage | https://home.mystikal.dev | — |
| Uptime Kuma | http://192.168.1.71:3001 | — |

!!! note
    HTTPS services using self-signed backend certs may need "Ignore TLS/SSL error" checked in Uptime Kuma, otherwise they'll be reported as down even when running.

## Alerts

Pushover notifications configured for all monitors. Alerts fire when a service goes down and when it recovers.

## Maintenance

```bash
cd /opt/homepage
docker compose pull        # update Uptime Kuma
docker compose up -d       # apply update
docker compose logs -f     # view logs
```
