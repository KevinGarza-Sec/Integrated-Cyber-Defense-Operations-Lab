# FW-CR-003 — DHCP Reservation for Vulnerable Target

## Change Summary

Convert the vulnerable Ubuntu target's dynamic DHCP lease into a stable DHCP reservation so firewall rules, vulnerability scans, remediation notes, and screenshots consistently refer to the same lab host IP.

## Business / Lab Justification

FW-CR-002 used a firewall rule scoped to the target IP:

```text
10.10.10.167
```

That IP is currently a dynamic DHCP lease. Dynamic addressing can change after lease expiration, VM cloning, adapter changes, or DHCP service reset. A stable reservation improves operational reliability and makes future evidence easier to interpret.

This is a low-risk network administration change because it preserves the same IP the host already uses.

## Scope

| Field | Value |
|---|---|
| Change ID | FW-CR-003 |
| Firewall | OPNsense |
| DHCP Backend | Dnsmasq DNS & DHCP |
| Target Hostname | `ubuntutarget` |
| Target Current IP | `10.10.10.167` |
| Target Interface | LAN |
| Reservation IP | `10.10.10.167` |
| Change Type | DHCP reservation / static mapping |
| Production Impact | None — isolated VMware lab |

## Pre-Change Evidence

Current active DHCP lease evidence:

```text
07-opnsense-dhcp-leases.png
```

Observed lease details:

```text
Interface: LAN
IP Address: 10.10.10.167
Hostname: ubuntutarget
Lease Type: dynamic
MAC Vendor: VMware, Inc.
```

## Implementation Plan

In OPNsense:

1. Go to `Services > Dnsmasq DNS & DHCP > Leases`.
2. Locate the `ubuntutarget` lease at `10.10.10.167`.
3. Use the page's add/static/reservation control if available, or note the MAC address from the lease.
4. Create a static DHCP reservation/mapping:

| Setting | Value |
|---|---|
| Interface | LAN |
| MAC address | Target VM MAC from lease page |
| IP address | `10.10.10.167` |
| Hostname | `ubuntutarget` |
| Description | `FW-CR-003 static DHCP reservation for vulnerable target` |

5. Save.
6. Apply changes.
7. Renew/reboot the target if needed.

## Validation Steps

From `ubuntutarget`:

```bash
ip -brief addr show ens33
ip route
resolvectl status
ping -c 4 10.10.10.254
resolvectl query google.com
```

Expected results:

```text
ens33 has 10.10.10.167/24
Default route remains via 10.10.10.254
DNS remains 10.10.10.254
Gateway ping succeeds
DNS query succeeds
```

Observed validation evidence:

```text
ip -brief addr show ens33: ens33 UP 10.10.10.167/24
ip route: default via 10.10.10.254 dev ens33 proto dhcp src 10.10.10.167
resolvectl status: DNS server 10.10.10.254; DefaultRoute=yes
ping -c 4 10.10.10.254: 4 transmitted, 4 received, 0% packet loss
resolvectl query google.com: success; returned IPv4 and IPv6 records via ens33
```

Interpretation:

- The vulnerable target retained the intended reserved IP address.
- The default gateway and DNS still point to OPNsense.
- Gateway reachability and DNS resolution still work after the reservation change.

In OPNsense:

1. Return to `Services > Dnsmasq DNS & DHCP > Leases`.
2. Confirm `ubuntutarget` is still associated with `10.10.10.167`.
3. If the UI distinguishes lease type, confirm it appears as static/reserved rather than only dynamic.

Observed OPNsense-side reservation evidence:

```text
Services > Dnsmasq DNS & DHCP > Hosts
Host: ubuntutarget
IP address: 10.10.10.167
Hardware address: VMware MAC for ubuntutarget
Description: FW-CR-003 static DHCP reservation for vulnerable target
Apply button: no pending orange Apply button visible in final capture
```

Interpretation:

- The Dnsmasq host override/static mapping exists for `ubuntutarget`.
- The final OPNsense Hosts capture confirms the mapping remains present after applying changes.
- Public repo version redacts the hardware/MAC address.
- FW-CR-003 is complete because the target remains associated with `ubuntutarget` at `10.10.10.167` and post-change target validation passed.

## Rollback Plan

If the reservation causes DHCP issues:

1. Remove/disable the static reservation.
2. Apply changes.
3. Reboot or renew the Ubuntu target network lease.
4. Verify the target receives an address in `10.10.10.100-10.10.10.199` and can reach the gateway/DNS.

If needed, revert the OPNsense snapshot:

```text
baseline-opnsense-dashboard-dhcp-rules-logs
```

## Evidence to Capture

| Evidence | Filename |
|---|---|
| DHCP reservation/static mapping page | `14-fw-cr-003-dhcp-reservation-created.png` |
| Target address, route, DNS server, and gateway validation after reservation | `15a-fw-cr-003-target-address-route-dns.png` |
| Target DNS query validation after reservation | `15b-fw-cr-003-target-dns-query.png` |

## Redaction Notes

- Private RFC1918 IPs are safe to show.
- MAC addresses are optional to redact before public GitHub publication.
- Do not expose OPNsense raw XML configuration backups.

## Interview Explanation

> After creating a firewall rule for one lab host, I converted that host's DHCP lease into a reservation. This prevents address drift from breaking firewall policy, scan targets, and documentation. It shows I understand that reliable security controls depend on stable asset identification, not just one-time rule creation.
