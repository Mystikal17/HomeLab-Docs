# Printer VLAN

*Last updated: May 12, 2026*

## Overview

The HP OfficeJet Pro 8034e is placed on the Trusted VLAN with a firewall rule blocking it from initiating connections to other trusted devices.

## VLAN Decision

The printer was initially placed on IoT VLAN but AirPrint relies on mDNS (Bonjour) which doesn't cross VLAN boundaries. Attempts to fix this failed:

- UniFi Gateway mDNS Proxy — doesn't work with OPNsense as third-party gateway
- Avahi mDNS repeater on OPNsense — requires D-Bus which is unavailable on FreeBSD

**Final decision: Trusted VLAN** — HP is a reputable brand, AirPrint works natively, and a block rule prevents the printer from initiating connections to other trusted devices.

## Network Details

| Setting | Value |
|---|---|
| Device | HP OfficeJet Pro 8034e |
| VLAN | Trusted (VLAN 20) |
| IP | 192.168.20.120 (reserved) |
| MAC | bc:0f:f3:ed:8c:80 |

## Firewall Rules (Trusted VLAN — order matters)

| # | Action | Source | Destination | Description |
|---|---|---|---|---|
| 1 | Pass | Trusted net | 255.255.255.255:67 | Allow DHCP (auto) |
| 2 | **Block** | 192.168.20.120 | 192.168.20.0/24 | Block printer → Trusted |
| 3 | Pass | Trusted net | 192.168.1.0/24 | Trusted → Management |
| 4 | Pass | Trusted net | 192.168.30.0/24 | Trusted → IoT |
| 5 | Pass | Trusted net | Any | Trusted allow all |

!!! warning
    Rule 2 **must** be above Rule 5. OPNsense is first-match — if "allow all" fires first, the block never applies.

## What the Printer Can and Cannot Do

| Action | Allowed |
|---|---|
| Print jobs from trusted devices | ✅ |
| Printer initiating to trusted devices | ❌ Blocked |
| Printer internet access | ✅ |
| AirPrint discovery | ✅ (same VLAN) |
| Printer to management VLAN | ❌ Blocked |
