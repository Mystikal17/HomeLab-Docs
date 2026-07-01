# Homepage Dashboard

*Last updated: May 11, 2026*

## Overview

Self-hosted dashboard providing a single page to access and monitor all homelab services.

| Setting | Value |
|---|---|
| URL | https://home.mystikal.dev |
| LXC | CT 103 (homepage) |
| IP | 192.168.1.71 |
| VLAN | Management |
| Docker image | ghcr.io/gethomepage/homepage:latest |
| Config location | /opt/homepage/config/ |

## Docker Compose

`/opt/homepage/docker-compose.yml`

```yaml
services:
  homepage:
    image: ghcr.io/gethomepage/homepage:latest
    container_name: homepage
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - HOMEPAGE_ALLOWED_HOSTS=192.168.1.71:3000,home.mystikal.dev
    volumes:
      - ./config:/app/config
      - /var/run/docker.sock:/var/run/docker.sock

  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    restart: unless-stopped
    ports:
      - "3001:3001"
    volumes:
      - ./uptime-kuma:/app/data
```

!!! note
    `HOMEPAGE_ALLOWED_HOSTS` is required. Without it Homepage rejects all requests. Both the environment variable AND `settings.yaml` must include the domain.

## Config Files

### settings.yaml

```yaml
title: Homelab
allowedHosts:
  - 192.168.1.71
  - home.mystikal.dev
theme: dark
color: slate
headerStyle: clean
layout:
  Network:
    icon: mdi-network
  Infrastructure:
    icon: mdi-server
```

### services.yaml

```yaml
- Network:
    - OPNsense:
        href: https://192.168.1.1
        description: Firewall & Router
        icon: opnsense.png
    - UniFi:
        href: https://unifi.mystikal.dev
        description: Network Controller
        icon: unifi.png

- Infrastructure:
    - Proxmox:
        href: https://proxmox.mystikal.dev
        description: Hypervisor
        icon: proxmox.png
    - Vaultwarden:
        href: https://vault.mystikal.dev
        description: Password Manager
        icon: vaultwarden.png
    - Uptime Kuma:
        href: https://monitor.mystikal.dev
        description: Service Monitor
        icon: uptime-kuma.png
```

## Adding a New Service

1. Add to `services.yaml` under the appropriate group
2. Restart the container: `cd /opt/homepage && docker compose restart`
3. Homepage has built-in icons for most services: [dashboard-icons](https://github.com/walkxcode/dashboard-icons)

## Maintenance

```bash
cd /opt/homepage
docker compose pull       # update images
docker compose up -d      # apply updates
docker compose restart    # restart containers
docker compose logs -f    # view logs
```
