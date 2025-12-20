# IP Addressing Plan

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
