---
hide:
  - navigation
  - toc
---

# Frank's Homelab

A self-hosted homelab built on enterprise-grade hardware — documenting the setup, 
configuration, and lessons learned along the way.

---

## Infrastructure at a Glance

<div class="grid cards" markdown>

-   :material-router: **Routing & Firewall**

    OPNsense running on a Lenovo ThinkCentre M720q with an Intel i350-T4 quad-port NIC.

    [:octicons-arrow-right-24: OPNsense](infra/opnsense.md)

-   :material-server-network: **Proxmox Cluster**

    Three node cluster running on Dell Optiplex Micro PCs with a combined 72GB of RAM.

    [:octicons-arrow-right-24: Proxmox](infra/proxmox.md)

-   :material-switch: **Switching**

    Ubiquiti USW-24 PoE+ and USW-48 managing all wired connections.

    [:octicons-arrow-right-24: Network](network/overview.md)

-   :material-wifi: **Wireless**

    Two Ubiquiti U6+ access points providing full floor coverage.

    [:octicons-arrow-right-24: Network](network/overview.md)

-   :material-lightning-bolt: **Power**

    APC 1500VA UPS protecting all critical infrastructure.

    [:octicons-arrow-right-24: Hardware](infra/hardware.md)

-   :material-nas: **Storage**

    UGREEN NAS planned for centralized storage and backup.

    [:octicons-arrow-right-24: Hardware](infra/hardware.md)

</div>

---

## Hardware Summary

| Device | Role | 
|--------|------|
| Lenovo ThinkCentre M720q | OPNsense Router/Firewall |
| Dell Optiplex 3060 Micro x3 | Proxmox Cluster Nodes |
| Dell Optiplex 7010 Micro | Proxmox Cluster Node |
| Ubiquiti USW-24 PoE+ | Primary Switch |
| Ubiquiti USW-48 | Secondary Switch |
| Ubiquiti U6+ x2 | Wireless Access Points |
| APC 1500VA | UPS |

---

!!! note "Work in Progress"
    This documentation is actively being built out as the homelab evolves.