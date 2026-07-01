# WAN Hardening

*Last updated: May 13, 2026*

## Overview

All services are restricted to internal interfaces only. The public IP shows as completely dark — no open ports visible from the internet.

## Port Audit Results

**Before hardening:**

| Port | Service | Issue |
|---|---|---|
| 22 | SSH | Left enabled after WireGuard setup |
| 53 | DNS | Unbound listening on all interfaces including WAN |
| 80 | HTTP | OPNsense web UI accessible on WAN |
| 443 | HTTPS | OPNsense web UI accessible on WAN |

**After hardening:**

```bash
nmap 74.105.76.137
# Note: Host seems down. No open ports.
```

!!! note
    WireGuard runs on UDP 51820 which doesn't appear in standard TCP nmap scans. It's the only intentional entry point and is invisible to casual scanning.

## Changes Made

### SSH Disabled

System → Settings → Administration → Secure Shell → unchecked

Re-enable temporarily if shell access is needed. Disable immediately after.

### Web UI — Internal Interfaces Only

System → Settings → Administration → Listen Interfaces

Changed from: All interfaces  
Changed to: LAN, Servers, Trusted, IoT, Guest, WireGuardInterface

### Unbound DNS — Internal Interfaces Only

Services → Unbound DNS → General → Listen Interfaces

Changed from: All interfaces  
Changed to: LAN, Servers, Trusted, IoT, Guest, WireGuardInterface

## Current Exposure

| Check | Status |
|---|---|
| Public IP visible ports | None (host appears down) |
| WireGuard | UDP 51820 only — invisible to TCP scans |
| Web UI | Internal + WireGuard only |
| DNS | Internal + WireGuard only |
| SSH | Disabled |

## Verification Commands

```bash
# What the internet sees (should show host as down)
nmap $(curl -s ifconfig.me)

# Internal router scan (53, 80, 443 expected open internally)
nmap 192.168.1.1

# Check WireGuard UDP port
nmap -sU -p 51820 $(curl -s ifconfig.me)
# Expected: 51820/udp open|filtered wireguard
```

Run these periodically to verify posture hasn't drifted.
