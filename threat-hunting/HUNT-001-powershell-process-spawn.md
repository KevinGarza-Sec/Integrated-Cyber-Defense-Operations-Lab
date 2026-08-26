# HUNT-001 — PowerShell Process Spawn from PowerShell

## Hunt ID

HUNT-001

## Hypothesis

If an endpoint user or process launches PowerShell from an existing PowerShell session, Wazuh/Sysmon should generate process-creation telemetry that a SOC analyst can hunt and inspect for command-line context.

This behavior is not automatically malicious. Administrators use PowerShell legitimately. The hunt objective is to prove endpoint telemetry visibility and analyst triage workflow using safe lab-generated activity.

## Data Sources

- Windows endpoint: `WIN11-ENDPOINT01`
- Wazuh Agent: `001`
- Wazuh Manager: `ubuntuserver`
- Windows telemetry source: Sysmon Operational log
- Wazuh index: `wazuh-alerts-4.x-2026.08.26`
- Sysmon Event ID: `1` — Process Create
- Wazuh rule: `92027` — PowerShell process spawned PowerShell instance

## MITRE ATT&CK Mapping

| Tactic | Technique | Reason |
|---|---|---|
| Execution | `T1059` — Command and Scripting Interpreter | The activity uses a command interpreter to execute a command. |
| Execution | `T1059.001` — PowerShell | The observed process and command line are PowerShell-specific. |

## Query / Search Method

In Wazuh Threat Hunting, filter to the Windows endpoint and review events around the benign test timestamp.

Useful search/filter terms:

```text
agent.name: WIN11-ENDPOINT01
powershell.exe
ExecutionPolicy Bypass
rule.id: 92027
```

The captured Wazuh view used endpoint filtering and showed 57 events during the selected 30-minute window, including rule `92027` for the PowerShell process-spawn behavior.

## Expected Benign Test Activity

The Windows endpoint generated controlled activity from an elevated PowerShell session:

```powershell
New-Item -Path "$env:TEMP\soc-lab-telemetry-test.txt" -ItemType File -Force
powershell.exe -NoProfile -ExecutionPolicy Bypass -Command "Write-Output 'SOC lab benign PowerShell telemetry test'"
Get-Process powershell | Select-Object -First 5 Id, ProcessName, StartTime
```

Expected telemetry:

| Activity | Expected event |
|---|---|
| Temp file creation | Sysmon Event ID `11` — File Created |
| Nested PowerShell launch | Sysmon Event ID `1` — Process Create |
| Wazuh process-spawn detection | Wazuh rule `92027` |

## Findings

Wazuh Threat Hunting displayed events for `WIN11-ENDPOINT01` during the test window. The visible results included:

| Rule ID | Rule Level | Description |
|---:|---:|---|
| `92027` | `4` | PowerShell process spawned PowerShell instance |
| `92213` | `15` | Executable file dropped in folder commonly used by malware |

The expanded document details for the PowerShell event showed:

| Field | Observed value |
|---|---|
| `agent.name` | `WIN11-ENDPOINT01` |
| `agent.ip` | `192.168.109.133` |
| `data.win.system.channel` | `Microsoft-Windows-Sysmon/Operational` |
| `data.win.system.eventID` | `1` |
| `data.win.eventdata.image` | `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe` |
| `data.win.eventdata.parentImage` | `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe` |
| `data.win.eventdata.commandLine` | PowerShell launched with `-NoProfile -ExecutionPolicy Bypass` and the benign lab output string |

The public evidence screenshot is sanitized to preserve the useful detection fields while avoiding unnecessary local identity details.

## Analyst Conclusion

The event is expected benign lab activity because it directly matches the operator-generated PowerShell telemetry test. However, the detection is operationally useful: in a real environment, nested PowerShell execution with `ExecutionPolicy Bypass` would deserve review of the command line, parent process, user context, host role, and surrounding events.

This hunt proves the SOC telemetry chain:

```text
Windows endpoint action → Sysmon event generation → Wazuh ingestion → Threat Hunting search → expanded event triage
```

## Screenshot Evidence

| Screenshot | Purpose |
|---|---|
| `screenshots/28-windows-sysmon-service-and-events.png` | Sysmon64 running and recent Sysmon Operational events available |
| `screenshots/29-benign-powershell-telemetry-generation.png` | Benign PowerShell/file activity generated on Windows endpoint |
| `screenshots/30-wazuh-threat-hunt-powershell-results.png` | Wazuh Threat Hunting results show PowerShell-related alerts for the endpoint |
| `screenshots/31a-wazuh-powershell-rule-details.png` | Supplemental Wazuh rule logic for rule `92027` |
| `screenshots/31-wazuh-powershell-event-details.png` | Expanded event details for the specific PowerShell process creation event |
