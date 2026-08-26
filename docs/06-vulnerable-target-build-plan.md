# 06 — Vulnerable Target Build Plan

## Purpose

The `Vuln-Ubuntu-Target` VM is the authorized scan/remediation target for the integrated cyber defense lab. It supports vulnerability management, firewall logging, threat-hunting telemetry, and before/after remediation evidence.

## VM Settings

| Setting | Value |
|---|---|
| VM name | `Vuln-Ubuntu-Target` |
| Guest OS | Ubuntu Server LTS 64-bit |
| CPU | 1–2 vCPU |
| RAM | 2 GB preferred; 1 GB acceptable if host resources are constrained |
| Disk | 20 GB thin provisioned |
| CD/DVD | Ubuntu Server LTS ISO |
| Network Adapter | Host-only / same VMware LAN side used by OPNsense LAN |

## Network Placement

Current OPNsense state:

| Interface | Role | Network |
|---|---|---|
| WAN `em0` | VMware NAT/outside | `192.168.109.0/24` |
| LAN `em1` | VMware host-only/lab LAN | `192.168.153.0/24` |

For the target VM:

```text
Network Adapter = Host-only / same network as OPNsense LAN
Expected subnet = 192.168.153.0/24
Expected gateway = OPNsense LAN IP
```

If VMware host-only DHCP gives the target an address instead of OPNsense, document that observation and adjust after confirming the OPNsense LAN/DHCP design.

## Install Choices

During Ubuntu Server install:

- Use a generic lab hostname such as `vuln-target01`.
- Install OpenSSH Server if offered.
- Do not enable paid Ubuntu Pro.
- Do not add real credentials, employer/client names, or secrets to screenshots.
- Keep the VM inside the lab network; do not expose it publicly.

## Install Status

Operator completed installation and rebooted the `Vuln-Ubuntu-Target` VM.

Install choices recorded:

- Standard Ubuntu Server installation
- Network interface `ens33` received `192.168.153.129/24` during install
- Entire 20 GB virtual disk used
- LVM enabled
- LUKS disk encryption not enabled
- OpenSSH Server installed
- Ubuntu Pro not enabled

## First Validation Commands

After install, run on the target:

```bash
ip addr
ip route
resolvectl status
ping -c 4 <opnsense-lan-ip>
ping -c 4 8.8.8.8
```

Expected outcomes:

- Target has a valid lab LAN IP.
- Default gateway points toward OPNsense LAN.
- Target can reach OPNsense.
- Internet may require OPNsense LAN rules/NAT/DNS verification.

Observed post-install target validation:

```text
Target IP: 192.168.153.130/24
Default route: 192.168.153.1 dev ens33
DNS server: 192.168.153.1
Ping OPNsense LAN 192.168.153.128: success, 0% packet loss
Ping 8.8.8.8: Network is unreachable
```

Interpretation:

- The target is on the host-only lab subnet and can reach OPNsense LAN.
- VMware host-only DHCP is currently assigning the target gateway/DNS as `192.168.153.1`.
- Host-only does not provide internet routing, so internet fails.
- For firewall-lab correctness, the target should use OPNsense as its default gateway and DNS. Operator chose the cleaner Option B: move the lab LAN to dedicated `VMnet2` where OPNsense owns DHCP. See `docs/07-dedicated-vmnet2-opnsense-dhcp-plan.md`.

Observed post-VMnet2 validation:

```text
Target interface: ens33
Target IP: 10.10.10.167/24
Default route: via 10.10.10.254 dev ens33 proto dhcp
DNS server: 10.10.10.254
DefaultRoute: yes
```

Interpretation:

- OPNsense DHCP on VMnet2 is working.
- The target is now on the dedicated lab LAN.
- The target uses OPNsense LAN (`10.10.10.254`) as gateway and DNS.
- This fixes the earlier VMware host-only DHCP issue.

Post-VMnet2 connectivity validation:

```text
ping -c 4 10.10.10.254: 4 transmitted, 4 received, 0% packet loss
ping -c 4 8.8.8.8: 4 transmitted, 4 received, 0% packet loss
```

Interpretation:

- Target can reach OPNsense LAN.
- Target can route through OPNsense WAN to the internet.
Additional lease-display troubleshooting:

```text
sudo dhclient -r ens33: dhclient command not found
sudo dhclient -v ens33: dhclient command not found
sudo networkctl renew ens33: command accepted/no visible error
Target rebooted; OPNsense Kea DHCPv4 lease GUI still empty
```

Interpretation:

- Ubuntu Server image does not include `dhclient` by default.
- `networkctl renew ens33` is the appropriate systemd-networkd renewal path.
- Because the target still has OPNsense gateway/DNS and working routing, DHCP function is validated even if the GUI lease table remains empty.
- Next read-only checks: OPNsense Kea DHCP log file and Kea/Dnsmasq service status; do not change DHCP config solely to make the lease table populate.

## Screenshot Markers

| Filename | Capture |
|---|---|
| `05-target-address-route-dns-baseline.png` | VMware settings for CPU/RAM/disk/network |
| `07-opnsense-dhcp-leases.png` | Target terminal showing IP/gateway/DNS after install |
