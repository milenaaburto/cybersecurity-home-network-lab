# IP Addressing Plan

## Design and Implementation

The R1 interfaces below belong to the Cisco Packet Tracer design.
The VirtualBox implementation uses an Ubuntu firewall/router with
the following interface mapping:

| VirtualBox interface | IPv4 address | Network |
|---|---|---|
| enp0s10 | 192.168.10.1/24 | Attacker |
| enp0s8 | 192.168.20.1/24 | DMZ |
| enp0s9 | 192.168.30.1/24 | Internal |
| enp0s3 | DHCP; observed 10.0.2.15/24 | VirtualBox NAT uplink |

The NAT uplink belongs to Firewall-Router. Its presence does not
mean that forwarded Internet access is permitted for lab hosts.

See [current validation](../docs/validation.md) for tested behavior.

## Router Interfaces

| Device | Interface | IP Address | Network |
|------|----------|-----------|--------|
| R1 | G0/0 | 192.168.10.1 | Attacker Network |
| R1 | G0/1 | 192.168.20.1 | DMZ |
| R1 | G0/2 | 192.168.30.1 | Internal LAN |

## End Devices

| Device | IP Address | Gateway | Network |
|------|-----------|--------|--------|
| PC-Attacker | 192.168.10.10 | 192.168.10.1 | Attacker |
| Server-DMZ | 192.168.20.10 | 192.168.20.1 | DMZ |
| PC-User1 | 192.168.30.10 | 192.168.30.1 | Internal |
