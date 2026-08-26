# 01 — Scenario and Scope

## Scenario

A fictional small enterprise, **Fortress Regional Services**, needs to improve defensive operations across three areas:

1. Firewall policy and segmentation
2. Threat hunting and detection visibility
3. Vulnerability management and remediation tracking

As the security analyst, the objective is to build a defensible workflow that connects preventive controls, detection telemetry, vulnerability findings, and risk-based remediation.

## In Scope

- OPNsense firewall deployment and rule governance
- Lab network segmentation
- Firewall log review
- Wazuh/Sysmon endpoint telemetry
- Suricata IDS telemetry where feasible
- Greenbone/OpenVAS vulnerability management after an attempted Qualys Community Edition path proved unsuitable for this no-cost private lab workflow
- Greenbone/OpenVAS fallback for internal scanning if needed
- Remediation tracker and before/after validation

## Out of Scope

- Production firewall changes
- Scanning public IPs that are not owned/authorized
- Paid vulnerability management subscriptions
- Exploit development
- Unsafe malware execution
- Claims of production enterprise experience or formal compliance certification

## Safety Rules

- Only scan systems you own or explicitly control.
- Do not expose private lab hosts to the internet just to enable external scanning.
- Use benign test activity for detection proof.
- Snapshot VMs before major changes.
- Redact public IPs, account IDs, emails, tokens, keys, SIDs, and real usernames.
