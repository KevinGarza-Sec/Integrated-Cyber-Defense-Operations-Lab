# 03 — Build Phases

## Phase 1 — Planning and Architecture

- [x] Confirm VM resources and IP plan.
- [x] Create repo scaffold.
- [x] Define scenario, scope, and screenshot checklist.
- [x] Document evidence index and screenshot proof chain.

## Phase 2 — OPNsense Firewall Management

- [x] Deploy OPNsense VM.
- [x] Configure WAN/LAN interfaces.
- [x] Validate VMnet2 protected lab LAN.
- [x] Verify Dnsmasq DHCP lease for the vulnerable target.
- [x] Capture baseline dashboard, firewall rules, and live log evidence.
- [x] Document rollback/snapshot control in FW-CR-001.
- [x] Implement and validate a narrow firewall block rule in FW-CR-002.
- [x] Create and validate a stable DHCP reservation in FW-CR-003.
- [x] Document change requests and rule review.
- [ ] Optional: enable Suricata IDS/IPS later after the firewall baseline is stable.

## Phase 3 — Wazuh/Sysmon Threat Hunting

- [x] Verify Ubuntu SOC server and Wazuh services.
- [x] Verify Wazuh dashboard/indexer/API/agent ports.
- [x] Enroll Windows endpoint in Wazuh.
- [x] Install/verify Sysmon.
- [x] Generate benign PowerShell and file-creation telemetry.
- [x] Capture Wazuh Threat Hunting results and expanded event details.
- [x] Write first hunt report and MITRE mapping for PowerShell telemetry.
- [ ] Optional next hunts: failed logons, local user/group changes, Nmap scans.

## Phase 4 — Greenbone/OpenVAS Vulnerability Management

- [x] Deploy Greenbone/OpenVAS scanner on the protected lab LAN.
- [x] Verify scanner network route/DNS and target reachability.
- [x] Access OpenVAS Community Edition dashboard.
- [x] Define authorized single-host target scope.
- [x] Create and run baseline scan.
- [x] Capture baseline results and identify medium SSH findings.
- [x] Build remediation tracker for the two medium findings.

## Phase 5 — Remediation and Validation

- [x] Capture SSH before-state evidence.
- [x] Apply SSH hardening to reduce Terrapin-related cipher exposure.
- [x] Validate SSH syntax, effective cipher list, service health, and port listener.
- [x] Run/check Greenbone follow-up validation results.
- [x] Close the tracked medium findings with transparent validation notes.

## Phase 6 — Executive Reporting and Repo Packaging

- [x] Create Greenbone management summary Markdown.
- [x] Create Greenbone static HTML summary dashboard.
- [x] Capture management summary screenshot.
- [x] Update evidence index and screenshot checklist.
- [x] Add first threat-hunting report.
- [x] Create SOC/threat-hunting management summary Markdown, HTML dashboard, and screenshot.
- [x] Complete private GitHub push after validation and approval.

## Optional Future Enhancements

- [ ] Enable Suricata IDS/IPS after the firewall baseline is stable.
- [ ] Add additional Wazuh hunts such as failed logons, local user/group changes, or Nmap scans.
- [ ] Perform final GitHub web UI review before making the private repository public.
