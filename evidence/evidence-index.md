# Evidence Index

This folder stores sanitized validation notes, screenshots, exported reports, and command outputs used as proof for the lab.

## Screenshot and Artifact Evidence

| Evidence ID | Artifact | Status | Notes |
|---|---|---|---|
| EVID-001 | `screenshots/01-opnsense-dashboard.png` | Captured | OPNsense dashboard baseline showing system health, LAN/WAN interfaces, gateway status, and firewall activity. |
| EVID-002 | `screenshots/02-opnsense-configuration-wizard.png` | Captured | Initial OPNsense configuration wizard access after installation. |
| EVID-003 | `screenshots/03-vmware-vmnet2-network-editor.png` | Captured | VMware virtual network editor showing VMnet2 as the isolated host-only lab network `10.10.10.0/24`. |
| EVID-004 | `screenshots/04-opnsense-console-lan-wan-addresses.png` | Captured | OPNsense console showing LAN `10.10.10.254/24`, WAN on VMware NAT, and HTTPS management URL. |
| EVID-005 | `screenshots/05-target-address-route-dns-baseline.png` | Captured | Vulnerable Ubuntu target baseline address, route, and DNS configuration through OPNsense. |
| EVID-006 | `screenshots/06-target-internet-dns-baseline.png` | Captured | Target baseline internet and DNS validation using `ping`, `nslookup`, and `resolvectl query`. |
| EVID-007 | `screenshots/07-opnsense-dhcp-leases.png` | Captured | Dnsmasq DHCP lease assigning `ubuntutarget` to `10.10.10.167`. |
| EVID-008 | `screenshots/08-opnsense-firewall-rules-baseline.png` | Captured | Baseline OPNsense LAN firewall rules before custom policy changes. |
| EVID-009 | `screenshots/09-opnsense-firewall-live-log-baseline.png` | Captured | OPNsense live firewall log baseline proving traffic visibility and default rule behavior. |
| EVID-010 | `screenshots/10-target-after-snapshot-validation.png` | Captured | Post-snapshot target validation proving gateway reachability, internet connectivity, and DNS resolution. |
| EVID-011 | `screenshots/11-fw-cr-002-rule-created.png` | Captured | FW-CR-002 deny rule blocking `ubuntutarget` ICMP traffic to `8.8.8.8`. |
| EVID-012 | `screenshots/12-fw-cr-002-validation-blocked-icmp.png` | Captured | Target-side validation showing ICMP to `8.8.8.8` blocked while gateway and DNS continue working. |
| EVID-013 | `screenshots/13-fw-cr-002-block-log-evidence.png` | Captured | OPNsense live log showing ICMP blocks from `10.10.10.167` to `8.8.8.8` under FW-CR-002. |
| EVID-014 | `screenshots/14-fw-cr-003-dhcp-reservation-created.png` | Captured | Dnsmasq host reservation/static mapping for `ubuntutarget` at `10.10.10.167`. |
| EVID-015 | `screenshots/15a-fw-cr-003-target-address-route-dns.png` | Captured | Target still uses `10.10.10.167`, default route `10.10.10.254`, and DNS server `10.10.10.254` after DHCP reservation. |
| EVID-016 | `screenshots/15b-fw-cr-003-target-dns-query.png` | Captured | Target DNS query after DHCP reservation proves name resolution remains functional. |
| EVID-017 | `screenshots/16-greenbone-scanner-network-validation.png` | Captured | Greenbone/OpenVAS scanner VM on `10.10.10.100/24` with gateway/DNS via OPNsense and reachability to `10.10.10.167`. |
| EVID-018 | `screenshots/16a-greenbone-install-network-config.png` | Captured | Ubuntu installer network configuration for the Greenbone scanner VM on VMnet2. |
| EVID-019 | `screenshots/17-greenbone-platform-login-page.png` | Captured | OpenVAS Community Edition login page reached successfully; no credentials shown. |
| EVID-020 | `screenshots/18-greenbone-platform-dashboard.png` | Captured | Authenticated OpenVAS dashboard showing vulnerability intelligence/NVT data loaded. |
| EVID-021 | `screenshots/19-greenbone-target-scope.png` | Captured | Authorized Greenbone scan scope for `Vuln-Ubuntu-Target` / `10.10.10.167` only. |
| EVID-022 | `screenshots/20-greenbone-baseline-scan-task-created.png` | Captured | Baseline scan task created for the authorized target using the `Full and fast` scan configuration. |
| EVID-023 | `screenshots/20a-greenbone-baseline-scan-task-config.png` | Captured | Supplemental Greenbone task configuration evidence for target/scanner/scan profile. |
| EVID-024 | `screenshots/21-greenbone-scan-running.png` | Captured | Baseline scan actively running against the single authorized lab target. |
| EVID-025 | `screenshots/22-greenbone-findings-summary.png` | Captured | Initial Greenbone results summary showing low/log findings from the authorized target. |
| EVID-026 | `screenshots/22b-greenbone-updated-results-14-summary.png` | Captured | Updated baseline Greenbone results showing `14` findings total, including `2` medium SSH findings. |
| EVID-027 | `screenshots/23-remediation-tracker.png` | Captured | CSV remediation tracker populated with the two medium Greenbone SSH findings. |
| EVID-028 | `screenshots/24a-ssh-before-remediation-proof.png` | Captured | Before-remediation SSH proof showing OpenSSH version and effective algorithms prior to hardening. |
| EVID-029 | `screenshots/24-validation-scan-or-remediation-proof.png` | Captured | SSH remediation validation: syntax passed, AES-GCM ciphers enforced, SSH active, and port `22` listening. |
| EVID-030 | `screenshots/24b-greenbone-validation-scan-cleared-terrapin.png` | Captured | Follow-up Greenbone validation showing no medium-severity findings remaining; low/log findings remain. |
| EVID-031 | `screenshots/25-greenbone-vulnerability-management-summary.png` | Captured | Portfolio management summary showing target, `2` initial medium findings, `0` validated medium findings, and closed lifecycle status. |
| EVID-032 | `screenshots/26-wazuh-services-and-ports.png` | Captured | Wazuh SOC server service and listener validation for dashboard, agent intake, enrollment, and API ports. |
| EVID-033 | `screenshots/27-wazuh-dashboard-agent-status.png` | Captured | Wazuh endpoint dashboard showing active Windows 11 agent `WIN11-ENDPOINT01`. |
| EVID-034 | `screenshots/28-windows-sysmon-service-and-events.png` | Captured | Windows endpoint Sysmon64 service running with recent Sysmon Event ID `1` and `11` telemetry. |
| EVID-035 | `screenshots/29-benign-powershell-telemetry-generation.png` | Captured | Benign PowerShell/file-creation telemetry generated on the Windows endpoint. |
| EVID-036 | `screenshots/30-wazuh-threat-hunt-powershell-results.png` | Captured | Wazuh Threat Hunting results showing `57` hits, including rule `92027` and rule `92213`. |
| EVID-037 | `screenshots/31-wazuh-powershell-event-details.png` | Captured | Expanded Wazuh document details for the Sysmon Event ID `1` PowerShell process creation event. |
| EVID-038 | `screenshots/31a-wazuh-powershell-rule-details.png` | Captured | Supplemental Wazuh rule-details view for rule `92027` explaining the detection logic. |
| EVID-039 | `threat-hunting/HUNT-001-powershell-process-spawn.md` | Captured | Hunt report for benign PowerShell process-spawn telemetry mapped to Sysmon Event ID `1`, Wazuh rule `92027`, and MITRE ATT&CK `T1059.001`. |
| EVID-040 | `vulnerability-management/greenbone-management-summary.md` | Captured | Written management summary for Greenbone vulnerability discovery, remediation, validation, and closure. |
| EVID-041 | `dashboards/greenbone-management-summary.html` | Captured | Static portfolio dashboard summarizing the Greenbone vulnerability-management lifecycle. |
| EVID-042 | `threat-hunting/soc-threat-hunting-summary.md` | Captured | Management-level SOC threat-hunting summary for the Wazuh/Sysmon PowerShell hunt lifecycle. |
| EVID-043 | `dashboards/soc-threat-hunting-summary.html` | Captured | Static portfolio dashboard summarizing SOC telemetry, Wazuh hunt results, and analyst conclusion. |
| EVID-044 | `screenshots/32-soc-threat-hunting-summary.png` | Captured | Visual SOC threat-hunting summary dashboard screenshot for portfolio presentation. |

## Supplemental Evidence

| Artifact | Reason |
|---|---|
| `evidence/supplemental/23-greenbone-remediation-tickets-empty.png` | Shows Greenbone internal remediation tickets were empty, supporting the decision to use the repo-local CSV remediation tracker for portfolio workflow evidence. |
