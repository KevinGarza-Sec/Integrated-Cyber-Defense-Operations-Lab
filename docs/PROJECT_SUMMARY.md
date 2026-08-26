# Project Summary — Integrated Cyber Defense Operations Lab

## Repository Description

Portfolio cyber defense lab with OPNsense firewall operations, Wazuh/Sysmon threat hunting, and Greenbone/OpenVAS vulnerability management. Demonstrates scoped scanning, remediation tracking, SSH hardening, validation, MITRE mapping, and evidence-backed reporting.

## Executive Overview

This project demonstrates a complete defensive security operations workflow in a private VMware lab. The environment combines firewall administration, vulnerability management, endpoint telemetry, and threat hunting into one evidence-backed portfolio project.

The lab uses OPNsense to provide routing, DHCP/DNS, firewall rule enforcement, and live traffic logging. A vulnerable Ubuntu target is placed behind the firewall on a protected lab LAN. Greenbone/OpenVAS is used for internal vulnerability scanning after an attempted Qualys Community Edition workflow was limited by signup/access constraints. Wazuh and Sysmon provide endpoint telemetry and threat-hunting evidence from a Windows 11 endpoint.

## What This Lab Proves

| Capability | Evidence |
|---|---|
| Network segmentation and firewall operations | OPNsense LAN/WAN setup, VMnet2 lab LAN, DHCP lease, firewall baseline, live logs |
| Controlled firewall change management | FW-CR-002 ICMP block rule and FW-CR-003 DHCP reservation evidence |
| Vulnerability management lifecycle | Greenbone target scope, scan task, findings, remediation tracker, SSH hardening, validation scan |
| Endpoint telemetry and threat hunting | Wazuh services, active Windows endpoint, Sysmon Event ID 1/11 proof, PowerShell hunt report |
| Analyst reporting | Evidence index, screenshot checklist, MITRE mapping, management summary dashboards |

## Key Outcomes

- Built an isolated lab network behind OPNsense using VMnet2 and private RFC1918 addressing.
- Validated target routing, DNS, internet access, firewall rules, and firewall log visibility.
- Created a controlled firewall change request and verified both block behavior and logging.
- Used Greenbone/OpenVAS to scan a single authorized lab target.
- Tracked the two medium SSH findings in a remediation tracker.
- Hardened SSH cipher exposure and validated service health after change.
- Confirmed follow-up Greenbone results no longer showed medium-severity findings.
- Verified Wazuh services, Windows agent enrollment, Sysmon telemetry, and searchable Wazuh alerts.
- Documented a PowerShell process-spawn hunt mapped to MITRE ATT&CK `T1059.001`.

## Portfolio Positioning

This is a practical security operations project, not just a tool installation. It shows the operator can:

1. Define authorized scope.
2. Build a defensible lab topology.
3. Capture evidence before and after changes.
4. Translate scanner findings into remediation work.
5. Validate that remediation reduced risk without breaking service availability.
6. Generate endpoint telemetry and analyze it in a SIEM-style workflow.
7. Communicate results through clear documentation and management summaries.

## Primary Artifacts

- `README.md`
- `SECURITY.md`
- `evidence/evidence-index.md`
- `screenshots/README.md`
- `evidence/network-foundation-validation.md`
- `vulnerability-management/remediation-tracker.csv`
- `vulnerability-management/greenbone-management-summary.md`
- `dashboards/greenbone-management-summary.html`
- `threat-hunting/HUNT-001-powershell-process-spawn.md`
- `threat-hunting/soc-threat-hunting-summary.md`
- `dashboards/soc-threat-hunting-summary.html`
