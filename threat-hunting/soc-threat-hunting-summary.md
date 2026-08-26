# SOC Threat Hunting Summary

## Executive Summary

The SOC telemetry phase validated that a Windows endpoint can generate Sysmon telemetry, forward it through the Wazuh agent, and expose the event in Wazuh Threat Hunting for analyst review. A benign PowerShell process-spawn test produced searchable Wazuh alerts and expanded event details suitable for a SOC analyst workflow.

## Scope

| Item | Value |
|---|---|
| SOC platform | Wazuh |
| Endpoint telemetry | Sysmon |
| Endpoint | `WIN11-ENDPOINT01` |
| Wazuh agent ID | `001` |
| Endpoint IP | `192.168.109.133` |
| Event source | `Microsoft-Windows-Sysmon/Operational` |
| Primary event ID | Sysmon Event ID `1` — Process Create |
| Wazuh rule | `92027` — PowerShell process spawned PowerShell instance |
| MITRE mapping | `T1059.001` — PowerShell |

## Test Procedure

A controlled PowerShell test generated benign process and file telemetry:

```powershell
New-Item -Path "$env:TEMP\soc-lab-telemetry-test.txt" -ItemType File -Force
powershell.exe -NoProfile -ExecutionPolicy Bypass -Command "Write-Output 'SOC lab benign PowerShell telemetry test'"
Get-Process powershell | Select-Object -First 5 Id, ProcessName, StartTime
```

## Validation Results

| Validation Item | Result |
|---|---|
| Wazuh server services | Dashboard/indexer/API/agent ports validated |
| Windows endpoint enrollment | `WIN11-ENDPOINT01` active in Wazuh |
| Sysmon service | `Sysmon64` running |
| Local telemetry | Sysmon Event ID `1` and Event ID `11` observed |
| Wazuh hunt results | `57` hits in the selected 30-minute window |
| Detection rule | Rule `92027` observed for nested PowerShell behavior |
| Event detail review | Expanded Wazuh document showed command line, image, parent image, agent, and Sysmon fields |

## Analyst Conclusion

The observed PowerShell activity was expected benign lab activity. In a real SOC, the same pattern would justify analyst review because nested PowerShell execution with `ExecutionPolicy Bypass` can appear in administrative workflows and adversary tradecraft. The lab demonstrates the ability to collect endpoint telemetry, search SIEM events, inspect raw event fields, and document a defensible analyst conclusion.

## Evidence

| Artifact | Purpose |
|---|---|
| `screenshots/26-wazuh-services-and-ports.png` | Wazuh service and listener validation |
| `screenshots/27-wazuh-dashboard-agent-status.png` | Active Windows endpoint agent proof |
| `screenshots/28-windows-sysmon-service-and-events.png` | Sysmon service and local event proof |
| `screenshots/29-benign-powershell-telemetry-generation.png` | Controlled benign telemetry generation |
| `screenshots/30-wazuh-threat-hunt-powershell-results.png` | Wazuh hunt results and alert counts |
| `screenshots/31-wazuh-powershell-event-details.png` | Expanded event details for the PowerShell process creation event |
| `screenshots/31a-wazuh-powershell-rule-details.png` | Supplemental Wazuh rule logic for rule `92027` |
| `threat-hunting/HUNT-001-powershell-process-spawn.md` | Detailed hunt report and MITRE mapping |

## Portfolio Value

This phase demonstrates a practical SOC workflow:

```text
Endpoint action → Sysmon telemetry → Wazuh ingestion → Threat Hunting query → event details → analyst conclusion
```
