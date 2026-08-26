# OPNsense Dashboard Baseline Checklist

## Purpose

Capture the first clean firewall-management evidence after OPNsense, VMnet2, DHCP, routing, and DNS have been validated.

This phase is intentionally read-only except for optional documentation/snapshot actions. Do not modify production/home network equipment; this applies only to the VMware lab firewall.

## Current Confirmed Network State

| Component | Value |
|---|---|
| OPNsense LAN | `10.10.10.254/24` |
| OPNsense WAN | VMware NAT / `192.168.109.0/24` |
| LAN DHCP range | `10.10.10.100`–`10.10.10.199` |
| Vulnerable target lease | `10.10.10.167/24` observed |
| Target gateway | `10.10.10.254` |
| Target DNS | `10.10.10.254` |
| Target internet test | Passed via `ping 8.8.8.8` |
| Target DNS test | Passed via `nslookup google.com` / `resolvectl query google.com` |

## Wizard Observation

After first web login, OPNsense opened `System > Configuration > Wizard` at the Welcome step. Since this lab has already configured interfaces, LAN IP, WAN DHCP, LAN DHCP, routing, and DNS from the console, the wizard should not be used unless intentionally changing those settings. For baseline evidence, stop/abort the wizard and navigate to the dashboard and read-only status pages.

## Captured Dashboard Baseline

Observed dashboard evidence for `01-opnsense-dashboard.png`:

```text
Page: Lobby > Dashboard
Name: OPNsense.internal
Version: OPNsense 26.7-amd64
FreeBSD: 15.1-RELEASE-p1
OpenSSL: 3.5.7
WAN: 192.168.109.135/24
LAN: 10.10.10.254/24
Gateway: WAN_DHCP active, 192.168.109.2
Memory: approximately 434 MB / 4045 MB
Disk: approximately 1.2 GB / 11.0 GB
Firewall widget: visible
Traffic graph: visible
Services: visible/running indicators
```

Interpretation:

- Screenshot satisfies `01-opnsense-dashboard.png` baseline evidence.
- It proves the firewall dashboard is reachable and shows correct LAN/WAN separation.

## DHCP Lease Page Observation

Observed `Services > Kea DHCP > Leases DHCPv4` showed no results:

```text
Showing 0 to 0 of 0 entries
```

Interpretation:

- The visible Kea leases page is not currently showing the target lease, even though the target received an OPNsense DHCP configuration (`10.10.10.167`, gateway/DNS `10.10.10.254`).
- Possible causes include: lease display needing refresh, the lease being held by a different DHCP backend/view, Kea control agent/service state, or the GUI not showing leases until services/cache update.
- Because client-side validation already proves DHCP/gateway/DNS, do not change DHCP settings yet. First check which DHCP backend is actually serving the lease.

Observed Kea DHCPv4 settings/subnets after empty leases/logs:

```text
Kea DHCPv4 Enabled: unchecked
Kea Interfaces: Nothing selected
Kea Subnets: no results found
Kea Logs: empty
```

Interpretation:

- Empty Kea leases/logs are expected because Kea is not enabled or configured.
- The target's working DHCP/gateway/DNS must be coming from another backend, likely `Services > Dnsmasq DNS & DHCP`, or a legacy/console-created DHCP service path.
- Do not enable Kea just to populate the lease table; first verify the active DHCP backend.

## Dashboard Evidence to Capture

| Screenshot | OPNsense Location | Capture Goal | Redact |
|---|---|---|---|
| `01-opnsense-dashboard.png` | Dashboard home | Shows firewall platform is reachable and operational | Browser profile/bookmarks, public IPs if visible |
| `04-opnsense-console-lan-wan-addresses.png` | OPNsense console LAN/WAN status | Shows LAN/WAN addressing and management URL | Public IPs if not RFC1918 |
| `07-opnsense-dhcp-leases.png` | Services > Dnsmasq DNS & DHCP > Leases | Shows `ubuntutarget` received DHCP lease `10.10.10.167` on LAN | MAC address optional; hostname if personal |
| `08-opnsense-firewall-rules-baseline.png` | Firewall > Rules > LAN | Shows default/baseline LAN rules before hardening | None usually, unless personal aliases exist |
| `09-opnsense-firewall-live-log-baseline.png` | Firewall > Log Files > Live View | Shows baseline logging visibility | Public IPs, hostnames if personal |

## Captured Firewall Rule Baseline

Observed `08-opnsense-firewall-rules-baseline.png`:

```text
Page: Firewall > Rules
Selected interface: LAN
Visible LAN interface rules:
- IPv4: LAN network * -> * * ; Description: Default allow LAN to any rule
- IPv6: LAN network * -> * * ; Description: Default allow LAN IPv6 to any rule
```

Interpretation:

- This is the default permissive LAN baseline.
- It is useful before/after evidence because future firewall hardening can show controlled changes from this state.

## Captured Firewall Log Baseline

Observed `09-opnsense-firewall-live-log-baseline.png` / firewall live view:

```text
Page: Firewall > Log Files > Live View
Examples:
- LAN TCP from 10.10.10.1 ephemeral ports to 10.10.10.254:443: pass, anti-lockout rule
- WAN UDP from 192.168.109.1:63437 to 192.168.109.255:21027: block, Block private networks from WAN
- WAN UDP NTP from 192.168.109.135:123 to public NTP servers: pass, let out anything from firewall host itself
```

Interpretation:

- Logging is functional.
- Green rows show permitted management/firewall-originated traffic.
- Red rows show blocked WAN-side private/broadcast traffic.
- Public NTP destination IPs appear in logs; redact if preferred for public screenshots, though they are third-party NTP IPs rather than home public IPs.

## Baseline Checks in Dashboard

### 1. Confirm Interfaces

Look for:

```text
LAN: 10.10.10.254/24
WAN: 192.168.109.x/24 via DHCP
```

Expected interpretation:

- LAN is the protected VMnet2 network.
- WAN is the VMware NAT side.

### 2. Confirm DHCP Lease

Look for target lease:

```text
10.10.10.167 or another 10.10.10.100-199 address
```

Expected interpretation:

- OPNsense is the DHCP authority for lab clients.
- The vulnerable target is behind the firewall.

### 3. Confirm Baseline Firewall Rules

On LAN, OPNsense usually has default allow rules for LAN traffic. Capture them before making changes.

Why this matters:

> Firewall management is strongest when you can show before/after policy states, not just final rules.

### 4. Confirm Logs

Open firewall live view or system logs and confirm logs are available. Do not worry if there are not many alerts yet; we have not generated traffic intentionally.

## Snapshot Recommendation

After screenshots are captured and before changing rules or enabling IDS/IPS, take a VMware snapshot:

```text
OPNsense-Firewall snapshot: baseline-dashboard-dhcp-working
Vuln-Ubuntu-Target snapshot: baseline-opnsense-dhcp-working
```

## Interview Explanation

"After deploying OPNsense, I validated that the firewall had a dedicated LAN and WAN, confirmed that lab clients received DHCP leases from the firewall, verified gateway/DNS/internet connectivity from the protected subnet, and captured a baseline firewall rule state before making policy changes. This gave me a clean starting point for firewall management, vulnerability scanning, and threat-hunting evidence."
