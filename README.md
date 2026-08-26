# Integrated Cyber Defense Operations Lab

## Firewall Management + Threat Hunting + Vulnerability Management

This portfolio lab demonstrates an end-to-end defensive security workflow using free/no-cost resources: **OPNsense** for firewall management, **Wazuh + Sysmon** for endpoint telemetry and threat hunting, and **Greenbone/OpenVAS Community Edition** for internal vulnerability management.

The project simulates a small enterprise environment where an analyst manages firewall policy, detects suspicious endpoint activity, identifies vulnerable assets, prioritizes remediation, validates risk reduction, and packages the work as evidence-backed operational documentation.

## Core Story

```text
Network and firewall foundation
        ↓
Controlled firewall change and log validation
        ↓
Internal vulnerability discovery with Greenbone/OpenVAS
        ↓
Risk-based remediation tracking
        ↓
SSH hardening and validation scan
        ↓
Wazuh/Sysmon threat hunting
        ↓
Analyst report and management-ready summary
```

## Implemented Tool Stack

| Domain | Primary Tool | Purpose | Cost Note |
|---|---|---|---|
| Firewall management | OPNsense | Firewall rules, routing, DHCP/DNS, live traffic logs | Free/open-source |
| Endpoint telemetry | Sysmon | Windows process/file telemetry for detection and hunting | Free |
| SIEM/threat hunting | Wazuh | Agent enrollment, alert search, rule details, event triage | Free/open-source |
| Vulnerability management | Greenbone/OpenVAS Community Edition | Internal vulnerability scanning and validation | Free/open-source |
| Analyst/testing | Ubuntu/Windows lab hosts | Benign test traffic, endpoint validation, remediation testing | Free/no-cost lab VMs |

## Claim Boundaries

- This is a simulated portfolio lab, not a production enterprise assessment.
- The Greenbone/OpenVAS scan was limited to a single authorized internal lab target.
- Private RFC1918 lab IPs are intentionally shown where they explain topology and evidence.
- The lab documents analyst process and defensive validation; it does not claim production incident response authority.

## Current Lab Inventory

Current reusable VMs:

- `Ubuntu` VMware display name — Kali/analyst-style Linux VM, 4 GB RAM, 2 vCPU, 30 GB disk
- `Windows 11 x64` — Windows endpoint with Wazuh Agent and Sysmon, 4 GB RAM, 2 vCPU, 65 GB disk
- `Ubuntu Server` — SOC server hosting Wazuh components, 8 GB RAM, 4 vCPU, 60 GB disk
- `OPNsense-Firewall` — firewall/router with WAN on VMware NAT and LAN on VMnet2
- `Vuln-Ubuntu-Target` — vulnerable/remediation target on the OPNsense LAN, reserved as `ubuntutarget` at `10.10.10.167`
- `Greenbone_OpenVAS` — internal vulnerability scanner on VMnet2 at `10.10.10.100`

## Completed Evidence Highlights

| Phase | Evidence |
|---|---|
| Network foundation | OPNsense dashboard, VMnet2, LAN/WAN addressing, DHCP lease, target route/DNS validation |
| Firewall operations | Baseline rules, live logs, controlled ICMP block rule, blocked traffic validation |
| Vulnerability management | Greenbone target scope, scan task, baseline results, remediation tracker, SSH hardening validation, follow-up scan |
| Threat hunting | Wazuh services, active Windows endpoint, Sysmon telemetry, benign PowerShell test, Wazuh event details, hunt report |

## Key Portfolio Artifacts

- `docs/PROJECT_SUMMARY.md`
- `SECURITY.md`
- `threat-hunting/soc-threat-hunting-summary.md`
- `dashboards/soc-threat-hunting-summary.html`
- `evidence/evidence-index.md`
- `screenshots/README.md`
- `evidence/network-foundation-validation.md`
- `firewall/change-requests/FW-CR-002-block-target-icmp-to-internet.md`
- `firewall/change-requests/FW-CR-003-dhcp-reservation-for-vulnerable-target.md`
- `vulnerability-management/remediation-tracker.csv`
- `vulnerability-management/qualys-attempt-and-greenbone-fallback.md`
- `vulnerability-management/greenbone-management-summary.md`
- `dashboards/greenbone-management-summary.html`
- `threat-hunting/HUNT-001-powershell-process-spawn.md`
- `threat-hunting/mitre-attack-mapping.md`

## High-Level Repository Structure

```text
docs/                         Scenario, architecture, methodology, executive summary
firewall/                     OPNsense rulebase, change requests, segmentation policy
threat-hunting/               Hunt writeups, MITRE mapping, hunting templates
vulnerability-management/     Findings, remediation tracker, validation notes
diagrams/                     Architecture and data-flow diagrams
configs/                      Sanitized configuration notes/examples
evidence/                     Validation reports and evidence index
screenshots/                  Screenshot checklist and sanitized proof images
dashboards/                   Final visual summaries
```
