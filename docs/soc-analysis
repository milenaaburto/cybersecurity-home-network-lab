# SOC Analysis: Network Segmentation Validation

## Objective

Assess whether the lab firewall permits intended HTTP access to
the DMZ while blocking tested connections from the attacker and
DMZ networks to the internal workstation.

This was an authorized lab exercise. The events below were
generated deliberately, not discovered during a real incident.

## Findings

| Source | Destination | Test | Observed result |
| --- | --- | --- | --- |
| Kali — 192.168.10.10 | DMZ — 192.168.20.10 | HTTP, TCP/80 | HTTP 200; Apache access entry |
| Fedora — 192.168.30.10 | DMZ — 192.168.20.10 | HTTP, TCP/80 | HTTP 200 |
| Kali — 192.168.10.10 | Fedora — 192.168.30.10 | ICMP | No replies; matching firewall drop counter increased |
| DMZ — 192.168.20.10 | Fedora — 192.168.30.10 | ICMP | No replies; matching firewall drop counter increased |
| Kali and DMZ | Fedora — 192.168.30.10 | TCP/8080 | Requests timed out; matching firewall logs and drop counters |

## How the Evidence Was Evaluated

A connection timeout alone does not establish that a firewall
blocked traffic: an unavailable service could produce a similar
result.

For the TCP/8080 test, a temporary HTTP server was started on
Fedora. Firewall-Router received HTTP 200 from that service,
confirming it was available from the router.

Requests from Kali and Ubuntu-Server then timed out, while
matching firewall log entries and DROP counters supported
attribution to firewall enforcement.

The temporary HTTP server was stopped after testing.

## Firewall Log Analysis

The exported TCP evidence contained ten SYN log entries:
five from Kali and five from Ubuntu-Server, directed to
192.168.30.10:8080.

Relevant fields included source and destination IP addresses,
source and destination ports, ingress and egress interfaces,
and TCP flags.

Repeated SYN packets included connection retries. Ten log
entries must not be interpreted as ten separate attacks.

The LAB-INTERNAL-DENY prefix identifies the logging rule.
Logging itself does not block traffic; the subsequent DROP
rules enforce the policy.

## Apache Log Analysis

Four Kali access-log entries were exported from Ubuntu-Server.
The reviewed entries recorded HEAD requests and HTTP 200
responses.

These records demonstrate application-level visibility of
permitted HTTP traffic. A successful HTTP response does not
establish that the application is free of vulnerabilities.

The source address and reported curl User-Agent are consistent
with the test context, but neither alone proves malicious intent.

## Analyst Disposition

Classification: authorized security-control validation.

The tested connections behaved as intended: HTTP access to
the DMZ succeeded, while the tested attacker-to-internal and
DMZ-to-internal connections were blocked.

The reviewed evidence does not establish a compromise.

If similar blocked connections occurred outside an approved
test, the next steps would be to identify the source host and
owner, review endpoint process and authentication records,
check the scope of attempted destinations, and investigate
any associated successful connections.

## Limitations

- Ubuntu-Server clock synchronization remains unresolved.
  Precise cross-host timestamp correlation was not established.
- Firewall logging is rate-limited; logs are not a complete
  record of every dropped packet.
- Results apply to the tested IPv4 paths and protocols.
  IPv6 enforcement was not validated.
- Router INPUT and OUTPUT policies remain ACCEPT; router
  hardening was not demonstrated by these forwarding tests.
- Intermittent VM boot problems remain an operational issue.
- Raw log exports remain on the VMs; screenshots and the
  validation document provide the current repository evidence.
- No centralized SIEM ingestion or automated alerting was
  implemented in this exercise.

## Supporting Documentation

See [Network Segmentation Validation](validation.md) for
the topology, test procedure, screenshots, and persistence checks.
