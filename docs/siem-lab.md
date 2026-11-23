# SIEM Lab: Wazuh deployment and firewall detections

Extension of the [segmented network lab](../README.md). Goal: centralize logs from the lab hosts in a SIEM and build detections on top of the firewall evidence that was previously analyzed manually.

**Status:** Complete for the scoped firewall-detection exercise. Two agents connected and all three custom rules generated alerts during authorized lab tests. Apache ingestion and Windows event analysis are follow-on work.

All test traffic was generated in an authorized lab environment.

## Design

| Component | Detail |
| --- | --- |
| SIEM | Wazuh 4.14.8 (OVA) in VirtualBox, 8 GB RAM, 4 vCPU |
| Placement | InternalNet, static 192.168.30.20/24 |
| Dashboard access | Separate Host-only adapter, used for dashboard access from the Windows host |
| Agents | `fedora-internal` (001) and `firewall-router` (002), both Active |
| Agent protocol | TCP 1514 to the manager |

**Why InternalNet:** the Firewall-Router already uses all four VirtualBox adapters (NAT, DMZNet, InternalNet, AttackerNet), so a dedicated management network was not possible without changing the router. The SIEM therefore sits in the internal zone, behind the firewall, with no Internet access.

See the [VirtualBox topology](../diagrams/virtualbox-topology.md) for network placement and the dashboard access path.

## Deployment summary

1. Imported the Wazuh OVA and attached it to InternalNet plus a Host-only adapter (no bridged networking).
2. Set a permanent static IP with systemd-networkd, matching the interface by MAC address. Verified after reboot.
3. Installed both agents **offline**, because the internal zone has no Internet by design: downloaded the package on the host, recorded its SHA-512, transferred it to the SIEM VM, served it temporarily over HTTP inside InternalNet, downloaded it on the target, and checked its SHA-512 after transfer.
4. Confirmed each connection in the agent log and in the dashboard.

## From iptables log line to alert

The router's iptables `LOG` rule records eligible forwarded packets toward the internal network before the drop rules with the prefix `LAB-INTERNAL-DENY` (limited to 6 per minute, burst 10).

- **Collection:** the Wazuh agent on the router already reads the systemd journal, which carries the kernel messages. No extra log file had to be configured.
- **Decoding:** `wazuh-logtest` showed that the built-in decoder extracts `srcip`, `dstip`, `srcport`, `dstport` and `protocol` from the line.
- **Rule matching:** out of the box the line matches rule 4100 ("Firewall rules grouped", level 0), which creates no alert. That is why the dashboard showed nothing at first. Custom rules were added on top of it.

## Custom detections

Rules are in [`siem/local_rules.xml`](../siem/local_rules.xml).

| Rule | Level | Logic | MITRE ATT&CK |
| --- | --- | --- | --- |
| 100100 | 6 | A firewall event matched the internal-deny log prefix | none assigned |
| 100101 | 10 | 5 matches of 100100 from the same source IP within 60 s | T1046 in shipped rule. Treat as repeated deny activity during triage. |
| 100102 | 12 | 5 matches of 100100 from the same source IP within 60 s, with different destination ports | T1046 |

The threshold of five accounts for the firewall's rate-limited logging. Higher thresholds would be harder to reach in this lab.

## Tests

| Test (from the attacker VM, target 192.168.30.10) | Expected | Observed |
| --- | --- | --- |
| `ping` and `curl` to TCP/8080 | Blocked and logged | Client checks failed. Separate captures show counter increases and 100100 alerts. |
| Multi-port validation | Possible scan alert | The 00:35 UTC client capture reports 100 filtered TCP ports and the export shows two 100101 alerts in that window. Two 100102 alerts appeared during a separate run at 00:45 UTC. |
| 6 consecutive `curl` requests to the same port 8080 | Repeated-attempts alert, **not** scan alert | 100101 fired twice. The captured dashboard query for 100102 returned no match during the single-port window. |

## November 23 alert investigation

Source: [selected Wazuh alert records](../logs/wazuh-lab-alerts-2026-11-23.json). The export retains up to five recent records per rule and includes timestamps, rule metadata, and decoded network fields. All records show TCP from Kali (`192.168.10.10`) to Fedora (`192.168.30.10`). Times below are UTC on November 23, 2026.

| Window | Observed in the export | Interpretation |
| --- | --- | --- |
| 00:35:24 | Two `100101` alerts with destination ports 23 and 71 | The repeated-attempt rule can also match across ports. |
| 00:45:12 | Two level-12 `100102` alerts with destination ports 69 and 15 | Correlates with the operator-confirmed multi-port test. These records show the triggering events included in the export. |
| 00:54:14–00:54:20 | Five `100100` alerts and two `100101` alerts for TCP/8080 | Correlates with the operator-confirmed single-port test. The 24-hour dashboard query for agent 002 with `LAB-INTERNAL-DENY` and rule `100102` covers this window and returns only the two earlier results. |

**Disposition:** Authorized detection validation for the two operator-confirmed test windows. The alerts establish rule matches. The [earlier segmentation revalidation](final-revalidation-2026-10-09.md) documents firewall enforcement using separate evidence, including counter changes and a service availability check.

## Screenshot evidence

Dashboard timestamps are displayed six hours behind the UTC JSON timestamps. The 100102 dashboard capture corroborates the 00:45 UTC alerts and covers the later single-port window. Its result is scoped to the displayed agent, query, and time range.

| Evidence | What it shows |
| --- | --- |
| 01 | [Two active agents in the dashboard](../screenshots/wazuh/01-agents-active.png) |
| 02 | [Windows package download, SHA-512 calculation, and transfer to the manager](../screenshots/wazuh/02-offline-transfer-windows.png) |
| 03 | [Early 100100 dashboard results](../screenshots/wazuh/03-alert-100100.png) |
| 04 | [Four 100101 results, including both the earlier scan and the single-port test](../screenshots/wazuh/04-alert-100101-single-port-test.png) |
| 05 | [Two 100102 dashboard results](../screenshots/wazuh/05-alert-100102-port-scan.png) |
| 06 | [Firewall-agent event overview](../screenshots/wazuh/06-events-overview.png) |
| 07 | [Kernel decoder fields and base rule 4100](../screenshots/wazuh/07-wazuh-logtest.png) |
| 08 | [Separate router counter and kernel-log capture](../screenshots/wazuh/08-firewall-counters-kernlog.png) |
| 09 | [Rule configuration check and manager service status](../screenshots/wazuh/09-rules-validation.png) |
| 10 | [Earlier scan result at 00:35 UTC](../screenshots/wazuh/10-nmap-scan-result.png) |
| 11 | [Six single-port client timeouts](../screenshots/wazuh/11-single-port-test.png) |
| 12 | [Fedora agent connection to the manager](../screenshots/wazuh/12-fedora-agent-connected.png) |

The router counter capture shows the LOG and Kali-to-internal DROP counters rising from 0 to 9, with TCP/8080 SYN records at 23:26 UTC on November 22. This is a separate capture from the 00:54 UTC alert window. The client screenshot shows six timeouts. The earlier scan screenshot is dated 00:35 GMT and supports the earlier 100101 window. The 00:45 detection is documented by the JSON and dashboard.

## Analyst note (how I would triage these alerts)

- **100100 alone:** a matching firewall log event. Check the source and destination against policy and look for a corresponding DROP verdict.
- **100101:** the same source keeps hitting a blocked path. Identify the source host and its owner, check what it is trying to reach, and look at its own logs for what is running there.
- **100102:** one source touching many ports. Investigate possible reconnaissance. If the source is unexpected, escalate; if it is a known scanner or an authorized test, document and close.
- Check the matching firewall verdict or counter changes to corroborate enforcement, and look for any accepted connections from the same source.

## Troubleshooting notes

| Problem | Cause | Fix |
| --- | --- | --- |
| Static IP lost after restarting networking | An older `20-eth0.network` also matched eth1 and took priority alphabetically | Renamed the new file to `10-eth1.network` |
| Agent failed to start on Fedora 43 | The `wazuh` user and group were not created by the RPM install | Created them manually, then reinstalled |
| Agent pointed to the wrong server | Wrong manager IP typed during install | Reinstalled and checked `ossec.conf` |
| Dashboard showed no firewall events | Built-in rule 4100 is level 0 (no alert) | Added custom rules |
| Router did not boot | Stuck early in boot | Booted after removing `quiet splash`. Long-term stability and root cause remain unverified. |
| Duplicate rule warning | Same rule block appended twice to `local_rules.xml` | Restored the backup and appended once; check with `wazuh-analysisd -t` before restarting |

## Limitations

- Thresholds are tuned for this lab. A real environment needs tuning to reduce false positives (for example a user retrying a blocked connection could trigger 100101).
- The firewall log is rate limited, so the SIEM sees a sample of the traffic. Correlation depends on how many matching events reach Wazuh within the configured window.
- Correlation tests covered one source host and TCP traffic. The exported sample does not establish full-window alert counts.
- Default credentials were used during setup and must be changed before any exposure.

## Follow-on work

- Narrow firewall rule allowing only DMZ -> SIEM on TCP 1514/1515, documented in the security policy, then the agent on the DMZ web server (Apache logs).
- Windows event analysis.
