# Subdomains & Reverse Proxy

*Last updated: May 16, 2026*

## Overview

All HTTPS termination is handled by Caddy on CT 102 (`192.168.10.70`). Let's Encrypt certificates are issued automatically via Cloudflare DNS-01 challenge — no ports need to be open to the internet.

## Active Subdomains

| Subdomain | Backend | Notes |
|---|---|---|
| vault.mystikal.dev | localhost:8080 | Vaultwarden on same LXC |
| home.mystikal.dev | 192.168.1.71:3000 | Homepage |
| monitor.mystikal.dev | 192.168.1.71:3001 | Uptime Kuma |
| proxmox.mystikal.dev | https://192.168.1.10:8006 | Self-signed backend |
| unifi.mystikal.dev | https://192.168.1.69:11443 | Self-signed backend + CORS fix |

## Full Caddyfile

`/etc/caddy/Caddyfile`

```
{
  email franksantosfung@gmail.com
}

vault.mystikal.dev {
  reverse_proxy localhost:8080
  tls {
    dns cloudflare {env.CLOUDFLARE_API_TOKEN}
  }
}

home.mystikal.dev {
  reverse_proxy 192.168.1.71:3000
  tls {
    dns cloudflare {env.CLOUDFLARE_API_TOKEN}
  }
}

monitor.mystikal.dev {
  reverse_proxy 192.168.1.71:3001
  tls {
    dns cloudflare {env.CLOUDFLARE_API_TOKEN}
  }
}

proxmox.mystikal.dev {
  reverse_proxy https://192.168.1.10:8006 {
    transport http {
      tls_insecure_skip_verify
    }
  }
  tls {
    dns cloudflare {env.CLOUDFLARE_API_TOKEN}
  }
}

unifi.mystikal.dev {
  reverse_proxy https://192.168.1.69:11443 {
    transport http {
      tls_insecure_skip_verify
      versions 1.1
    }
    header_up Host {upstream_hostport}
    header_up X-Real-IP {remote_host}
    header_up X-Forwarded-For {remote_host}
    header_up X-Forwarded-Proto {scheme}
  }
  header {
    -X-Frame-Options
  }
  tls {
    dns cloudflare {env.CLOUDFLARE_API_TOKEN}
  }
}
```

## Cloudflare API Token

The token is stored in `/etc/caddy/caddy.env` — never in the Caddyfile directly.

```bash
# /etc/caddy/caddy.env (owned by caddy:caddy, chmod 600)
CLOUDFLARE_API_TOKEN=<token>
```

Loaded via systemd override at `/etc/systemd/system/caddy.service.d/override.conf`:

```ini
[Service]
EnvironmentFile=/etc/caddy/caddy.env
```

!!! danger
    If the token is ever exposed (pasted in chat, committed to git, etc.) — rotate it immediately in Cloudflare → My Profile → API Tokens → Roll.

## UniFi CORS Fix

UniFi OS validates WebSocket origin headers against an internal CORS allowlist. Without adding `unifi.mystikal.dev`, nginx returns HTTP 500 on every WebSocket handshake after MFA, preventing the UI from loading.

Fix — added inside the UniFi podman container at `/data/unifi-core/config/http/`:

```nginx
# cors-origin-mystikal.conf
~^https://unifi\.mystikal\.dev$ $http_origin;

# cors-credentials-mystikal.conf
~^https://unifi\.mystikal\.dev$ "true";
```

These files persist in the `uosserver_data` podman volume — they survive container updates.

## Adding a New Subdomain

1. Add A record in Cloudflare → point to `192.168.10.70`, DNS only (grey cloud)
2. Add host override in OPNsense → Services → Unbound DNS → Host Overrides → `192.168.10.70`
3. Add block to `/etc/caddy/Caddyfile`
4. `systemctl reload caddy` — cert is issued automatically

```
newservice.mystikal.dev {
  reverse_proxy <backend-ip>:<port>
  tls {
    dns cloudflare {env.CLOUDFLARE_API_TOKEN}
  }
}
```

## Caddy Commands

```bash
systemctl status caddy       # check status
systemctl reload caddy       # reload config (no downtime)
systemctl restart caddy      # full restart
journalctl -u caddy -f       # live logs
caddy version                # check version
```
