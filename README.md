# Cybersecurity Home Network Lab

A VirtualBox lab demonstrating IPv4 network segmentation, firewall validation,
and manual analysis of Linux firewall and Apache logs.

**Status:** Core segmentation tests completed; operational limitations remain
documented. All test traffic was generated in an authorized lab environment.

## Objective

Permit HTTP access to a DMZ web server while blocking tested connections from
the attacker and DMZ networks to the internal workstation. Evaluate the
results using service availability checks, firewall counters, and logs.

## Environment

| System | Address | Role |
|---|---|---|
| Kali Linux | 192.168.10.10/24 | Attacker test host |
| Ubuntu Server | 192.168.20.10/24 | DMZ web server running Apache |
| Fedora | 192.168.30.10/24 | Internal workstation |
| Ubuntu Firewall-Router | .1 gateway in each subnet | Routing, iptables filtering and logging |

Each zone uses a separate VirtualBox internal network. Firewall-Router also
has a VirtualBox NAT uplink. Only Ubuntu's explicitly scoped DNS and NTP
egress is permitted; general forwarded Internet access is not enabled.

See the [IP plan and interface mapping](diagrams/ip-addressing.md) for the
VirtualBox implementation and the earlier Cisco Packet Tracer design.

See the [VirtualBox topology](diagrams/virtualbox-topology.md) for network attachments
and the [reproduction guide](docs/reproduce-tests.md) for commands and expected results.

## Validated Results

| Test | Result | Supporting evidence |
|---|---|---|
| Kali and Fedora to DMZ TCP/80 | HTTP 200 | Client responses; Apache records for Kali |
| Kali and DMZ to internal ICMP | Blocked in tested paths | No replies and increasing DROP counters |
| Kali and DMZ to internal TCP/8080 | Blocked in tested paths | Timeouts, SYN logs and increasing DROP counters |
| Router to temporary internal TCP/8080 service | HTTP 200 | Service availability control |
| Firewall persistence | Original seven segmentation rules verified; later nine FORWARD and two NAT rules restored after boot recovery | [Follow-up](docs/ntp-recovery.md) |

The temporary TCP/8080 service was stopped after testing. A timeout alone was
not treated as proof of firewall enforcement.

## Evidence and Analysis

- [Network Segmentation Validation](docs/validation.md): current policy, test
  results, screenshots, log-export inventory and persistence checks.
- [SOC Analysis](docs/soc-analysis.md): evidence interpretation, analyst
  disposition and investigation steps for similar activity outside a test.
- [Security Policy](firewall-rules/security-policy.md): intended restrictions
  and implementation scope.
- [Setup History](docs/setup-history.md): earlier configuration steps and
  troubleshooting, including historical rules that are no longer current.

Reviewed [original log extracts](logs/README.md) and the
[exported IPv4 ruleset](firewall-rules/firewall-rules.v4) are available alongside
screenshots and written analysis.

## Skills Demonstrated

- IPv4 subnetting, routing and network segmentation.
- Stateful iptables forwarding rules and rate-limited logging.
- Positive and negative connectivity tests with a service availability control.
- Manual interpretation of firewall and Apache logs.
- Evidence-based conclusions and explicit documentation of uncertainty.

## Limitations and Follow-up

- Router and Ubuntu now report NTP synchronization; Ubuntu reboot verification
  remains pending. Historical logs retain their timestamp limitations.
  See [DNS/NTP recovery](docs/ntp-recovery.md).
- Intermittent VM startup problems remain unresolved.
- IPv6 filtering was not validated; router INPUT and OUTPUT remain ACCEPT.
- Results cover the tested paths and protocols, not every possible attack.
- This version does not include centralized SIEM ingestion or automated alerting.

The reproduction guide covers tests on a prepared lab; a fully automated fresh
build is outside this version's scope.
