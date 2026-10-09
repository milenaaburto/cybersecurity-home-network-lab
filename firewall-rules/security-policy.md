# Security Policy

## Objective
Protect the internal network by restricting access from untrusted and semi-trusted zones.

## Rules Summary
- Deny all traffic from External to Internal network
- Allow HTTP traffic from External to DMZ
- Deny DMZ-initiated traffic to Internal network
- Allow Internal users to access DMZ services

## Security Rationale
These rules reduce attack surface and opportunities for lateral movement,
and restrict network access to the documented flows.

## VirtualBox Implementation Scope

In this lab, "External" refers to the isolated Kali attacker
network (192.168.10.0/24), not the public Internet.

The current IPv4 forwarding implementation permits new TCP/80
connections to 192.168.20.10 from the attacker and internal networks.
Other DMZ services are not explicitly permitted.

ESTABLISHED and RELATED traffic is accepted before the
source-to-destination restrictions. Deny statements therefore
describe prohibited initiation of connections, while return
traffic for permitted connections is allowed.

The FORWARD policy is DROP. Router INPUT and OUTPUT policies
remain ACCEPT; this validation does not demonstrate router
management-plane hardening.

Segmentation reduces opportunities for lateral movement.
The completed tests establish enforcement only for the tested
paths and protocols, not prevention of every possible attack.

See [validation results](../docs/validation.md) and
[SOC analysis](../docs/soc-analysis.md).

## DNS and NTP egress exceptions

Ubuntu-Server (192.168.20.10) may initiate UDP/53 to 8.8.8.8 and UDP/123
to 186.177.18.74 through enp0s8 -> enp0s3, with matching MASQUERADE rules.
These two exceptions supplement the seven segmentation rules. General Internet
access, TCP DNS, and alternate resolvers are not permitted by these exceptions.
The numeric NTP endpoint requires maintenance if it becomes unavailable.
See [configuration and verification](../docs/ntp-recovery.md).
