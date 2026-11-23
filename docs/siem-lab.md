# SIEM Lab: Wazuh deployment (work in progress)

Extension of the [segmented network lab](../README.md). Goal: centralize logs from the lab hosts in a SIEM and build detections on top of the firewall and web server evidence already analyzed manually.

**Status:** SIEM deployed and first agent connected. Router and DMZ log collection and detection rules are in progress. No automated detections are claimed yet.

## Design

| Component | Detail |
| --- | --- |
| SIEM | Wazuh 4.14.8 (OVA) in VirtualBox, 8 GB RAM, 4 vCPU |
| Placement | InternalNet, static 192.168.30.20/24 |
| Dashboard access | Separate Host-only adapter, reachable only from the host PC |
| Agents | Fedora 43 (agent 001) connected; router and DMZ planned |
| Agent protocol | TCP 1514 to the manager |

**Why InternalNet:** the Firewall-Router already uses all four VirtualBox adapters (NAT, DMZNet, InternalNet, AttackerNet), so a dedicated management network was not possible without changing the router. The SIEM therefore sits in the internal zone, behind the firewall, with no Internet access.

<!-- Add your updated topology image here -->

## What was done

1. Imported the Wazuh OVA and attached it to InternalNet plus a Host-only adapter (no bridged networking, so it stays inside the lab).
2. Set a permanent static IP with systemd-networkd, matching the interface by MAC address. Verified after reboot.
3. Installed the Wazuh agent on Fedora **offline**, because the internal zone has no Internet by design: downloaded the RPM on the host, verified its SHA-512, transferred it to the SIEM VM, served it temporarily over HTTP inside InternalNet, downloaded it on Fedora, and verified the SHA-512 again. The temporary web server was stopped afterwards.
4. Confirmed the connection in two places: the agent log (`Connected to the server ([192.168.30.20]:1514/tcp)`) and the dashboard (agent 001 Active, events arriving).

<!-- Screenshot: dashboard agent list showing fedora-internal Active (hide any credentials) -->

## Troubleshooting notes

| Problem | Cause | Fix |
| --- | --- | --- |
| Static IP lost after restarting networking | An older `20-eth0.network` file also matched eth1 and took priority alphabetically | Renamed the new file to `10-eth1.network` |
| Agent service failed to start on Fedora 43 | The `wazuh` user and group were not created by the RPM install | Created them manually, then reinstalled |
| Agent pointed to the wrong server | Wrong manager IP typed during install | Reinstalled with the correct address and checked `ossec.conf` |

## Next

- Agent on the Firewall-Router to collect iptables logs.
- Narrow firewall rule allowing only DMZ -> SIEM on TCP 1514/1515, documented in the security policy, then agent on the DMZ web server.
- Three detections: blocked traffic toward the internal network, port scan, repeated connection attempts. Test with Nmap from Kali and map to MITRE ATT&CK.

## Limitations

- Lab environment only; default credentials were used during setup and must be changed before any exposure.
- Detection rules are not implemented yet.
