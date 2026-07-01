# Hardware Inventory

*Last updated: May 8, 2026*

## Network

| Device | Model | Role |
|---|---|---|
| Firewall/Router | Lenovo ThinkCentre M720q + Intel i350-T4 | OPNsense 26.1.x |
| Switch 1 | Ubiquiti USW-24 PoE+ | Primary switch |
| Switch 2 | Ubiquiti USW-48 Standard | Uplinked to USW-24 |
| Access Points | 2x Ubiquiti U6+ | Managed by UniFi controller |
| UPS | APC 1500 | Battery backup |

## Compute

| Device | Role | IP |
|---|---|---|
| Dell OptiPlex 3060 Micro (x3) | Proxmox cluster nodes | 192.168.1.10–.12 |
| Dell OptiPlex 7010 | Proxmox additional node | — |
| UGREEN NAS | Network storage | — |

## LXC Containers

| CT ID | Hostname | IP | VLAN | Services |
|---|---|---|---|---|
| 101 | unifi-os-server | 192.168.1.69 | Management | UniFi Network Controller |
| 102 | vaultwarden | 192.168.10.70 | Servers | Vaultwarden + Caddy |
| 103 | homepage | 192.168.1.71 | Management | Homepage + Uptime Kuma |

## UPS Load

**On battery backup (APC 1500):**

- Proxmox nodes (have storage/data)
- UGREEN NAS
- Network switches and router

**On surge protector only:**

- Monitors/displays
- Passive devices
