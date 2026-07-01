# WireGuard VPN

*Last updated: May 8, 2026*

## Overview

WireGuard is configured on OPNsense to provide secure remote access to all homelab VLANs. It's the only intentional entry point from the internet.

| Setting | Value |
|---|---|
| Interface | wg0 |
| Listen port | 51820 (UDP) |
| VPN subnet | 10.0.0.0/24 |
| OPNsense VPN IP | 10.0.0.1 |
| Endpoint domain | vpn.mystikal.dev |

## Peers

| Device | VPN IP | Public Key (prefix) |
|---|---|---|
| iPhone | 10.0.0.2/32 | wswFPdOy... |
| MacBook Pro M5 | 10.0.0.3/32 | ZLKPJ+ET... |

Next available peer IP: `10.0.0.4/32`

## OPNsense Config

VPN → WireGuard → Instances:

| Setting | Value |
|---|---|
| Name | wg0 |
| Listen port | 51820 |
| Tunnel address | 10.0.0.1/24 |
| DNS | 192.168.1.1 |

## Split Tunnel

All peers use split tunnel — only homelab traffic routes through VPN:

```
AllowedIPs = 192.168.1.0/24, 192.168.10.0/24, 192.168.20.0/24, 10.0.0.0/24
```

Regular internet traffic goes direct, not through the tunnel.

## Key Concepts

| Key | Where it goes |
|---|---|
| Device private key | Stays on device only — in `[Interface] PrivateKey` |
| Device public key | Goes into OPNsense as the peer's public key |
| OPNsense public key | Goes into device config as `[Peer] PublicKey` |
| OPNsense private key | Stays on OPNsense only — never shared |

!!! warning
    After adding any new peer in OPNsense, WireGuard must be restarted: VPN → WireGuard → General → toggle off → save → toggle on → save.

## Adding a New Peer

1. Assign next available IP (10.0.0.4/32, etc.)
2. Install WireGuard on device, generate keypair
3. Add peer in OPNsense with device's public key and assigned IP
4. Restart WireGuard in OPNsense
5. Configure device with OPNsense's public key as `[Peer] PublicKey`
6. Set endpoint to `vpn.mystikal.dev:51820`
7. Set AllowedIPs for split tunnel

## Verification

```bash
# SSH to OPNsense (temporarily enable SSH first)
ssh root@192.168.1.1
wg show
```

Expected output shows all peers with recent handshakes.

## Firewall Rule

WAN → Allow → UDP → Any → WAN:51820 — required for WireGuard to accept incoming connections.
