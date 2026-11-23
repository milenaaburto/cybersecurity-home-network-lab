# SIEM Lab: Wazuh deployment and firewall detections

Extension of the [segmented network lab](../README.md). Goal: centralize logs from the lab hosts in a SIEM and build detections on top of the firewall evidence that was previously analyzed manually.

**Status:** SIEM deployed, two agents connected, three custom detections tested with controlled traffic from the attacker VM. DMZ web server collection is still planned.

All test traffic was generated in an authorized lab environment.

## Design

| Component | Detail |
| --- | --- |
| SIEM | Wazuh 4.14.8 (OVA) in VirtualBox, 8 GB RAM, 4 vCPU |
| Placement | InternalNet, static 192.168.30.20/24 |
| Dashboard access | Separate Host-only adapter, reachable only from the host PC |
| Agents | `fedora-internal` (001) and `firewall-router` (002), both Active |
| Agent protocol | TCP 1514 to the manager |

**Why InternalNet:** the Firewall-Router already uses all four VirtualBox adapters (NAT, DMZNet, InternalNet, AttackerNet), so a dedicated management network was not possible without changing the router. The SIEM therefore sits in the internal zone, behind the firewall, with no Internet access.

<!-- Add your updated topology image here -->

## Deployment summary

1. Imported the Wazuh OVA and attached it to InternalNet plus a Host-only adapter (no bridged networking).
2. Set a permanent static IP with systemd-networkd, matching the interface by MAC address. Verified after reboot.
3. Installed both agents **offline**, because the internal zone has no Internet by design: downloaded the package on the host, verified its SHA-512, transferred it to the SIEM VM, served it temporarily over HTTP inside InternalNet, downloaded it on the target, and verified the SHA-512 again.
4. Confirmed each connection in the agent log and in the dashboard.

<!-- Screenshot: dashboard agent list showing both agents Active (hide any credentials) -->

## From iptables log line to alert

The router's iptables `LOG` rule writes blocked traffic toward the internal network with the prefix `LAB-INTERNAL-DENY` (limited to 6 per minute, burst 10).

- **Collection:** the Wazuh agent on the router already reads the systemd journal, which carries the kernel messages. No extra log file had to be configured.
- **Decoding:** `wazuh-logtest` showed that the built-in decoder extracts `srcip`, `dstip`, `srcport`, `dstport` and `protocol` from the line.
- **Rule matching:** out of the box the line matches rule 4100 ("Firewall rules grouped", level 0), which creates no alert. That is why the dashboard showed nothing at first. Custom rules were added on top of it.

## Custom detections

Rules are in [`siem/local_rules.xml`](../siem/local_rules.xml).

| Rule | Level | Logic | MITRE ATT&CK |
| --- | --- | --- | --- |
| 100100 | 6 | A blocked connection toward the internal network was logged | none assigned |
| 100101 | 10 | 5 matches of 100100 from the same source IP within 60 s | T1046 |
| 100102 | 12 | 5 matches of 100100 from the same source IP within 60 s, with different destination ports | T1046 |

The threshold of 5 is deliberate: the firewall's `LOG` rule is rate limited, so a threshold of 20 would never be reached.

## Tests

| Test (from the attacker VM, target 192.168.30.10) | Expected | Observed |
| --- | --- | --- |
| `ping` and `curl` to TCP/8080 | Blocked and logged | Both failed; `LOG` and `DROP` counters rose; 100100 fired |
| Nmap SYN scan, ports 1-100, `-Pn` | Scan alert | All 100 ports reported `filtered`; 100102 fired twice |
| 6 consecutive `curl` requests to the same port 8080 | Repeated-attempts alert, **not** scan alert | 100101 fired twice; 100102 did not fire |

<!-- Screenshot: dashboard search rule.id:100102 -->
<!-- Screenshot: dashboard search rule.id:(100101 or 100102) after the single-port test -->

Observation: while the scan ran, only 100102 added alerts and 100101 did not add new ones. I have not investigated why Wazuh behaves this way, so I only record it.

## Analyst note (how I would triage these alerts)

- **100100 alone:** one or two blocked attempts. Check source and destination against the policy. Usually a misconfiguration or a one-off probe.
- **100101:** the same source keeps hitting a blocked path. Identify the source host and its owner, check what it is trying to reach, and look at its own logs for what is running there.
- **100102:** one source touching many ports. Treat as reconnaissance. If the source is unexpected, escalate; if it is a known scanner or an authorized test, document and close.
- In every case, the firewall dropped the traffic. The alert tells me someone tried, not that anything got through, so I would also confirm there are no *accepted* connections from the same source.

## Troubleshooting notes

| Problem | Cause | Fix |
| --- | --- | --- |
| Static IP lost after restarting networking | An older `20-eth0.network` also matched eth1 and took priority alphabetically | Renamed the new file to `10-eth1.network` |
| Agent failed to start on Fedora 43 | The `wazuh` user and group were not created by the RPM install | Created them manually, then reinstalled |
| Agent pointed to the wrong server | Wrong manager IP typed during install | Reinstalled and checked `ossec.conf` |
| Dashboard showed no firewall events | Built-in rule 4100 is level 0 (no alert) | Added custom rules |
| Router did not boot | Stuck early in boot | Removed `quiet splash` from GRUB and ran `update-grub` |
| Duplicate rule warning | Same rule block appended twice to `local_rules.xml` | Restored the backup and appended once; check with `wazuh-analysisd -t` before restarting |

## Limitations

- Thresholds are tuned for this lab. A real environment needs tuning to reduce false positives (for example a user retrying a blocked connection could trigger 100101).
- The firewall log is rate limited, so the SIEM sees a sample, not every packet. A slow scan that stays under the log rate could avoid the alerts.
- Tests covered one source host and TCP SYN traffic only.
- Default credentials were used during setup and must be changed before any exposure.

## Next

- Narrow firewall rule allowing only DMZ -> SIEM on TCP 1514/1515, documented in the security policy, then the agent on the DMZ web server (Apache logs).
- Windows event analysis.
