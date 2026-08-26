# Segmentation Policy Draft

## Objective

Use OPNsense to separate and control traffic between lab systems while preserving required telemetry and management flows.

## Default Principle

Deny by default. Allow only documented traffic required for lab function.

## Example Required Flows

| Source | Destination | Port/Protocol | Purpose |
|---|---|---|---|
| Windows Endpoint | Wazuh Manager | TCP/1514, TCP/1515 | Agent event forwarding/enrollment |
| Admin/Kali | Wazuh Dashboard | TCP/443 | Analyst dashboard access |
| Lab hosts | DNS resolver | UDP/TCP 53 | Name resolution |
| Lab hosts | Internet | TCP/80,443 | Updates/downloads |
| Kali | Vulnerable Target | Selected test ports | Authorized testing only |
| Vuln-Ubuntu-Target | 8.8.8.8 | ICMP echo | Blocked in FW-CR-002 to demonstrate controlled outbound policy enforcement |

## Evidence

- Firewall rulebase screenshot
- Blocked traffic log screenshot
- Config export or sanitized rule table
