# Lab Evidence Exports

These are original text extracts transferred from the lab VMs through a
VirtualBox shared folder and published without editing their contents.
They record authorized connectivity tests, not a real incident.

| File | Source | Records | Test |
|---|---|---|---|
| [internal-deny-evidence.log](internal-deny-evidence.log) | Firewall-Router kernel journal | 9 | Kali ICMP echo requests to Fedora |
| [tcp-deny-evidence.log](tcp-deny-evidence.log) | Firewall-Router kernel journal | 10 | Five SYN records each from Kali and DMZ to Fedora TCP/8080 |
| [apache-kali-evidence.log](apache-kali-evidence.log) | Ubuntu-Server /var/log/apache2/access.log | 4 | Kali HEAD / requests with HTTP 200 |

## Provenance and interpretation

The firewall extracts were exported to /home/netadmin/ on Firewall-Router.
The Apache extract was exported to /home/webadmin/ on Ubuntu-Server.
These are selected extracts, not complete journals or complete access logs.
The ICMP extract contains Kali traffic only; the separate DMZ ICMP test
is supported by the validation evidence rather than this extract.

Firewall records show 2026-10-08 03:09:01–03:09:17 +00:00 for ICMP
and 03:56:54–03:58:02 +00:00 for TCP. Apache records show
08/Oct/2026 at 00:05:34, 02:31:32, 03:08:41 and 17:37:42 +0000.

Timestamps are preserved as recorded. Ubuntu synchronization was unresolved
when the original evidence was collected; these timestamps must not establish
precise cross-host ordering. Synchronization was subsequently observed, including
a normal Ubuntu startup; this does not retroactively validate historical clocks.
Publication and export times are distinct from event timestamps.

Repeated SYN records include retries and do not mean ten separate attacks.
The LAB-INTERNAL-DENY prefix marks a LOG rule, not a DROP verdict.
Enforcement conclusions also rely on the service availability control,
matching DROP counters and client results. Logging is rate-limited.

The files were reviewed before publication: they contain lab private IPv4
addresses, virtual-interface MAC addresses, host identifiers and test metadata.
No credentials or tokens were found in these extracts.

## Related evidence

- [Validation and screenshots](../docs/validation.md)
- [SOC analysis](../docs/soc-analysis.md)
- [Exported IPv4 firewall rules](../firewall-rules/firewall-rules.v4)

The rules export is a later configuration snapshot, not a packet log.
Its recorded counters are not the original test counter measurements.
The current ruleset was transcribed from a complete console screenshot during
the DNS/NTP follow-up; it is not a byte-verified transfer of the original export.
This provenance differs from the three original log extracts listed above.

## October 9 revalidation

A new ten-record TCP extract was saved on Firewall-Router and inspected through
console screenshots. It has not yet been published here; publication approval
is pending. See the [final revalidation record](../docs/final-revalidation-2026-10-09.md)
for results, counter changes, evidence scope and operational follow-up.
