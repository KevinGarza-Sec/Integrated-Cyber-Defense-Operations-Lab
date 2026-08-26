# MITRE ATT&CK Mapping

MITRE ATT&CK mapping in this lab describes behavioral similarity for analyst training. It does not prove malicious activity by itself.

| Hunt | Behavior | Technique | Evidence | Notes |
|---|---|---|---|---|
| HUNT-001 | PowerShell process spawned from PowerShell | `T1059.001` — PowerShell under `T1059` Command and Scripting Interpreter | `HUNT-001-powershell-process-spawn.md`, screenshots `29`–`31` | Benign lab command used to prove Sysmon Event ID `1` telemetry and Wazuh rule `92027`. |
| Future Hunt | Network service discovery | `T1046` — Network Service Discovery | Pending | Can be generated with an authorized Nmap scan inside the lab. |
| Future Hunt | Failed logons | `T1110` — Brute Force / password guessing behavior | Pending | Use controlled failed login attempts only against lab systems. |
| Future Hunt | Local account or admin group change | `T1098` — Account Manipulation | Pending | Use a lab-only test user and reverse the change after evidence capture. |

## Claim Boundary

The completed PowerShell hunt is intentionally benign. The value of the evidence is the telemetry and triage workflow:

```text
Controlled endpoint activity
→ Sysmon event generation
→ Wazuh ingestion
→ rule/event inspection
→ analyst conclusion
```
