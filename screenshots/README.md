# Screenshot Checklist

This checklist maps the portfolio evidence screenshots to the defensive workflow they prove.

| Filename | Capture | Why It Matters |
|---|---|---|
| `01-opnsense-dashboard.png` | OPNsense dashboard | Proves the firewall platform baseline, LAN/WAN interfaces, gateway status, and visible firewall activity. |
| `02-opnsense-configuration-wizard.png` | OPNsense configuration wizard | Shows the initial firewall configuration path after installation. |
| `03-vmware-vmnet2-network-editor.png` | VMware Virtual Network Editor | Documents the isolated host-only VMnet2 lab network used for protected lab traffic. |
| `04-opnsense-console-lan-wan-addresses.png` | OPNsense console | Proves LAN `10.10.10.254/24`, WAN DHCP, and the HTTPS management URL. |
| `05-target-address-route-dns-baseline.png` | Target IP/route/DNS baseline | Shows the vulnerable target receives `10.10.10.167/24`, routes through OPNsense, and uses OPNsense DNS. |
| `06-target-internet-dns-baseline.png` | Target internet/DNS baseline | Proves internet reachability and DNS resolution before firewall controls are tested. |
| `07-opnsense-dhcp-leases.png` | Dnsmasq DHCP lease | Proves OPNsense DHCP assigned `ubuntutarget` to `10.10.10.167`. |
| `08-opnsense-firewall-rules-baseline.png` | Baseline firewall rules | Proves the starting LAN rule state before controlled changes. |
| `09-opnsense-firewall-live-log-baseline.png` | Baseline firewall live logs | Proves the firewall is producing usable operational logs. |
| `10-target-after-snapshot-validation.png` | Post-snapshot target validation | Proves the lab returned to a known-good baseline before change testing. |
| `11-fw-cr-002-rule-created.png` | ICMP block rule | Shows FW-CR-002 was implemented to block target ICMP traffic to `8.8.8.8`. |
| `12-fw-cr-002-validation-blocked-icmp.png` | Target-side block validation | Proves the ICMP block works while gateway and DNS remain operational. |
| `13-fw-cr-002-block-log-evidence.png` | OPNsense block logs | Proves the firewall logged the denied ICMP traffic from the target. |
| `14-fw-cr-003-dhcp-reservation-created.png` | DHCP reservation/static mapping | Proves stable target addressing for repeatable scans and firewall rules. |
| `15a-fw-cr-003-target-address-route-dns.png` | Target route/DNS after reservation | Proves the reservation did not break routing or DNS. |
| `15b-fw-cr-003-target-dns-query.png` | Target DNS query after reservation | Proves DNS resolution still works after the reservation change. |
| `16-greenbone-scanner-network-validation.png` | Greenbone scanner network validation | Shows the scanner at `10.10.10.100/24` can reach OPNsense and the target. |
| `16a-greenbone-install-network-config.png` | Greenbone VM installer network config | Documents scanner placement on VMnet2 during installation. |
| `17-greenbone-platform-login-page.png` | OpenVAS login page | Proves web access to the scanner platform without exposing credentials. |
| `18-greenbone-platform-dashboard.png` | OpenVAS dashboard | Proves the scanner application is reachable and populated with vulnerability data. |
| `19-greenbone-target-scope.png` | Greenbone target scope | Proves the authorized single-target scan boundary: `10.10.10.167`. |
| `20-greenbone-baseline-scan-task-created.png` | Greenbone task created | Proves the baseline scan task exists before execution. |
| `20a-greenbone-baseline-scan-task-config.png` | Greenbone task configuration | Supplemental proof of target, scanner, and scan profile configuration. |
| `21-greenbone-scan-running.png` | Scan running | Proves the baseline scan was started against the authorized target. |
| `22-greenbone-findings-summary.png` | Initial findings summary | Shows early Greenbone results for the target. |
| `22b-greenbone-updated-results-14-summary.png` | Updated findings summary | Shows `14` total findings including two medium SSH findings. |
| `23-remediation-tracker.png` | Remediation tracker | Proves findings were translated into a trackable remediation workflow. |
| `24a-ssh-before-remediation-proof.png` | SSH before-state proof | Shows OpenSSH version and algorithms before hardening. |
| `24-validation-scan-or-remediation-proof.png` | SSH remediation validation | Proves SSH syntax passed, AES-GCM ciphers were enforced, SSH stayed active, and port `22` remained listening. |
| `24b-greenbone-validation-scan-cleared-terrapin.png` | Follow-up Greenbone validation | Shows medium-severity findings cleared after remediation validation. |
| `25-greenbone-vulnerability-management-summary.png` | Greenbone management dashboard | Summarizes the vulnerability-management lifecycle for portfolio readers. |
| `26-wazuh-services-and-ports.png` | Wazuh services and ports | Proves the SOC server components and expected Wazuh ports are active. |
| `27-wazuh-dashboard-agent-status.png` | Wazuh agent status | Proves the Windows 11 endpoint is enrolled and active. |
| `28-windows-sysmon-service-and-events.png` | Sysmon service and events | Proves Sysmon64 is running and generating process/file telemetry. |
| `29-benign-powershell-telemetry-generation.png` | Benign telemetry generation | Shows the controlled PowerShell/file-creation activity used for the hunt. |
| `30-wazuh-threat-hunt-powershell-results.png` | Wazuh hunt results | Proves the endpoint telemetry reached Wazuh and was searchable in Threat Hunting. |
| `31-wazuh-powershell-event-details.png` | Wazuh event details | Proves the raw Sysmon Event ID `1` fields behind the PowerShell detection. |
| `31a-wazuh-powershell-rule-details.png` | Wazuh rule details | Supplemental proof explaining Wazuh rule `92027` detection logic. |
| `32-soc-threat-hunting-summary.png` | SOC threat-hunting management dashboard | Summarizes Wazuh/Sysmon telemetry, PowerShell hunt results, MITRE mapping, and analyst conclusion. |

## Evidence Chain

```text
Firewall foundation
→ target addressing and routing
→ controlled firewall change and log validation
→ Greenbone vulnerability discovery
→ remediation tracking
→ SSH hardening and validation
→ Wazuh/Sysmon endpoint telemetry
→ PowerShell hunt report
```
