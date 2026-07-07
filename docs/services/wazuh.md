# Wazuh SIEM

*Last updated: July 4, 2026*

## Overview

Wazuh is an open source SIEM (Security Information and Event Management) platform. It collects logs from all homelab systems, analyzes them in real time, and alerts on suspicious activity.

| Setting | Value |
|---|---|
| URL | https://192.168.10.71 (wazuh.mystikal.dev — pending) |
| LXC | CT 104 (wazuh) |
| IP | 192.168.10.71 |
| VLAN | Servers (10) |
| Version | 4.11.2 |
| Install type | All-in-one |

## Architecture

Three components run on a single LXC:

| Component | Purpose | Port |
|---|---|---|
| Wazuh Manager | Receives and analyzes logs from agents | 1514 (agents), 1515 (registration) |
| Wazuh Indexer | Stores log data (OpenSearch/Java) | 9200 (localhost only) |
| Wazuh Dashboard | Web UI for alerts and events | 443 |

### Agent vs Syslog

**Agent** — installed directly on Linux machines. Provides deep visibility: process monitoring, file integrity, active response. Used on LXC containers.

**Syslog** — network log forwarding protocol. Used for devices that can't run an agent like OPNsense (FreeBSD). Less visibility than an agent but better than nothing.

## LXC Specs

```bash
pct create 104 local:vztmpl/debian-12-standard_12.12-1_amd64.tar.zst \
  --hostname wazuh \
  --storage local-lvm \
  --rootfs local-lvm:50 \
  --memory 8192 \
  --cores 4 \
  --net0 name=eth0,bridge=vmbr0,ip=192.168.10.71/24,gw=192.168.10.1,tag=10 \
  --unprivileged 0 \
  --features nesting=1 \
  --start 1
```

!!! warning
    `--unprivileged 0` creates a **privileged** container. Required for Wazuh indexer (OpenSearch) which needs low-level system access.

!!! note "Why Servers VLAN"
    Wazuh is a service/workload, not management infrastructure. Servers VLAN limits blast radius — if compromised, attacker is isolated from Proxmox and OPNsense on Management VLAN.

## Installation

```bash
# Download installer and config
curl -sO https://packages.wazuh.com/4.11/wazuh-install.sh
curl -sO https://packages.wazuh.com/4.11/config.yml

# Edit config.yml — set all three IPs to 192.168.10.71
nano config.yml

# Run all-in-one install
apt install sudo -y
bash wazuh-install.sh -a
```

### config.yml

```yaml
nodes:
  indexer:
    - name: node-1
      ip: "192.168.10.71"
  server:
    - name: wazuh-1
      ip: "192.168.10.71"
  dashboard:
    - name: dashboard
      ip: "192.168.10.71"
```

!!! important
    Save the generated admin credentials to Vaultwarden **immediately** before doing anything else.

## Verify Installation

```bash
systemctl status wazuh-manager
systemctl status wazuh-indexer
systemctl status wazuh-dashboard

# Check ports
ss -tlnp | grep -E "443|9200|1514|1515"
```

Expected ports:

| Port | Process | Notes |
|---|---|---|
| 443 | node (dashboard) | Web UI |
| 1514 | wazuh-remoted | Agent ingestion |
| 1515 | wazuh-authd | Agent registration |
| 9200 | java (indexer) | Localhost only — correct by design |

## Agent Installation

### Vaultwarden (CT102)

```bash
pct enter 102

# Add Wazuh repo with GPG verification
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg \
  --no-default-keyring \
  --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import
chmod 644 /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] \
  https://packages.wazuh.com/4.x/apt/ stable main" \
  | tee /etc/apt/sources.list.d/wazuh.list

apt update && apt install wazuh-agent -y
```

Configure manager IP in `/var/ossec/etc/ossec.conf`:

```xml
<client>
  <server>
    <address>192.168.10.71</address>
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
</client>
```

```bash
systemctl enable wazuh-agent
systemctl start wazuh-agent
```

## Agent Coverage Plan

| Source | Method | Status |
|---|---|---|
| CT102 Vaultwarden | Agent | Installed — pending version fix |
| CT103 Homepage | Agent | Pending |
| pve1 Proxmox host | Agent | Pending |
| OPNsense | Syslog → port 1514 | Pending |

## Known Issues & Fixes

### Version Mismatch

Agent version must be <= manager version. If agent is newer:

```
ERROR: Agent version must be lower or equal to manager version
```

**Fix:** Upgrade the manager following official Wazuh upgrade docs. Never downgrade agents.

### Dashboard Cert Permission Error

After upgrade or cert regeneration:

```
EACCES: permission denied, open '/etc/wazuh-dashboard/certs/dashboard-key.pem'
```

**Fix:**
```bash
chown -R wazuh-dashboard:wazuh-dashboard /etc/wazuh-dashboard/certs/
chmod 500 /etc/wazuh-dashboard/certs/
chmod 400 /etc/wazuh-dashboard/certs/*
systemctl restart wazuh-dashboard
```

### Password Reset

If admin password is lost or mismatched after upgrade:

```bash
/usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh --change-all
systemctl restart wazuh-indexer wazuh-manager wazuh-dashboard
```

Save all generated passwords to Vaultwarden immediately.

## Snapshot Before Upgrades

Always snapshot CT104 before any major changes:

```bash
pct snapshot 104 pre-wazuh-upgrade --description "Before Wazuh upgrade"
```

Rollback if needed:

```bash
pct stop 104
pct rollback 104 pre-wazuh-upgrade
pct start 104
```

## Key Lessons Learned

- Save credentials to Vaultwarden immediately — never overwrite without confirming old ones still work
- Snapshot before every major change
- Read logs before touching anything — `journalctl -u wazuh-dashboard` and `ss -tlnp` diagnose most issues
- Upgrade manager not agents
- EACCES means wrong ownership — check `chown` and `chmod` first

## Next Steps

- Upgrade manager to 4.14.x using official upgrade guide
- Register Vaultwarden agent
- Install agents on CT103 and pve1
- Configure OPNsense syslog
- Add `wazuh.mystikal.dev` to Caddy

[:material-file-pdf-box: Download session notes](../session-logs/assets/pdfs/homelab-session-2026-07-04.pdf){ .md-button }

## Upgrade Notes (4.11.2 → 4.14.6)

### Upgrade Order

Always upgrade in this order:

```bash
apt install wazuh-indexer -y
apt install wazuh-manager -y
apt install wazuh-dashboard -y
```

### Post-Upgrade Checklist

After every upgrade run these immediately before restarting:

```bash
# Fix dashboard cert ownership
chown -R wazuh-dashboard:wazuh-dashboard /etc/wazuh-dashboard/certs/
chmod 500 /etc/wazuh-dashboard/certs/
chmod 400 /etc/wazuh-dashboard/certs/*

# Fix indexer cert ownership
chown -R wazuh-indexer:wazuh-indexer /etc/wazuh-indexer/certs/
chmod 500 /etc/wazuh-indexer/certs/
chmod 400 /etc/wazuh-indexer/certs/*

# Restart in order
systemctl restart wazuh-indexer
systemctl restart wazuh-manager
systemctl restart wazuh-dashboard
```

### IPv6/IPv4 Localhost Fix

If dashboard shows `ECONNREFUSED ::1:9200` — `localhost` resolved to IPv6. Fix:

```bash
sed -i 's/opensearch.hosts: https:\/\/localhost:9200/opensearch.hosts: https:\/\/127.0.0.1:9200/' \
  /etc/wazuh-dashboard/opensearch_dashboards.yml
systemctl restart wazuh-dashboard
```

!!! warning
    Always use explicit IP addresses (`127.0.0.1`) instead of `localhost` in service configs on IPv6-enabled systems.

## Upgrade Reference

Follow this exact sequence for any future Wazuh upgrade:

```bash
# 1. Snapshot first
pct snapshot 104 pre-wazuh-upgrade-<date>

# 2. Upgrade in order
apt install wazuh-indexer -y
apt install wazuh-manager -y
apt install wazuh-dashboard -y

# 3. Regenerate certs
rm /root/wazuh-install-files.tar
bash wazuh-install.sh -g
tar -xvf wazuh-install-files.tar

# 4. Copy dashboard certs
cp wazuh-install-files/dashboard.pem /etc/wazuh-dashboard/certs/dashboard.pem
cp wazuh-install-files/dashboard-key.pem /etc/wazuh-dashboard/certs/dashboard-key.pem
cp wazuh-install-files/root-ca.pem /etc/wazuh-dashboard/certs/

# 5. Copy indexer certs
cp wazuh-install-files/node-1.pem /etc/wazuh-indexer/certs/wazuh-indexer.pem
cp wazuh-install-files/node-1-key.pem /etc/wazuh-indexer/certs/wazuh-indexer-key.pem
cp wazuh-install-files/root-ca.pem /etc/wazuh-indexer/certs/

# 6. Fix ownership and permissions
chown -R wazuh-dashboard:wazuh-dashboard /etc/wazuh-dashboard/certs/
chown -R wazuh-indexer:wazuh-indexer /etc/wazuh-indexer/certs/
chmod 500 /etc/wazuh-dashboard/certs/ /etc/wazuh-indexer/certs/
chmod 400 /etc/wazuh-dashboard/certs/* /etc/wazuh-indexer/certs/*

# 7. Reset passwords
/usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh --change-all
# Save ALL passwords to Vaultwarden immediately

# 8. Restart in order
systemctl restart wazuh-indexer
systemctl restart wazuh-manager
systemctl restart wazuh-dashboard

# 9. Verify indexer health
curl -k -u admin:<password> https://localhost:9200/_cluster/health?pretty

# 10. Confirm IPv4 in dashboard config
grep opensearch.hosts /etc/wazuh-dashboard/opensearch_dashboards.yml
# Must show 127.0.0.1 not localhost
```

!!! warning "IPv6/IPv4 Gotcha"
    `opensearch.hosts` must use `https://127.0.0.1:9200` not `https://localhost:9200`. On Debian, `localhost` resolves to `::1` (IPv6) but the indexer only listens on `127.0.0.1` (IPv4). This causes `ECONNREFUSED ::1:9200`.

## CIS Benchmark — Vaultwarden

First scan results (July 4, 2026):

| Result | Count |
|---|---|
| Passed | 70 |
| Failed | 107 |
| Not Applicable | 30 |
| **Score** | **39%** |

### Priority Failures

| Category | Issue | Risk |
|---|---|---|
| Audit logging | auditd not installed | No kernel-level audit trail |
| Password policies | libpam-pwquality missing | No complexity enforcement |
| SSH hardening | PermitRootLogin not disabled | Brute force risk |
| Package CVEs | 7 Critical, 52 High | Outdated packages |

Hardening work planned for future session.
