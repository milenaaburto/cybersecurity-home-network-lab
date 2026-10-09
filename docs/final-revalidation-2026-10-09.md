# Final scoped revalidation — 2026-10-09

The IPv4 segmentation and manual log-analysis objectives are complete for the tested paths. Operational follow-up remains open. This is not a SIEM deployment or a claim of permanently reliable VM startup.

## Evidence basis

Results come from operator-provided console screenshots and explicit operator confirmations. They are point-in-time observations. The original raw log remains on the router; a [MAC-redacted copy](../logs/tcp-deny-revalidation-2026-10-09-sanitized.log) is now published from the actual transferred file.

## Results

| Test | Observed result |
|---|---|
| Ubuntu Apache and local HTTP | active; HTTP 200 |
| Kali and Fedora to DMZ HTTP | HTTP 200 |
| Router to Fedora ICMP control | 3/3 replies |
| Kali to Fedora ICMP | 0/3 replies; matching DROP counter 0 -> 3 |
| Ubuntu to Fedora ICMP | 0/3 replies; matching DROP counter 0 -> 3 |
| Router to temporary Fedora TCP service | HTTP 200 |
| Kali to temporary Fedora TCP service | timeout; matching DROP counter 3 -> 8 |
| Ubuntu to temporary Fedora TCP service | timeout; matching DROP counter 3 -> 8 |
| TCP logging counter | 6 -> 16 |
| Temporary service cleanup | directory removed; no listening socket remained |
| Ubuntu NTP after normal startup | clock synchronized: yes; NTP active |
| Failed systemd units | zero shown on Router and Ubuntu |

Nine FORWARD rules were visible in the counter checks. NAT and IPv4 forwarding persistence remain supported by the earlier recovery record; they were not separately re-exported in this session.

## Log interpretation

The operator exported ten TCP records and verified the count. Console inspection showed five SYN records from Kali and five from Ubuntu to the temporary internal service. Repetition of the same endpoints and ports is consistent with retries; sequence numbers were not recorded, so packet-level retransmission identification is not claimed.

LOG records a packet and does not itself drop it. The enforcement conclusion combines the successful service control, client timeouts, matching DROP counter increases and corresponding logs. Ten records do not imply ten attacks.

The original export is retained on the router. The operator supplied and authorized publication of a separate copy with all ten MAC values replaced by [REDACTED_MAC]. It preserves the remaining recorded fields and is not a screenshot transcription. See the [evidence inventory](../logs/README.md) for provenance. The existing published extracts remain unchanged. No precise inter-host clock offset or all-host synchronization was established; historical timestamp limitations remain applicable.

## Startup and completion scope

The operator confirmed that VirtualBox was updated and that both affected VMs started normally without recovery or GRUB edits. The exact installed version/build was not independently verified. One normal startup per affected VM is confirmed; the existing three-start stability criterion has not yet been met.

Ubuntu synchronization after a normal startup is verified. The old NTP endpoint failure remains unexplained, and the configured replacement has no fallback.

The scoped lab is complete with documented operational limitations. Remaining maintenance consists of startup observation and NTP dependency maintenance. IPv6 validation, router INPUT/OUTPUT hardening, centralized SIEM ingestion and automated alerting are outside this version.

See [NTP recovery](ntp-recovery.md), [NTP issue #1](https://github.com/milenaaburto/cybersecurity-home-network-lab/issues/1), and [startup issue #2](https://github.com/milenaaburto/cybersecurity-home-network-lab/issues/2).
