# Homelab Hardening — 2FA & Token Security

*Last updated: June 25, 2026*

## Summary

| Task | Status |
|---|---|
| Vaultwarden config audit | ✅ |
| Vaultwarden port binding locked to 127.0.0.1 | ✅ |
| Vaultwarden TOTP enrolled | ✅ |
| Proxmox non-root admin user created | ✅ |
| Proxmox TOTP on both accounts | ✅ |
| Cloudflare API token rotated | ✅ |
| Token moved to env variable | ✅ |
| UniFi reverse proxy fixed (WebSocket + CORS) | ✅ |

---

## Vaultwarden

### Config

`/opt/vaultwarden/.env` verified:

```env
SIGNUPS_ALLOWED=false
ADMIN_TOKEN=$$argon2id$$...  # Argon2id hashed, not plaintext
```

Generate hashed token:
```bash
docker exec -it vaultwarden /vaultwarden hash --preset owasp
```

### Port Binding

Changed from `0.0.0.0:8080` (all interfaces) to `127.0.0.1:8080` (localhost only):

```bash
sed -i 's/- "8080:80"/- "127.0.0.1:8080:80"/' /opt/vaultwarden/docker-compose.yml
cd /opt/vaultwarden && docker compose down && docker compose up -d
```

Verify: `docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"`  
Expected: `127.0.0.1:8080->80/tcp`

### TOTP

vault.mystikal.dev → Account Settings → Security → Two-step Login → Authenticator App → Manage → scan QR → confirm with live code → save recovery code as a Vaultwarden secure note.

---

## Proxmox

### Non-Root Admin User

```bash
pveum user add frank@pve --comment "Frank Admin"
pveum passwd frank@pve
pveum aclmod / -user frank@pve -role Administrator
```

!!! warning "Realm selection"
    Login with username `frank`, realm **Proxmox VE authentication server** — not PAM. Wrong realm = 401 error.

### TOTP

Top-right user icon → TFA → Add TOTP → scan QR → confirm. Enrolled on both `frank@pve` and `root@pam`.

Root (`root@pam`) is now break-glass only — use `frank@pve` for all daily access.

---

## Cloudflare API Token Security

### Problem

Token was stored in plaintext in `/etc/caddy/Caddyfile` — anyone with read access could hijack DNS records for `mystikal.dev`.

### Fix

**Step 1** — Rotate the token:
Cloudflare → My Profile → API Tokens → Roll

**Step 2** — Store in protected env file:
```bash
echo 'CLOUDFLARE_API_TOKEN=<new_token>' > /etc/caddy/caddy.env
chown caddy:caddy /etc/caddy/caddy.env
chmod 600 /etc/caddy/caddy.env
```

**Step 3** — Load via systemd override:
```bash
mkdir -p /etc/systemd/system/caddy.service.d/
cat > /etc/systemd/system/caddy.service.d/override.conf << 'EOF'
[Service]
EnvironmentFile=/etc/caddy/caddy.env
EOF
systemctl daemon-reload
```

**Step 4** — Reference in Caddyfile:
```
dns cloudflare {env.CLOUDFLARE_API_TOKEN}
```

---

## UniFi Reverse Proxy Fix

### Problem

`unifi.mystikal.dev` hung after MFA. The UI attempts WebSocket connections to `wss://unifi.mystikal.dev/api/ws/system` after login. UniFi OS has its own nginx inside a podman container that validates the WebSocket `Origin` header against a CORS allowlist. Since `mystikal.dev` wasn't in the allowlist, nginx returned HTTP 500 on every WebSocket handshake.

### Fix

Inside the UniFi OS podman container:

```bash
su - uosserver -c "podman exec -it uosserver bash"
```

```bash
echo '~^https://unifi\.mystikal\.dev$ $http_origin;' \
  > /data/unifi-core/config/http/cors-origin-mystikal.conf

echo '~^https://unifi\.mystikal\.dev$ "true";' \
  > /data/unifi-core/config/http/cors-credentials-mystikal.conf

nginx -s reload
```

These files live in the `uosserver_data` podman volume and persist through container updates.

---

[:material-file-pdf-box: Download full session notes](../session-logs/assets/pdfs/homelab-session-2026-06-25.pdf){ .md-button }
