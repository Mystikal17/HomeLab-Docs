# OPNsense

*Last updated: May 13, 2026*

## Overview

OPNsense runs on a Lenovo ThinkCentre M720q with an Intel i350-T4 quad-port NIC. It handles all routing, firewall rules, DHCP, DNS, WireGuard VPN, and DDNS.

| Setting | Value |
|---|---|
| Hardware | Lenovo ThinkCentre M720q |
| NIC | Intel i350-T4 quad-port |
| Version | OPNsense 26.1.x |
| LAN IP | 192.168.1.1 |
| WAN | ISP DHCP |

## Services Running

| Service | Purpose |
|---|---|
| Kea DHCPv4 | DHCP for all VLANs |
| Unbound DNS | Internal DNS resolution + host overrides |
| WireGuard | VPN remote access |
| Suricata | IDS (ET Open ruleset) |
| DDNS script | Cloudflare DDNS via cron |

## Key Configurations

### Listen Interfaces

Both the web UI and Unbound DNS are restricted to internal interfaces only — WAN is excluded.

- **Web UI:** System → Settings → Administration → Listen Interfaces → LAN, VLANs, WireGuard
- **Unbound DNS:** Services → Unbound DNS → General → Listen Interfaces → LAN, VLANs, WireGuard

!!! tip
    This ensures ports 80, 443, and 53 are invisible on the WAN. External nmap shows the host as down.

### Unbound Host Overrides (Split DNS)

All mystikal.dev subdomains resolve internally to Caddy (192.168.10.70):

| Host | Domain | IP |
|---|---|---|
| vault | mystikal.dev | 192.168.10.70 |
| home | mystikal.dev | 192.168.10.70 |
| monitor | mystikal.dev | 192.168.10.70 |
| proxmox | mystikal.dev | 192.168.10.70 |
| unifi | mystikal.dev | 192.168.10.70 |

### SSH

SSH is disabled by default. Re-enable temporarily under System → Settings → Administration → Secure Shell only when shell access is needed. Disable immediately after.

## Security Posture

| Check | Status |
|---|---|
| SSH disabled | ✅ |
| Web UI internal only | ✅ |
| DNS internal only | ✅ |
| WireGuard only external entry point | ✅ |
| VLAN segmentation | ✅ |
| IoT isolated | ✅ |
| Guest isolated | ✅ |
| Suricata IDS | ✅ |

## Useful Commands

```bash
# Verify external exposure
nmap $(curl -s ifconfig.me)
# Expected: Host seems down

# Check WireGuard is listening
nmap -sU -p 51820 $(curl -s ifconfig.me)
# Expected: 51820/udp open|filtered

# Internal router scan
nmap 192.168.1.1
# Expected: 53, 80, 443 open (normal for internal)
```
