# VLAN Architecture

*Last updated: May 10, 2026*

## Overview

VLANs segment traffic between device categories — servers, trusted devices, IoT, and guests. All routing is handled by OPNsense as the third-party gateway.

## Network Topology

| Device | Role |
|---|---|
| Lenovo ThinkCentre M720q | OPNsense firewall/router (bare metal) |
| Ubiquiti USW-24 PoE+ | Primary switch, APs connected here |
| Ubiquiti USW-48 Standard | Uplinked to USW-24 port 24 |
| 2x Ubiquiti U6+ | Access points (ports 15 & 16 on USW-24) |
| UniFi Network Controller | Proxmox LXC (192.168.1.69) |
| Proxmox | Hypervisor (192.168.1.10) |

**Physical path:**
```
Internet → M720q WAN → OPNsense
OPNsense LAN (em0) → USW-24 Port 1 (trunk, all VLANs)
USW-24 Port 24 → USW-48 Port 48 (trunk, all VLANs)
USW-24 Ports 15,16 → U6+ APs (trunk, all VLANs)
```

## VLAN Table

| VLAN ID | Name | Subnet | Purpose | WiFi SSID |
|---|---|---|---|---|
| 1 | Management | 192.168.1.0/24 | Router, Proxmox, switches, UniFi | — |
| 10 | Servers | 192.168.10.0/24 | LXC containers, self-hosted services | — |
| 20 | Trusted | 192.168.20.0/24 | Personal devices — PCs, phones, laptops | Main SSID |
| 30 | IoT | 192.168.30.0/24 | TVs, smart home devices | IoT + Media SSIDs |
| 40 | Guest | 192.168.40.0/24 | Isolated guest access | Guest SSID |

## DHCP Ranges

| VLAN | Range |
|---|---|
| Management | 192.168.1.100 – 192.168.1.200 |
| Servers | 192.168.10.100 – 192.168.10.200 |
| Trusted | 192.168.20.100 – 192.168.20.200 |
| IoT | 192.168.30.100 – 192.168.30.200 |
| Guest | 192.168.40.100 – 192.168.40.200 |

!!! note
    DHCP is handled by Kea DHCPv4 on OPNsense. Dnsmasq was disabled to prevent port 67 conflicts.

## OPNsense VLAN Interfaces

All VLANs are parented to `em0` (LAN):

| Interface | Tag | IP |
|---|---|---|
| OPT1 (Servers) | 10 | 192.168.10.1/24 |
| OPT3 (Trusted) | 20 | 192.168.20.1/24 |
| OPT4 (IoT) | 30 | 192.168.30.1/24 |
| OPT5 (Guest) | 40 | 192.168.40.1/24 |

## Firewall Rules Summary

| VLAN | Rule | Source | Destination |
|---|---|---|---|
| Trusted | Pass | Trusted net | Management (192.168.1.0/24) |
| Trusted | Pass | Trusted net | IoT (192.168.30.0/24) |
| Trusted | Pass | Trusted net | Any |
| IoT | Block | IoT net | Management, Servers, Trusted, Guest |
| IoT | Pass | IoT net | Any (internet only) |
| Guest | Block | Guest net | All internal VLANs |
| Guest | Pass | Guest net | Any (internet only) |
| Servers | Pass | Servers net | Any (temporary — to be tightened) |
| WAN | Pass | Any | WAN:51820 UDP (WireGuard) |

!!! warning
    Servers VLAN rules are temporary placeholders. Tighten once all services are migrated there.

## Switch Port Config

| Port | Role | Native | Tagged |
|---|---|---|---|
| USW-24 Port 1 | OPNsense uplink | 1 | 10,20,30,40 |
| USW-24 Ports 15,16 | AP uplinks | 1 | 10,20,30,40 |
| USW-24 Port 24 | USW-48 uplink | 1 | 10,20,30,40 |
| USW-48 Port 48 | USW-24 uplink | 1 | 10,20,30,40 |
| All others | Access ports | 1 | — |

## WiFi SSIDs

| SSID | VLAN | Purpose |
|---|---|---|
| Main | Trusted (20) | Personal devices |
| Media | IoT (30) | TVs, streaming |
| IoT | IoT (30) | Smart home |
| Guest | Guest (40) | Isolated guest access |
