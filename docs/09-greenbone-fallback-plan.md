# 09 — Qualys Attempt and Greenbone/OpenVAS Fallback Plan

## Why This Fallback Was Selected

Qualys Community Edition was reviewed first because it is recognizable vulnerability-management branding. The visible Qualys signup path presented a **full trial of the Enterprise TruRisk Platform** requiring a **work email**, and did not provide a clear personal-email Community Edition path.

Because this lab is no-cost and should avoid paid-trial ambiguity, the safe decision is:

```text
Do not proceed with the Enterprise trial form.
Pivot to Greenbone Community Edition / OpenVAS for internal VMware scanning.
```

This keeps the lab honest and safe while still demonstrating the full vulnerability-management lifecycle.

## Recommended Scanner Placement

Preferred clean design:

```text
Greenbone-Scanner VM on VMnet2 / OPNsense LAN
```

Network placement:

```text
VMware NAT / Internet
        |
     OPNsense
 WAN: 192.168.109.135/24
 LAN: 10.10.10.254/24
        |
      VMnet2
        |
        +-- Vuln-Ubuntu-Target: 10.10.10.167
        +-- Greenbone-Scanner: 10.10.10.x
```

Why a scanner VM instead of OPNsense or the target:

- Keeps OPNsense stable as the firewall/router.
- Keeps the target as the asset being assessed, not the assessment platform.
- Avoids overloading the future Wazuh/SOC server.
- Gives clean technical evidence: scanner, target, firewall, logs, remediation.

## Suggested Greenbone Scanner VM Settings

| Setting | Recommendation |
|---|---|
| VM name | `Greenbone-Scanner` |
| OS | Ubuntu Server LTS or Debian amd64 |
| CPU | 2 vCPU minimum; 4 vCPU preferred if the host can spare it |
| RAM | 4 GB minimum; 6–8 GB preferred for smoother feed sync/scans |
| Disk | 40–60 GB thin provisioned |
| NIC | VMnet2 / OPNsense LAN |
| Gateway | `10.10.10.254` |
| DNS | `10.10.10.254` |

Validated build choice for KG's lab:

| Setting | Value |
|---|---|
| VM name | `Greenbone_OpenVAS` |
| CPU | 4 vCPU |
| RAM | 4 GB |
| Disk | 40 GB |
| NIC | VMnet2 / OPNsense LAN |
| DHCP address observed during install | `10.10.10.100/24` |
| Runtime status | VM created, booted, and post-install network validation passed |
| Rollback snapshot | `16-greenbone-scanner-network-validation` taken before Greenbone install |

Post-install validation observed:

```text
Interface: ens33
Scanner IP: 10.10.10.100/24
Default gateway: 10.10.10.254
DNS server: 10.10.10.254
Ping OPNsense gateway 10.10.10.254: 4 transmitted, 4 received, 0% packet loss
Ping ubuntutarget 10.10.10.167: 4 transmitted, 4 received, 0% packet loss
DNS query google.com: success
```

Redaction note: the screenshot should redact the MAC-derived IPv6 link-local address before public release.

If resources are tight, use the existing Kali VM for lighter Nmap/manual vulnerability discovery temporarily, but the stronger technical version is a dedicated Greenbone scanner VM.

## First Greenbone Evidence Set

Continue the vulnerability-management screenshot sequence:

| Filename | Capture | Purpose | Redact |
|---|---|---|---|
| `16-greenbone-scanner-network-validation.png` | Greenbone scanner installer/network validation showing `10.10.10.100/24` on VMnet2 | Proves scanner is on the correct lab LAN | MAC address, MAC-derived identifiers |
| `17-greenbone-platform-login-page.png` | OpenVAS login page through the SSH tunnel | Proves secure administrative reachability | Never capture credentials |
| `18-greenbone-platform-dashboard.png` | Authenticated dashboard with populated CVE/NVT data | Proves platform readiness | Logged-in account name (`admin`) |
| `19-greenbone-target-scope.png` | Greenbone target config scoped only to `10.10.10.167` | Proves authorized scope | Credentials, scanner IDs if visible |
| `22b-greenbone-updated-results-14-summary.png` | First findings summary | Proves vulnerability discovery | MACs, unrelated public IPs |
| `23-remediation-tracker.png` | Remediation tracker populated with findings | Proves risk triage | Real owner names/ticket IDs |
| `24-validation-scan-or-remediation-proof.png` | Re-scan/manual validation after remediation | Proves closure loop | Same as findings |

## Guardrails

- Do not scan systems outside the VMware lab.
- Do not scan public IPs/domains KG does not own.
- Do not store Greenbone admin passwords in the repo.
- Change any default `admin/admin` credentials immediately if Greenbone creates them.
- Redact credentials, tokens, cookies, scanner IDs, and public IPs.


> I initially evaluated Qualys Community Edition, but the available signup path presented an Enterprise trial requiring a work email rather than a clear no-cost personal Community Edition path. To avoid billing or trial ambiguity, I pivoted to Greenbone/OpenVAS inside my private VMware lab. This kept scanning authorized, internal, and no-cost while still proving vulnerability discovery, prioritization, remediation tracking, and validation.
