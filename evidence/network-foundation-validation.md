# Network Foundation Validation

## Objective

Validate that the integrated cyber defense lab has a working routed firewall architecture before vulnerability scanning, threat hunting, and controlled firewall policy testing.

## Validated Topology

```text
Vuln-Ubuntu-Target
10.10.10.167/24
        |
        | VMnet2 protected lab LAN
        |
OPNsense LAN
10.10.10.254/24
        |
OPNsense WAN
192.168.109.135/24 via VMware NAT
        |
Internet
```

A Greenbone/OpenVAS scanner VM was also placed on the protected lab LAN:

```text
Greenbone_OpenVAS
10.10.10.100/24
        |
        | VMnet2 protected lab LAN
        |
OPNsense LAN / DNS / default gateway
10.10.10.254/24
```

## Validation Results

| Test | Result | Evidence |
|---|---|---|
| OPNsense LAN/WAN assigned | Passed | LAN `10.10.10.254/24`, WAN on VMware NAT |
| Target DHCP lease from lab network | Passed | Target received `10.10.10.167/24` |
| Target default gateway | Passed | `default via 10.10.10.254 dev ens33 proto dhcp` |
| Target DNS server | Passed | `DNS Servers: 10.10.10.254` |
| Target ping firewall LAN | Passed | `ping -c 4 10.10.10.254` returned `0%` packet loss |
| Target internet/DNS baseline | Passed | `ping`, `nslookup`, and `resolvectl query` returned expected results before custom block testing |
| Controlled ICMP block | Passed | FW-CR-002 blocked ICMP from `10.10.10.167` to `8.8.8.8` while gateway and DNS continued working |
| Firewall log validation | Passed | OPNsense live logs showed the blocked ICMP traffic under the named firewall rule |
| DHCP reservation | Passed | `ubuntutarget` reservation created for `10.10.10.167` |
| Greenbone scanner network placement | Passed | Scanner received `10.10.10.100/24`, used gateway/DNS `10.10.10.254`, and reached the target |

## Evidence Files

| Screenshot | Purpose |
|---|---|
| `screenshots/01-opnsense-dashboard.png` | OPNsense operational dashboard baseline |
| `screenshots/02-opnsense-configuration-wizard.png` | Initial OPNsense configuration workflow |
| `screenshots/03-vmware-vmnet2-network-editor.png` | VMnet2 isolated lab network configuration |
| `screenshots/04-opnsense-console-lan-wan-addresses.png` | OPNsense LAN/WAN console addressing |
| `screenshots/05-target-address-route-dns-baseline.png` | Target addressing/routing/DNS baseline |
| `screenshots/06-target-internet-dns-baseline.png` | Target internet and DNS baseline |
| `screenshots/07-opnsense-dhcp-leases.png` | DHCP lease for `ubuntutarget` |
| `screenshots/08-opnsense-firewall-rules-baseline.png` | Baseline firewall rule state |
| `screenshots/09-opnsense-firewall-live-log-baseline.png` | Baseline firewall log visibility |
| `screenshots/10-target-after-snapshot-validation.png` | Known-good post-snapshot validation |
| `screenshots/11-fw-cr-002-rule-created.png` | Controlled ICMP block rule implementation |
| `screenshots/12-fw-cr-002-validation-blocked-icmp.png` | Target-side block validation |
| `screenshots/13-fw-cr-002-block-log-evidence.png` | Firewall log proof of blocked traffic |
| `screenshots/14-fw-cr-003-dhcp-reservation-created.png` | DHCP reservation/static mapping |
| `screenshots/15a-fw-cr-003-target-address-route-dns.png` | Target stability after reservation |
| `screenshots/15b-fw-cr-003-target-dns-query.png` | DNS query validation after reservation |
| `screenshots/16-greenbone-scanner-network-validation.png` | Scanner network reachability validation |
| `screenshots/16a-greenbone-install-network-config.png` | Scanner VM installer network configuration |

## Security/Operational Interpretation

The vulnerable target and scanner are both inside the protected lab LAN behind OPNsense. OPNsense provides gateway, DNS/DHCP, firewall policy enforcement, and log visibility. This establishes a realistic defensive foundation for later Greenbone vulnerability scanning and Wazuh/Sysmon threat-hunting evidence.

The firewall test demonstrates the important operational distinction between **blocking one risky traffic path** and **breaking host connectivity**: ICMP to `8.8.8.8` was intentionally denied, while local gateway access and DNS resolution remained functional.
