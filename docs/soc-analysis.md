# SOC Analysis: Network Segmentation Validation

## Objective

Assess whether the lab firewall permits intended HTTP access to
the DMZ while blocking tested connections from the attacker and
DMZ networks to the internal workstation.

This was an authorized lab exercise. The events below were
generated deliberately, not discovered during a real incident.

## Findings

The tested HTTP paths to the DMZ succeeded; the tested ICMP and TCP paths
from Kali and Ubuntu to the internal workstation were blocked. See the
[validation matrix](validation.md) and [October 9 counter measurements](final-revalidation-2026-10-09.md)
for individual test results. This document focuses on interpretation.

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

The [October 9 sanitized export](../logs/tcp-deny-revalidation-2026-10-09-sanitized.log)
contains ten SYN records: five from Kali and five from Ubuntu. Below is its
first record, with line breaks added for readability only:

```text
2026-10-09T20:00:05+00:00 Firewall-Router kernel: LAB-INTERNAL-DENY
IN=enp0s10 OUT=enp0s9 MAC=[REDACTED_MAC]
SRC=192.168.10.10 DST=192.168.30.10 LEN=60 TOS=0x00 PREC=0x00
TTL=63 ID=40798 DF PROTO=TCP SPT=45374 DPT=8080
WINDOW=64240 RES=0x00 SYN URGP=0
```

| Field | Interpretation |
|---|---|
| Timestamp and +00:00 | Router-recorded event time in UTC; timezone alone does not prove clock accuracy. |
| Firewall-Router kernel | Host and component producing the record. |
| LAB-INTERNAL-DENY | Configured LOG prefix, not a DROP verdict. |
| IN=enp0s10 | Ingress interface for the attacker network. |
| OUT=enp0s9 | Routed output interface toward the internal network, not proof of transmission. |
| MAC=[REDACTED_MAC] | Link-layer identifiers removed from the publication copy. |
| SRC / DST | Kali test host / Fedora internal workstation. |
| LEN=60 | Logged IP packet length in bytes. |
| TOS / PREC | Logged IP service/precedence values, both zero. |
| TTL=63 | Remaining IP time-to-live; insufficient to identify the sender's operating system. |
| ID=40798 / DF | IPv4 identification value and Don't Fragment flag. |
| PROTO=TCP | Transport protocol. |
| SPT=45374 / DPT=8080 | Client source port / temporary service destination port. |
| WINDOW=64240 | Advertised TCP receive window field. |
| RES=0x00 | Logged TCP reserved bits are zero. |
| SYN | Connection initiation flag; does not establish a completed connection. |
| URGP=0 | TCP urgent pointer value. |

### What the evidence supports

The router observed a TCP connection attempt from Kali to Fedora's test service.
The service control returned HTTP 200; client attempts timed out, and the matching
DROP counters increased by five per source. Together these observations support
firewall enforcement for the tested paths.

### What the evidence does not establish

This line alone does not prove a drop, malicious intent, compromise or a completed
connection. Repeated SYNs with the same endpoints and ports are consistent with
retries; TCP sequence numbers are absent, so exact retransmission identification
requires packet-level evidence. Ten records do not mean ten separate attacks.
Rate-limited logging also prevents treating the extract as a complete packet history.

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

- Router and Ubuntu synchronization was subsequently observed, including Ubuntu
  after a normal startup. Original logs retain their historical clock limitations;
  precise inter-host offset and all-host alignment were not established.
- Firewall logging is rate-limited; logs are not a complete
  record of every dropped packet.
- Results apply to the tested IPv4 paths and protocols.
  IPv6 enforcement was not validated.
- Router INPUT and OUTPUT policies remain ACCEPT; router
  hardening was not demonstrated by these forwarding tests.
- Intermittent VM boot problems remain an operational issue.
- Published logs are selected extracts, not complete event histories.
  See the [evidence inventory](../logs/README.md) for sources and scope.
- No centralized SIEM ingestion or automated alerting was
  implemented in this exercise.

## Supporting Documentation

See [Network Segmentation Validation](validation.md) for
the topology, test procedure, screenshots, and persistence checks.
