# OPNsense Install Notes

## Install Status

OPNsense installation completed successfully on the dedicated `OPNsense-Firewall` VMware VM.

## Configuration Decisions

| Item | Value |
|---|---|
| Installer mode | Install to disk |
| Filesystem | ZFS |
| ZFS virtual device type | Stripe / no redundancy |
| Disk selected | VMware virtual disk `da0` |
| Root password | Changed by operator; not stored in repo |
| Reboot | Completed |

## Rationale

ZFS stripe/no redundancy is acceptable for this portfolio lab because the firewall VM uses a single virtual disk. In a production firewall deployment, storage redundancy and high availability would be evaluated based on business availability requirements.

## Next Verification Steps

After reboot:

1. Confirm OPNsense boots from the virtual disk, not the ISO.
2. Disconnect/remove the OPNsense ISO from the CD/DVD drive if it is still attached.
3. Confirm console interface assignments:
   - WAN should receive DHCP from VMware NAT.
   - LAN should have a private static IP.
4. Confirm the OPNsense web GUI is reachable from a VM on the LAN/host-only network.
5. Change LAN from default `192.168.1.1/24` to planned `10.10.10.1/24` if needed.

## Evidence Marker

`screenshots/04-opnsense-console-lan-wan-addresses.png` — capture the post-reboot console showing WAN/LAN IP assignments.

## Redaction

Do not capture or publish the root password. Redact public WAN IPs if a bridged/public adapter is ever used. VMware NAT/private RFC1918 IPs are acceptable for lab topology evidence.
