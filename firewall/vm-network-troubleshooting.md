# VM Network Troubleshooting Notes

## Symptom

Windows 11 VM shows:

- Network & internet: Not connected
- Ethernet0: No internet

## Likely Lab Causes

During the OPNsense build, a VM can lose internet if its VMware adapter is moved from NAT to Host-only/VMnet before OPNsense routing/DHCP is fully verified.

Common causes:

1. Windows VM adapter changed from NAT to Host-only/VMnet2.
2. OPNsense VM is powered off, so Host-only LAN clients have no firewall/router.
3. Windows is on a different host-only network than OPNsense LAN.
4. OPNsense LAN DHCP is not enabled or not serving the expected scope.
5. Windows still has an old IP/gateway/DNS lease.
6. VMware adapter is not connected or not connected at power on.

## Observed Error During Windows DHCP Recovery

When attempting DHCP recovery, Windows may report:

```text
An error occurred while releasing interface Ethernet0 : An address has not yet been associated with the network endpoint.

An error occurred while renewing interface Ethernet0 : The name specified in the network control block (NCB) is in use on a remote adapter.
The NCB is the data.
```

Interpretation:

- The release error is expected when the adapter only has an APIPA/link-local address and no real DHCP lease.
- The NCB error often points to a stale/conflicted Windows network adapter state, duplicate NetBIOS name, duplicate/cloned virtual adapter identity, or VMware NAT/DHCP service issue.
- Recovery should reset the Windows NIC state and, if needed, restart VMware NAT/DHCP on the host.

## Quick Restore Path

To immediately restore internet outside the firewall lab:

1. Shut down or suspend the Windows VM if desired.
2. VMware > Windows VM Settings > Network Adapter.
3. Set adapter back to `NAT`.
4. Ensure `Connected` and `Connect at power on` are checked.
5. Boot Windows and run:

```powershell
ipconfig /release
ipconfig /renew
ipconfig /all
```

Expected NAT state:

- IPv4 from VMware NAT subnet
- Default gateway present
- DNS servers present
- Internet reachable

## Firewall-Lab Path

To keep Windows behind OPNsense:

1. Keep OPNsense powered on.
2. OPNsense Adapter 1 = NAT/WAN.
3. OPNsense Adapter 2 = Host-only/VMnet2/LAN.
4. Windows Adapter = same Host-only/VMnet2 network as OPNsense LAN.
5. OPNsense LAN DHCP enabled.
6. Windows receives DHCP lease from OPNsense.

Expected behind-firewall state:

- Windows IPv4 from OPNsense LAN subnet
- Default gateway = OPNsense LAN IP
- DNS = OPNsense LAN IP or upstream DNS through OPNsense

## Host Reachability Finding — OPNsense GUI Timeout

Observed physical host `ipconfig` showed:

```text
VMware Network Adapter VMnet1: 192.168.153.1/24
VMware Network Adapter VMnet8: 192.168.109.1/24
Physical LAN: 192.168.4.108/22
Ping 192.168.1.1: timeout
```

OPNsense console showed:

```text
LAN (em0): 192.168.1.1/24
WAN (em1): 192.168.153.128/24
```

Interpretation:

- `VMnet8` is the VMware NAT network used by existing VMs (`192.168.109.0/24`).
- `VMnet1` is the VMware host-only network (`192.168.153.0/24`).
- OPNsense WAN receiving `192.168.153.128` means its WAN is currently on the host-only network, not VMware NAT.
- OPNsense LAN is `192.168.1.1/24`, but the host has no adapter on `192.168.1.0/24`, so the host cannot reach the GUI.

Likely fix:

1. Assign OPNsense WAN to the NAT-side NIC.
2. Assign OPNsense LAN to the host-only/custom lab NIC.
3. Put the LAN on a subnet reachable through that host-only adapter, preferably a dedicated custom VMnet with VMware DHCP disabled.

## Diagnostic Commands

On Windows:

```powershell
ipconfig /all
ping 127.0.0.1
ping <default-gateway>
ping 8.8.8.8
nslookup google.com
```

Interpretation:

- No IPv4 or `169.254.x.x`: DHCP problem.
- Gateway ping fails: wrong VMnet or firewall/router not reachable.
- `ping 8.8.8.8` works but `nslookup google.com` fails: DNS problem.
- Gateway and DNS work but browser fails: proxy/browser/VPN issue.
