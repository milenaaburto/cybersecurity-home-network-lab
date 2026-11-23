# Cybersecurity Home Network Lab

A VirtualBox lab that segments a network into attacker, DMZ and internal zones behind an Ubuntu iptables firewall, then validates selected IPv4 paths with controlled tests, firewall counters, and log analysis. The Wazuh extension adds three custom detections.

## Summary

- **What I built:** three isolated networks (Kali attacker, Ubuntu DMZ web server, Fedora internal workstation) routed through an Ubuntu firewall with stateful iptables rules and rate-limited logging.
- **What I found:** HTTP to the DMZ worked. The tested ICMP and TCP/8080 connections to the internal host were blocked, and the DROP counters increased by five per source in the final revalidation.
- **How I avoided a false conclusion:** a timeout alone is not proof of a block. I started a temporary service on the internal host, confirmed it answered (HTTP 200 from the router), then checked matching SYN logs and DROP counter increases before attributing the timeouts to firewall enforcement.
- **What I did with the logs:** broke a firewall log record down field by field, reviewed Apache access logs, and wrote an analyst disposition covering what the evidence supports and what it does not.
- **SIEM extension:** Wazuh deployed with two agents and three tested detections on the firewall logs (see the [Wazuh alert investigation](docs/siem-lab.md#november-23-alert-investigation)).

All test traffic was generated in an authorized lab environment.

## Environment

| System                 | Address                   | Role                                    |
| ---------------------- | ------------------------- | --------------------------------------- |
| Kali Linux             | 192.168.10.10/24          | Attacker test host                      |
| Ubuntu Server          | 192.168.20.10/24          | DMZ web server running Apache           |
| Fedora                 | 192.168.30.10/24          | Internal workstation                    |
| Wazuh manager          | 192.168.30.20/24          | Firewall alert collection and correlation |
| Ubuntu Firewall-Router | .1 gateway in each subnet | Routing, iptables filtering and logging |

Each zone uses a separate VirtualBox internal network. The router also has a NAT uplink, but only explicitly scoped forwarded DNS and NTP egress from the DMZ web server is permitted; general forwarded Internet access is not enabled.

Details: [IP plan](diagrams/ip-addressing.md) · [VirtualBox topology](diagrams/virtualbox-topology.md) · [Reproduction guide](docs/reproduce-tests.md)

## Validated Results

| Test                                          | Result                  | Supporting evidence                                  |
| --------------------------------------------- | ----------------------- | ---------------------------------------------------- |
| Kali and Fedora to DMZ TCP/80                 | HTTP 200                | Client responses; Apache records for Kali            |
| Kali and DMZ to internal ICMP                 | Blocked in tested paths | No replies and increasing DROP counters              |
| Kali and DMZ to internal TCP/8080             | Blocked in tested paths | Timeouts, SYN logs and increasing DROP counters      |
| Router to temporary internal TCP/8080 service | HTTP 200                | Service availability control                         |
| Firewall persistence                          | Rules restored after reboot | [Follow-up](docs/appendix/ntp-recovery.md)       |

The temporary TCP/8080 service was stopped after testing.

## Evidence and Analysis

- [Final revalidation, October 9](docs/final-revalidation-2026-10-09.md): repeated tests, counter changes and cleanup.
- [Network Segmentation Validation](docs/validation.md): policy, results, screenshots and persistence checks.
- [SOC Analysis](docs/soc-analysis.md): how the evidence was interpreted, analyst disposition, and next steps if similar activity occurred outside a test.
- [Security Policy](firewall-rules/security-policy.md): intended restrictions and scope.
- [Original log extracts](logs/README.md) and the [exported IPv4 ruleset](firewall-rules/firewall-rules.v4).
- [Setup history](docs/setup-history.md): earlier configuration and troubleshooting, including historical rules no longer in use.
- [Wazuh alert export](logs/wazuh-lab-alerts-2026-11-23.json) and [custom rules](siem/local_rules.xml).

## Skills Demonstrated

- IPv4 subnetting, routing and network segmentation.
- Stateful iptables forwarding rules and rate-limited logging.
- Positive and negative connectivity tests with a service availability control.
- Manual interpretation of firewall and Apache logs.
- Evidence-based conclusions with explicit documentation of uncertainty.

## Project closure

The October validation completed the scoped IPv4 segmentation and manual analysis. The November extension completed firewall alert collection and validation of three Wazuh rules. Follow-on work includes Apache ingestion, Windows event analysis, and host hardening.

## Limitations

- Results cover the tested IPv4 paths and protocols, not every possible attack.
- IPv6 filtering was not validated, and router INPUT/OUTPUT policies remain ACCEPT.
- Firewall logging is rate-limited, so the published extracts are not a complete packet history.
- The base lab analysis is manual. Automated detection exists only in the SIEM extension, with thresholds tuned for the lab.

Operational notes (clock synchronization and intermittent VM startup) are in the [appendix](docs/appendix/ntp-recovery.md).
