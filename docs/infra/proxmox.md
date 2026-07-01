# Proxmox

*Last updated: May 8, 2026*

## Cluster

Four nodes running Proxmox VE:

| Node | Device | IP |
|---|---|---|
| pve1 | Dell OptiPlex 3060 Micro | 192.168.1.10 |
| pve2 | Dell OptiPlex 3060 Micro | 192.168.1.11 |
| pve3 | Dell OptiPlex 3060 Micro | 192.168.1.12 |
| pve4 | Dell OptiPlex 7010 | — |

Storage: UGREEN NAS

## Post-Install Setup

Run on each node after installation:

```bash
# Disable enterprise repo
sed -i 's/^deb/#deb/' /etc/apt/sources.list.d/pve-enterprise.list

# Add no-subscription repo
echo "deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription" \
  > /etc/apt/sources.list.d/pve-no-subscription.list

# Update
apt update && apt dist-upgrade
```

## User Setup

Daily admin work is done as `frank@pve` — root is break-glass only.

```bash
pveum user add frank@pve --comment "Frank Admin"
pveum passwd frank@pve
pveum aclmod / -user frank@pve -role Administrator
```

!!! warning "Realm selection"
    At the Proxmox login screen, use username `frank` and select **Proxmox VE authentication server** from the realm dropdown — not PAM. Using PAM causes a 401 error.

## 2FA

TOTP enrolled on both `frank@pve` and `root@pam` via the web UI:

Top-right user icon → TFA → Add TOTP → scan QR → confirm with live code.

## LXC Static IPs

Always set static IPs in **both** places — inside the LXC and in Proxmox config. One without the other is not sufficient when DHCP is active.

```bash
# Set static IP in Proxmox config
pct set 101 --net0 name=eth0,bridge=vmbr0,hwaddr=BC:24:11:E9:6F:44,ip=192.168.1.69/24,gw=192.168.1.1
pct set 102 --net0 name=eth0,bridge=vmbr0,hwaddr=BC:24:11:E0:E4:9B,ip=192.168.10.70/24,gw=192.168.10.1,tag=10
pct set 103 --net0 name=eth0,bridge=vmbr0,hwaddr=BC:24:11:38:5A:B1,ip=192.168.1.71/24,gw=192.168.1.1
```

!!! note
    The `tag=10` parameter places the LXC on VLAN 10. Without it the LXC stays on the untagged Management VLAN regardless of the IP set inside.

## Useful Commands

```bash
pct list              # list all containers
pct enter <id>        # enter container shell
pct reboot <id>       # reboot container
pct set <id> --net0   # update network config
```
