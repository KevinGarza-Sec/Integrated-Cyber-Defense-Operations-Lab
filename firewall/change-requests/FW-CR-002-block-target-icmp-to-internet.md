# FW-CR-002 — Block Vulnerable Target ICMP to Internet

## Change Summary

Create a controlled firewall rule that blocks ICMP echo traffic from the vulnerable Ubuntu target to a known public test IP while leaving normal DNS/web connectivity intact.

This is the first active firewall-management demonstration after the clean baseline.

## Business / Lab Justification

The baseline LAN rule currently allows LAN clients to reach any destination. This change demonstrates that OPNsense can enforce a documented outbound policy and generate log evidence for blocked traffic.

The change is intentionally narrow and low-risk:

- It only affects one lab host: `ubuntutarget`.
- It only affects ICMP to one test destination: `8.8.8.8`.
- It does not block DNS, web browsing, updates, SSH, or access to OPNsense.
- It is easy to validate and roll back.

## Scope

| Field | Value |
|---|---|
| Change ID | FW-CR-002 |
| Firewall | OPNsense |
| Interface | LAN |
| Source | `ubuntutarget` / `10.10.10.167` |
| Destination | `8.8.8.8` |
| Service/Port | ICMP / Echo Request |
| Action | Block |
| Logging | Enabled |
| Rule Placement | Above default `allow LAN to any` rule |
| Production Impact | None — isolated VMware lab |

## Pre-Change Baseline

Prior evidence shows ICMP worked before the change:

```text
ping -c 4 8.8.8.8: 4 transmitted, 4 received, 0% packet loss
```

Baseline screenshot:

```text
10-target-after-snapshot-validation.png
```

## Implementation Plan

In OPNsense:

1. Navigate to `Firewall > Rules > LAN`.
2. Add a new rule above the default allow rule.
3. Configure the rule:

| Setting | Value |
|---|---|
| Action | Block |
| Interface | LAN |
| Direction | In |
| TCP/IP Version | IPv4 |
| Protocol | ICMP |
| ICMP Type | Echo request if available, otherwise any ICMP |
| Source | Single host or alias: `10.10.10.167` |
| Destination | Single host or alias: `8.8.8.8` |
| Log packets matched by this rule | Enabled |
| Description | `FW-CR-002 block ubuntutarget ICMP to 8.8.8.8` |

4. Save the rule.
5. Apply changes.
6. Validate from the vulnerable target.

## Validation Steps

From `ubuntutarget`:

```bash
ping -c 4 8.8.8.8
ping -c 4 10.10.10.254
resolvectl query google.com
```

Observed validation results:

```text
ping -c 4 8.8.8.8: 4 transmitted, 0 received, 100% packet loss
ping -c 4 10.10.10.254: 4 transmitted, 4 received, 0% packet loss
resolvectl query google.com: success; returned IPv4 and IPv6 records via ens33
```

Interpretation:

- FW-CR-002 successfully blocks the intended ICMP echo traffic to `8.8.8.8`.
- Local gateway access to OPNsense remains functional.
- DNS resolution remains functional.
- The rule is narrow and did not break general network/DNS functionality.

## Rollback Plan

If the rule causes unintended behavior:

1. Disable or delete the FW-CR-002 rule.
2. Apply changes.
3. Re-test:

```bash
ping -c 4 8.8.8.8
```

Expected rollback result:

```text
4 transmitted, 4 received, 0% packet loss
```

If firewall access is disrupted unexpectedly, revert the VMware snapshot:

```text
baseline-opnsense-dashboard-dhcp-rules-logs
```

## Evidence to Capture

| Evidence | Filename |
|---|---|
| Rule created above default allow | `11-fw-cr-002-rule-created.png` |
| Target ping blocked to `8.8.8.8` while gateway/DNS still work | `12-fw-cr-002-validation-blocked-icmp.png` |
| OPNsense live log showing blocked ICMP | `13-fw-cr-002-block-log-evidence.png` |
| Optional rollback validation if tested | `12-fw-cr-002-validation-blocked-icmp.png` |

Observed log evidence:

```text
Firewall > Log Files > Live View
Filter: 10.10.10.167
LAN In ICMP 10.10.10.167 -> 8.8.8.8: block
Label: FW-CR-002 block ubuntutarget ICMP to 8.8.8.8
Observed timestamps: 2026-08-13T05:01:17 through 2026-08-13T05:01:20
Related DHCP allow entries: 10.10.10.167:68 -> 10.10.10.254:67 pass
```

Interpretation:

- OPNsense logged four blocked ICMP attempts matching the four ping requests.
- The block matched the intended source, destination, protocol, interface, and rule label.
- DHCP-related traffic remained allowed, confirming the rule did not broadly disrupt LAN services.

## Redaction Notes

- `10.10.10.167`, `10.10.10.254`, and `8.8.8.8` are safe to show.
- `8.8.8.8` is a public Google DNS test IP, not KG's public IP.
- Redact browser profile, personal bookmarks, account identifiers, and any unrelated public IPs if they appear in logs.
- No passwords or OPNsense config XML should be committed.
