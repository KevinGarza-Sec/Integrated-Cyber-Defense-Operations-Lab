# 02 — Architecture and VM Plan

## Recommended VM Layout

The cleanest version uses **five VMs**:

| VM | Role | Required? | Notes |
|---|---|---:|---|
| OPNsense | Firewall/router/IDS | Yes | New VM recommended |
| Ubuntu SOC Server | Wazuh manager/dashboard/indexer | Yes | Can reuse existing SOC server if healthy |
| Windows Endpoint | Wazuh agent + Sysmon telemetry | Yes | Can reuse existing Windows lab VM |
| Kali Linux | Analyst/testing workstation | Yes | Can reuse existing Kali VM |
| Vulnerable Linux Target | Scan/test target | Strongly recommended | New lightweight VM or container host |

## Minimum Practical Version

If resources are tight, use **four VMs**:

1. OPNsense
2. Ubuntu SOC Server
3. Windows Endpoint
4. Kali Linux

Then run vulnerable services in Docker on the Ubuntu SOC server or a separate container host. This saves resources but weakens segmentation realism.

## Best Portfolio Version

Use the five-VM layout. It gives the cleanest story:

```text
Internet/NAT
    |
[OPNsense]
    |
Lab LAN / monitored segment
    |------------------|------------------|------------------|
[Ubuntu SOC]       [Windows Endpoint]   [Vulnerable Linux]  [Kali Analyst]
 Wazuh              Sysmon/Wazuh Agent    Scan target         Test traffic
```

## Why an Extra VM Is Recommended

A separate vulnerable target gives you a safe place to:

- Run vulnerability scans
- Remediate findings
- Validate before/after improvement
- Generate firewall and IDS telemetry
- Avoid weakening your SOC server

## Candidate Vulnerable Target Options

| Option | Recommendation | Notes |
|---|---|---|
| Ubuntu Server with intentionally outdated/misconfigured services | Best professional choice | Safer and easier to explain than intentionally vulnerable distros |
| Metasploitable | Good learning target | Very vulnerable; isolate carefully |
| DVWA on Docker | Good web-app target | Useful for web findings and Wazuh/Suricata alerts |
| OWASP Juice Shop | Good modern web target | Good screenshots, but more appsec-oriented |

## Draft Network Design

Exact IPs will be confirmed during build. Do not assume final addresses until verified.

| Segment | Purpose | Example |
|---|---|---|
| WAN | NAT/outbound internet for updates | VMware NAT adapter |
| LAN | Lab systems behind OPNsense | RFC1918 lab subnet |
| Optional DMZ | Vulnerable target segment | Separate OPNsense interface if resources allow |

## Snapshot Points

Take snapshots/checkpoints:

1. Before installing OPNsense — completed
2. After basic OPNsense LAN/WAN access works — pending
3. Before installing Wazuh
4. After Wazuh dashboard and agent enrollment works
5. Before vulnerability remediation
6. After validation scan
