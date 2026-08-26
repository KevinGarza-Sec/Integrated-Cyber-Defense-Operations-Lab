# FW-CR-001 — Baseline Backup and Snapshot Before Firewall Policy Changes

## Change Summary

Create rollback points before modifying OPNsense firewall policy.

This change does **not** modify firewall rules. It captures a safe pre-change state so later firewall-management work can be performed with a clear rollback path.

## Business / Lab Justification

The lab has completed baseline firewall deployment and validation:

- OPNsense dashboard reachable.
- LAN/WAN interfaces validated.
- Dnsmasq DHCP lease observed for `ubuntutarget`.
- Vulnerable target routing and DNS validated.
- Baseline firewall rules and live logs captured.

Before implementing firewall policy changes, a professional workflow requires a known-good rollback point.

## Scope

| Item | Value |
|---|---|
| Firewall | OPNsense |
| Lab LAN | `10.10.10.0/24` |
| Firewall LAN | `10.10.10.254/24` |
| Vulnerable Target | `ubuntutarget` / `10.10.10.167` |
| Change Type | Backup / snapshot only |
| Production Impact | None — isolated VMware lab |

## Planned Actions

1. Take VMware snapshot of `OPNsense-Firewall`.
2. Take VMware snapshot of `Vuln-Ubuntu-Target`.
3. Export/download OPNsense configuration backup for private rollback use.
4. Store the config backup outside the public repo or in a gitignored private/raw-evidence location.
5. Verify the firewall and target still function after snapshots/backups.

## Rollback Plan

If later firewall policy changes break access or routing:

1. Revert `OPNsense-Firewall` to snapshot `baseline-opnsense-dashboard-dhcp-rules-logs`.
2. Revert `Vuln-Ubuntu-Target` to snapshot `baseline-target-opnsense-dhcp-routing-dns` if target state changed.
3. If needed, restore the private OPNsense XML config backup through the OPNsense UI.

## Validation Steps

After backup/snapshot:

```bash
ping -c 4 10.10.10.254
ping -c 4 8.8.8.8
resolvectl query google.com
```

Expected results:

- Gateway ping succeeds.
- Internet IP ping succeeds.
- DNS query succeeds.

Observed validation evidence:

```text
ping -c 4 10.10.10.254: 4 transmitted, 4 received, 0% packet loss
ping -c 4 8.8.8.8: 4 transmitted, 4 received, 0% packet loss
resolvectl query google.com: success; returned IPv4 and IPv6 records via link ens33
```

Interpretation:

- Target can still reach the OPNsense LAN gateway.
- Target can still route to the internet through OPNsense/VMware NAT.
- DNS resolution still works from the target.

## Evidence

| Evidence | Screenshot / Artifact |
|---|---|
| OPNsense dashboard baseline | `01-opnsense-dashboard.png` |
| DHCP lease evidence | `07-opnsense-dhcp-leases.png` |
| Firewall rules baseline | `08-opnsense-firewall-rules-baseline.png` |
| Firewall live log baseline | `09-opnsense-firewall-live-log-baseline.png` |
| VMware snapshots | Optional screenshot: `10-target-after-snapshot-validation.png` |
| OPNsense config backup | Private rollback artifact; do **not** commit raw XML if it contains secrets/certs |

## Security / Redaction Notes

- Do not publish raw OPNsense XML backups without review.
- OPNsense config backups may contain sensitive material such as hashed passwords, certificates, private keys, VPN material, API keys, or service secrets depending on installed features.
- Public screenshots may keep RFC1918 lab IPs visible.
- Public IPs and account identifiers should be redacted.
