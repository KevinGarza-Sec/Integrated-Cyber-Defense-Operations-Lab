# 05 — VM Build Sheet and Download Links

## Target VM Count

Use **five total VMs** for the complete lab design:

| # | VM Name | Role | New or Existing |
|---:|---|---|---|
| 1 | `OPNsense-Firewall` | Firewall/router/IDS | New |
| 2 | `Ubuntu Server` | SOC server / Wazuh | Existing |
| 3 | `Windows 11 x64` | Windows endpoint / Sysmon / Wazuh Agent | Existing |
| 4 | `Ubuntu` | Kali analyst VM | Existing, despite VMware display name |
| 5 | `Vuln-Ubuntu-Target` | Vulnerable Linux scan/remediation target | New, strongly recommended |

Minimum practical version is four VMs by skipping the vulnerable Linux VM and running vulnerable services in Docker, but the five-VM layout gives the cleanest technical coverage.

## Download Links

| Component | Official Download Link | What to Download |
|---|---|---|
| OPNsense | https://opnsense.org/download/ | `amd64` architecture, `dvd` image type for VM install. Extract the downloaded `.bz2` to get the `.iso`. |
| OPNsense install docs | https://docs.opnsense.org/manual/install.html | Use for install/reference only. |
| Ubuntu Server LTS | https://ubuntu.com/download/server | Latest Ubuntu Server LTS ISO. Do not enable paid Ubuntu Pro during install. |
| Debian alternative | https://www.debian.org/download | Optional alternative target OS; use amd64 netinst ISO. |
| Greenbone/OpenVAS Community Edition | https://greenbone.github.io/docs/latest/ | Internal vulnerability scanning for the private VMware lab. |

## New VM 1 — OPNsense Firewall

### VMware Settings

| Setting | Recommendation |
|---|---|
| VM name | `OPNsense-Firewall` |
| Guest OS type | FreeBSD 13/14 64-bit, or closest FreeBSD 64-bit option |
| CPU | 2 vCPU |
| RAM | 2 GB |
| Disk | 20 GB, thin provisioned |
| CD/DVD | OPNsense `dvd` ISO |
| Network Adapter 1 | NAT — this becomes OPNsense WAN |
| Network Adapter 2 | Custom host-only lab network, recommended `VMnet2` — this becomes OPNsense LAN |

### Recommended OPNsense Lab LAN Plan

| Setting | Value |
|---|---|
| LAN IP | `10.10.10.1/24` |
| DHCP range | `10.10.10.100`–`10.10.10.199` |
| DNS for LAN clients | OPNsense default or upstream DNS through OPNsense |
| WAN | DHCP from VMware NAT |

### VMware Network Recommendation

Create a dedicated host-only network if possible:

```text
VMnet2 = Host-only / isolated lab LAN
VMware DHCP on VMnet2 = Disabled if possible
OPNsense DHCP = Enabled on LAN
```

Why disable VMware DHCP?

> OPNsense should be the firewall/router and DHCP server for the protected lab LAN. Two DHCP servers on the same lab LAN can cause confusing IP assignments.

If VMware Player does not let you create/customize VMnet2, use the available Host-only network and tell Brainiac before proceeding so the plan can be adjusted.

## New VM 2 — Vulnerable Linux Target

### VMware Settings

| Setting | Recommendation |
|---|---|
| VM name | `Vuln-Ubuntu-Target` |
| Guest OS | Ubuntu Server LTS, or Debian amd64 |
| CPU | 1–2 vCPU |
| RAM | 2 GB preferred; 1 GB acceptable if constrained |
| Disk | 20 GB, thin provisioned |
| CD/DVD | Ubuntu Server LTS ISO or Debian netinst ISO |
| Network Adapter | Custom host-only lab network / `VMnet2` only |

### Install Choices

Recommended Ubuntu Server install choices:

- Minimal/server install is fine.
- Install OpenSSH Server if you want easier administration from Kali.
- Do not enable paid Ubuntu Pro.
- Do not expose this VM to the internet directly.
- Put it behind OPNsense on the lab LAN.

### Purpose

This VM becomes the authorized scan/remediation target for:

- Greenbone/OpenVAS Community Edition for the completed internal vulnerability scan workflow
- Greenbone/OpenVAS fallback if needed
- Nmap validation
- Firewall log generation
- Suricata/IDS telemetry
- Before/after remediation proof

## Existing VM Network Changes Later

Do not change existing VMs immediately. After OPNsense is installed and verified, move one VM at a time from NAT to the OPNsense LAN network.

Planned later state:

| VM | Current Network | Later Network |
|---|---|---|
| `Ubuntu` / Kali | NAT | `VMnet2` behind OPNsense |
| `Windows 11 x64` | NAT | `VMnet2` behind OPNsense |
| `Ubuntu Server` / SOC | NAT | `VMnet2` behind OPNsense |
| `Vuln-Ubuntu-Target` | New | `VMnet2` behind OPNsense |

## Safe Build Order

1. Create custom lab network `VMnet2` if available.
2. Create OPNsense VM with NAT + VMnet2 adapters.
3. Install OPNsense.
4. Verify OPNsense WAN gets internet through VMware NAT.
5. Configure/verify OPNsense LAN as `10.10.10.1/24`.
6. Create Vulnerable Linux Target on VMnet2.
7. Verify it receives an OPNsense DHCP lease.
8. Only then move Kali/Windows/SOC behind OPNsense one at a time.

## Screenshot Markers

| Filename | Capture |
|---|---|
| `01-opnsense-dashboard.png` | OPNsense dashboard after first successful login; operator chose this as screenshot 01 |
| `05-target-address-route-dns-baseline.png` | Vulnerable Linux target VM settings |
| `07-opnsense-dhcp-leases.png` | Vulnerable target showing DHCP/IP details from the lab LAN |
| `03-vmware-vmnet2-network-editor.png` | VM list or diagram showing five-VM design |
