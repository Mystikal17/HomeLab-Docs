# Services Overview

Current service inventory for mystikal.dev.

## Active Services

| Service | URL | LXC | VLAN |
|---|---|---|---|
| Vaultwarden | [vault.mystikal.dev](https://vault.mystikal.dev) | CT 102 | Servers |
| Homepage | [home.mystikal.dev](https://home.mystikal.dev) | CT 103 | Management |
| Uptime Kuma | [monitor.mystikal.dev](https://monitor.mystikal.dev) | CT 103 | Management |
| Proxmox | [proxmox.mystikal.dev](https://proxmox.mystikal.dev) | — | Management |
| UniFi | [unifi.mystikal.dev](https://unifi.mystikal.dev) | CT 101 | Management |
| OPNsense | 192.168.1.1 | bare metal | Management |

## Reverse Proxy

All HTTPS termination is handled by Caddy on CT 102 (192.168.10.70) using Let's Encrypt certificates via Cloudflare DNS-01 challenge.

All `mystikal.dev` subdomains point to `192.168.10.70` in both Cloudflare DNS and OPNsense Unbound host overrides.

## Planned Services

| Service | Purpose | Status |
|---|---|---|
| Wazuh | SIEM — log ingestion and alerting | Planned |
| Home Assistant | Smart home on IoT VLAN | Planned |
| Authentik / Keycloak | SSO across all services | Planned |
| Grafana + Prometheus | Metrics and observability | Planned |
| AdGuard Home | DNS-level ad blocking | Planned |
