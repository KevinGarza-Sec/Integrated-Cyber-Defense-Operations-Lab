# 07 — Dedicated VMnet2 + OPNsense DHCP Plan

## Decision

Use the cleaner technical design: create a dedicated VMware host-only lab network and let OPNsense own routing, DNS forwarding, and DHCP for lab clients.

## Why

The vulnerable target initially received DHCP from VMware host-only networking:

```text
Target IP: 192.168.153.130/24
Default gateway: 192.168.153.1
DNS: 192.168.153.1
OPNsense LAN: 192.168.153.128
```

That allowed the target to reach OPNsense, but it did not route traffic through OPNsense as the default gateway. For firewall management, threat hunting, and vulnerability management evidence, lab systems should route through the firewall.

## Target Design

```text
VMware NAT / VMnet8 / 192.168.109.0/24
        |
        | OPNsense WAN
        |
 [OPNsense-Firewall]
        |
        | OPNsense LAN: 10.10.10.254/24
        |
VMnet2 host-only lab LAN: 10.10.10.0/24
        |
        +-- Vuln-Ubuntu-Target: DHCP 10.10.10.100-199
        +-- Later Kali / Windows / SOC as needed
```

## Planned Addressing

| Component | Address/Setting |
|---|---|
| VMnet2 subnet | `10.10.10.0/24` |
| VMware DHCP on VMnet2 | **Disabled** — screenshot showed it was initially enabled; disable before moving lab clients |
| Host VMnet2 adapter | likely `10.10.10.1/24` |
| OPNsense LAN | `10.10.10.254/24` |
| OPNsense DHCP range | `10.10.10.100`–`10.10.10.199` |
| Lab client gateway | `10.10.10.254` |
| Lab client DNS | `10.10.10.254` |
| OPNsense WAN | DHCP from VMware NAT / VMnet8 |

Why not use `10.10.10.1` for OPNsense LAN?

> VMware host-only adapters often claim `.1` for the host. Using `10.10.10.254` for OPNsense avoids an IP conflict while keeping the firewall address predictable.

## Safe Change Procedure

1. Take/snapshot OPNsense and Vuln-Ubuntu-Target first if possible.
2. Create `VMnet2` as host-only in VMware Virtual Network Editor.
3. Set VMnet2 subnet to `10.10.10.0/24`.
4. Disable VMware DHCP for VMnet2.
5. Power off `OPNsense-Firewall` and `Vuln-Ubuntu-Target`.
6. Set OPNsense adapter 1/WAN to NAT.
7. Set OPNsense adapter 2/LAN to Custom `VMnet2`.
8. Set Vuln-Ubuntu-Target adapter to Custom `VMnet2`.
9. Boot OPNsense.
10. From OPNsense console, set LAN IP to `10.10.10.254/24` and enable DHCP.
    - Important: when OPNsense asks `Configure IPv4 address LAN interface via DHCP?`, answer **No**. That question controls the firewall's LAN interface address. The LAN interface must be static.
    - Later, when it asks whether to enable the DHCP server on LAN for clients, answer **Yes** and use the planned range.
11. Boot Vuln-Ubuntu-Target.
12. Verify target receives `10.10.10.x`, gateway `10.10.10.254`, DNS `10.10.10.254`.

Current implemented state after console configuration:

```text
OPNsense LAN (em1): 10.10.10.254/24
OPNsense WAN (em0): DHCP4 192.168.109.135/24
Unbound DNS: started
Web GUI: started
Web GUI URL: https://10.10.10.254
LAN DHCP range: 10.10.10.100-10.10.10.199
```

## Verification Commands

On the vulnerable target:

```bash
ip addr
ip route
resolvectl status
ping -c 4 10.10.10.254
ping -c 4 8.8.8.8
```

Expected:

```text
IP: 10.10.10.100-199/24
Default gateway: 10.10.10.254
DNS: 10.10.10.254
Ping OPNsense LAN: success
Ping 8.8.8.8: success if OPNsense NAT/firewall is working
```

## Rollback

If VMnet2 does not work:

1. Power off the VMs.
2. Return OPNsense LAN adapter and target adapter to the previous host-only VMnet.
3. Restore the last working OPNsense/target snapshots if needed.
4. Keep Windows VM on NAT during this phase.

## Screenshot Markers

| Filename | Capture |
|---|---|
| `03-vmware-vmnet2-network-editor.png` | VMnet2 subnet with VMware DHCP disabled |
| `04-opnsense-console-lan-wan-addresses.png` | OPNsense LAN IP/DHCP range |
| `07-opnsense-dhcp-leases.png` | Target showing DHCP lease/gateway/DNS from OPNsense |
