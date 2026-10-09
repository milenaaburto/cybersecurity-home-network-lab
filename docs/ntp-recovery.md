# DNS/NTP recovery and boot follow-up

## Latest status — October 9 daytime session

Ubuntu reported synchronized time with NTP active after a normal startup.
The operator confirmed normal starts for both affected VMs without recovery or
GRUB edits following a VirtualBox update. One start per VM is confirmed; startup
stability remains under observation. HTTP and segmentation tests were repeated.
See [final revalidation](final-revalidation-2026-10-09.md).

The sections below preserve the earlier recovery session and its then-pending checks.

## Verification scope — 2026-10-09 UTC

This follow-up records observations from the October 8 evening session (UTC-6).
The original segmentation logs and their timestamp limitations are unchanged.
Values below were transcribed from operator-provided console screenshots; they
are observations at collection time, not continuous monitoring.

## Restricted egress

Ubuntu-Server (192.168.20.10) now has two explicit forwarding exceptions through
enp0s8 -> enp0s3, each with matching POSTROUTING MASQUERADE:

| Destination | Protocol | Purpose |
|---|---|---|
| 8.8.8.8 | UDP/53 | DNS |
| 186.177.18.74 | UDP/123 | NTP |

The seven segmentation rules remain, followed by these two exceptions.
FORWARD remains DROP; INPUT and OUTPUT remain ACCEPT. No general Internet
egress is enabled. TCP DNS and alternate DNS resolvers are not covered.

DNS resolution was observed to succeed after the DNS exception was added.
NTP synchronized after the numeric time source and matching egress rules
were configured. The earlier failed NTP server's behavior was not isolated
with a packet capture; this does not establish a single cause for every timeout.

## Persistent Ubuntu time configuration

File: /etc/systemd/timesyncd.conf.d/zz-lab-ntp-test.conf

```ini
[Time]
NTP=
NTP=186.177.18.74
FallbackNTP=
```

The temporary /run override was removed, and synchronization succeeded after
restarting systemd-timesyncd with the persistent /etc configuration.
The older zz-lab-ntp.conf.disabled file is inactive.
The chosen numeric pool-member address is a lab dependency, not a guaranteed
permanent endpoint. If it changes, both the time configuration and firewall
destination need review. There is no configured fallback.

## Observed results

| System | Time source | Last reported offset | Packet count |
|---|---|---|---|
| Firewall-Router | 200.59.19.5 (2.ubuntu.pool.ntp.org) | -5.585 ms | 5 |
| Ubuntu-Server | 186.177.18.74 | -3.007 ms | 3 |

Both reported synchronized clocks with NTP active. These offsets are relative
to each machine's selected source; they are not a measured inter-host offset.
Ubuntu reached 192.168.20.1 with 3/3 ping replies.

Rules were saved using netfilter-persistent. After the router's subsequent
boot recovery, all nine FORWARD rules and both NAT rules were present.
The [updated export](../firewall-rules/firewall-rules.v4) was generated at
05:00:07 UTC and transcribed from the complete console screenshot.
Ubuntu itself has not yet passed a planned reboot test of this new configuration.
The segmentation traffic tests were not repeated in this follow-up.

## Boot issue remains open

The planned router reboot again resulted in a black screen. A subsequent boot
with quiet/splash temporarily removed in GRUB reached login. That sequence
does not establish that those parameters caused the failure or that removing
them is a permanent fix.

After recovery, all four network interfaces were UP with the expected addresses,
and systemctl --failed showed zero failed units. The boot journal nevertheless
reported RCU stalls, CPU soft lockups, vmwgfx unsupported-hypervisor errors,
and a missing pam_lastlog.so module.

The supplied VirtualBox log showed version 7.2.4 on Windows 11, Hyper-V/NEM
execution, and a timer catch-up warning of approximately 404 seconds.
That warning is not an NTP offset. No single root cause is confirmed.
No host security setting was changed as part of this recovery.

## Remaining work

- Completed in the later session: Ubuntu synchronization after a normal startup.
- Establish reliable VM startup with a documented cause or bounded workaround.
- Maintain the fixed NTP dependency and check time status before future captures.
- Keep historical log timestamps unchanged; current synchronization does not
  retroactively validate old event ordering.

Track progress in [NTP issue #1](https://github.com/milenaaburto/cybersecurity-home-network-lab/issues/1)
and [boot issue #2](https://github.com/milenaaburto/cybersecurity-home-network-lab/issues/2).
