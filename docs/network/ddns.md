# Dynamic DNS

*Last updated: May 17, 2026*

## Overview

A shell script on OPNsense updates the `vpn.mystikal.dev` A record in Cloudflare every 5 minutes. WireGuard always connects to `vpn.mystikal.dev:51820` — if the ISP changes the public IP, the DNS record updates automatically.

## How It Works

1. Cron job fires every 5 minutes on OPNsense
2. Script fetches current public IP from `api.ipify.org`
3. Script calls Cloudflare API to update `vpn.mystikal.dev`
4. WireGuard clients always resolve the current IP via the domain

## Cloudflare DNS Record

| Setting | Value |
|---|---|
| Type | A |
| Name | vpn |
| Full domain | vpn.mystikal.dev |
| Proxy status | DNS only (grey cloud) |
| TTL | 60 seconds |

## API Token

A dedicated DDNS token with minimal permissions — separate from the Caddy DNS token.

| Setting | Value |
|---|---|
| Token name | DDNS |
| Permissions | Zone / DNS / Edit |
| Zone | mystikal.dev only |
| Stored | Vaultwarden secure note |

!!! tip
    Using separate tokens limits blast radius. If the DDNS token is compromised, it can only edit DNS — nothing else.

## Script

Location: `/usr/local/bin/cloudflare-ddns.sh`

```bash
#!/bin/sh
CF_API_TOKEN="<DDNS_API_TOKEN>"
CF_RECORD_NAME="vpn.mystikal.dev"

CURRENT_IP=$(curl -s https://api.ipify.org)

CF_ZONE_ID=$(curl -s -X GET "https://api.cloudflare.com/client/v4/zones?name=mystikal.dev" \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  -H "Content-Type: application/json" | grep -o '"id":"[^"]*"' | head -1 | cut -d'"' -f4)

CF_RECORD_ID=$(curl -s -X GET "https://api.cloudflare.com/client/v4/zones/$CF_ZONE_ID/dns_records?name=$CF_RECORD_NAME" \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  -H "Content-Type: application/json" | grep -o '"id":"[^"]*"' | head -1 | cut -d'"' -f4)

curl -s -X PUT "https://api.cloudflare.com/client/v4/zones/$CF_ZONE_ID/dns_records/$CF_RECORD_ID" \
  -H "Authorization: Bearer $CF_API_TOKEN" \
  -H "Content-Type: application/json" \
  --data "{\"type\":\"A\",\"name\":\"$CF_RECORD_NAME\",\"content\":\"$CURRENT_IP\",\"ttl\":60,\"proxied\":false}"
```

## Cron Job

```bash
# /etc/crontab
*/5 * * * * /usr/local/bin/cloudflare-ddns.sh
```

Verify: `cat /etc/crontab | grep ddns`

## Testing

```bash
# Run manually
sh /usr/local/bin/cloudflare-ddns.sh

# Verify DNS updated
nslookup vpn.mystikal.dev 1.1.1.1
```
