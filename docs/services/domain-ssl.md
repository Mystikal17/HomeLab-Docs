# Domain & SSL

*Last updated: May 16, 2026*

## Overview

`mystikal.dev` is registered through Cloudflare. Let's Encrypt certificates are issued automatically by Caddy using the DNS-01 challenge via the Cloudflare API — no ports need to be open.

## How DNS-01 Works

Traditional cert issuance (HTTP-01) requires port 80 open. DNS-01 works differently:

1. Caddy asks Let's Encrypt for a certificate
2. Let's Encrypt says: prove you own the domain by adding a TXT record
3. Caddy calls the Cloudflare API to add that TXT record automatically
4. Let's Encrypt verifies and issues the cert
5. No ports need to be open — everything goes through DNS

## Domain

| Setting | Value |
|---|---|
| Domain | mystikal.dev |
| Registrar | Cloudflare |
| Cost | $12/year |
| DNS | Cloudflare (included) |

## Cloudflare API Token

| Setting | Value |
|---|---|
| Token name | Caddy DNS |
| Permissions | Zone / DNS / Edit |
| Zone | mystikal.dev only |
| Stored | /etc/caddy/caddy.env |

## Caddy Cloudflare Plugin

Standard Caddy from apt doesn't include the Cloudflare DNS plugin. A custom binary was built with `xcaddy`:

```bash
# Install Go 1.22+ (Debian 12 ships with Go 1.19 which is too old)
curl -OL https://go.dev/dl/go1.22.3.linux-amd64.tar.gz
rm -rf /usr/local/go
tar -C /usr/local -xzf go1.22.3.linux-amd64.tar.gz
export PATH=/usr/local/go/bin:$PATH

# Build Caddy with Cloudflare plugin
apt install -y xcaddy
xcaddy build --with github.com/caddy-dns/cloudflare
mv /root/caddy /usr/bin/caddy
chmod +x /usr/bin/caddy
```

| Setting | Value |
|---|---|
| Caddy version | 2.11.3 |
| Plugin | github.com/caddy-dns/cloudflare |
| Binary | /usr/bin/caddy |

## Cloudflare DNS Records

All subdomains point to Caddy's IP (`192.168.10.70`) with proxy disabled:

| Record | Type | Value | Proxy |
|---|---|---|---|
| vault | A | 192.168.10.70 | DNS only ☁️ |
| home | A | 192.168.10.70 | DNS only ☁️ |
| monitor | A | 192.168.10.70 | DNS only ☁️ |
| proxmox | A | 192.168.10.70 | DNS only ☁️ |
| unifi | A | 192.168.10.70 | DNS only ☁️ |
| vpn | A | \<public IP\> | DNS only ☁️ |

!!! warning
    Proxy must be **DNS only** (grey cloud). If proxied (orange cloud), Cloudflare tries to forward traffic to a private IP from the internet, which fails.

## Split DNS

Cloudflare won't publicly resolve private IPs, so OPNsense Unbound handles internal resolution:

Services → Unbound DNS → Host Overrides → all subdomains → `192.168.10.70`

- **Inside the network:** Unbound resolves directly to `192.168.10.70`
- **Via WireGuard:** Routes through the tunnel, Unbound resolves correctly
- **Public internet:** DNS record exists but points to private IP — unreachable without VPN

## Certificate Renewal

Caddy renews Let's Encrypt certificates automatically before expiry (every 90 days). No manual action needed as long as:

- Caddy is running (`systemctl status caddy`)
- Cloudflare API token is valid
- DNS record still exists
