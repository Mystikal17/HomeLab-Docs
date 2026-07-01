# Vaultwarden

*Last updated: June 25, 2026*

## Overview

Self-hosted password manager (Bitwarden-compatible) running as a Docker container on CT 102.

| Setting | Value |
|---|---|
| URL | https://vault.mystikal.dev |
| LXC | CT 102 (vaultwarden) |
| IP | 192.168.10.70 |
| VLAN | Servers (10) |
| Docker image | vaultwarden/server:latest |
| Compose location | /opt/vaultwarden/ |

## Docker Compose

`/opt/vaultwarden/docker-compose.yml`

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    volumes:
      - ./data:/data
    ports:
      - "127.0.0.1:8080:80"
    env_file:
      - .env
```

!!! important
    Port is bound to `127.0.0.1:8080` — not `0.0.0.0:8080`. This ensures only Caddy (running on the same LXC) can reach Vaultwarden. Nothing else on the network can connect directly on port 8080.

## Environment Config

`/opt/vaultwarden/.env`

```env
WEBSOCKET_ENABLED=true
SIGNUPS_ALLOWED=false
ADMIN_TOKEN=$$argon2id$$...  # hashed — never plaintext
```

Generate a secure admin token:

```bash
docker exec -it vaultwarden /vaultwarden hash --preset owasp
```

!!! warning
    Dollar signs in the Argon2 hash must be doubled (`$$`) in the `.env` file to prevent Docker Compose from interpreting them as variables.

## Security Hardening

- `SIGNUPS_ALLOWED=false` — no public self-registration
- Argon2id hashed admin token (OWASP preset)
- Port bound to localhost only
- TOTP 2FA enforced on the user account
- Recovery code stored as a Vaultwarden secure note

## Reverse Proxy (Caddy)

Caddy block in `/etc/caddy/Caddyfile`:

```
vault.mystikal.dev {
  reverse_proxy localhost:8080
  tls {
    dns cloudflare {env.CLOUDFLARE_API_TOKEN}
  }
}
```

## Maintenance

```bash
# Enter LXC
pct enter 102

# Update Vaultwarden
cd /opt/vaultwarden
docker compose pull
docker compose up -d

# View logs
docker compose logs -f

# Restart
docker compose restart
```

## Backup

Vault data lives at `/opt/vaultwarden/data`. Take a Proxmox snapshot of CT 102 before any major changes.
