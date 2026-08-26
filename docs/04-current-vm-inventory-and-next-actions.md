# 04 — Current VM Inventory and Next Actions

## Current Confirmed VMs

| VMware Display Name | Lab Role | RAM | vCPU | Disk | Network Role | Notes |
|---|---|---:|---:|---:|---|---|
| `Ubuntu` | Analyst/Kali-style Linux VM | 4 GB | 2 | 30 GB | Analyst/test workstation | Used for validation and lab operations. |
| `Windows 11 x64` | Windows endpoint | 4 GB | 2 | 65 GB | Wazuh/Sysmon endpoint | Enrolled in Wazuh as `WIN11-ENDPOINT01`, agent ID `001`, active. |
| `Ubuntu Server` | SOC server | 8 GB | 4 | 60 GB | Wazuh server | Wazuh dashboard/indexer/API/agent services validated. |
| `OPNsense-Firewall` | Lab firewall/router | 2 GB class | 2 class | 20 GB class | WAN on VMware NAT, LAN on VMnet2 | LAN `10.10.10.254/24`; WAN on `192.168.109.0/24`. |
| `Vuln-Ubuntu-Target` | Vulnerable/remediation target | 1–2 GB class | 1–2 class | 20 GB class | Protected VMnet2 LAN | Hostname `ubuntutarget`, DHCP reservation `10.10.10.167`. |
| `Greenbone_OpenVAS` | Vulnerability scanner | 4 GB class | 4 class | 80 GB class | Protected VMnet2 LAN | Scanner address `10.10.10.100`; scans authorized lab target only. |

## Validated Network Layout

```text
VMware NAT / WAN: 192.168.109.0/24
        |
        | OPNsense WAN
        |
OPNsense firewall/router
LAN: 10.10.10.254/24
        |
        | VMnet2 protected lab LAN: 10.10.10.0/24
        |
        +-- Vuln-Ubuntu-Target: 10.10.10.167
        +-- Greenbone_OpenVAS:   10.10.10.100
```

## Completed Major Milestones

- OPNsense deployed and verified with LAN/WAN interfaces.
- VMnet2 protected lab network documented.
- Dnsmasq DHCP lease and static reservation created for `ubuntutarget`.
- Controlled firewall change FW-CR-002 blocked target ICMP to `8.8.8.8` and produced firewall log evidence.
- Greenbone/OpenVAS scanner deployed on the lab LAN and validated.
- Greenbone scan lifecycle completed: target scope, scan task, results, remediation tracker, SSH hardening, follow-up validation.
- Wazuh/Sysmon telemetry lifecycle completed for one benign PowerShell hunt: services, active agent, Sysmon proof, generated telemetry, hunt results, event details, and hunt report.

## Current Next Actions

1. Optional: create a SOC/threat-hunting management summary dashboard to match the Greenbone dashboard style.
2. Optional: add a second hunt such as failed logons, local admin group changes, or Nmap network-service discovery.
3. Review screenshots and docs for final public-repo polish.
4. Run final validation checks before any local commit.
5. Do not push to GitHub until explicitly approved.
