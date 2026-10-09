# Reproduce the Segmentation Tests

This guide repeats the documented tests on a prepared lab. It is not an
automated installation guide or a claim that a fresh rebuild has been tested.
New runs produce new timestamps and counters; preserve the published evidence.

## Prerequisites

- Four running VMs with the [documented attachments and addresses](../diagrams/virtualbox-topology.md).
- Apache installed on Ubuntu-Server; curl on Kali, Fedora and Firewall-Router.
- Python 3 on Fedora for the temporary control service.
- iptables and kernel-journal access on Firewall-Router.
- IPv4 forwarding enabled and the nine documented FORWARD rules active.
- No additional host firewall restriction preventing the router-to-Fedora TCP/8080 control.

The [exported ruleset](../firewall-rules/firewall-rules.v4) is an IPv4
iptables-save snapshot. It is not an executable shell script and does not
configure IP addresses, routes, IPv4 forwarding, services or IPv6.
Review interface names and existing rules before using it on a rebuilt,
isolated lab router. Do not replace the current VM configuration just to run these tests.

Install prerequisites before applying restricted egress. The final forwarding
policy does not permit general Internet access from the lab hosts.

## 1. Check the baseline

On each VM:

```bash
ip -br -4 addr
ip -4 route
```

On Firewall-Router:

```bash
sysctl net.ipv4.ip_forward
sudo iptables -S FORWARD
sudo iptables -t nat -S
sudo iptables -vnL FORWARD --line-numbers
```

Expected: forwarding value 1, FORWARD policy DROP, nine rules matching
the export and two scoped DNS/NTP MASQUERADE rules. Record counters before each test; compare
their increases rather than expecting the historical absolute values.

On Ubuntu-Server:

```bash
systemctl is-active apache2
curl --max-time 5 -I http://127.0.0.1/
```

Expected: active and HTTP 200. Resolve failed prerequisites before interpreting later failures.

## 2. Test permitted HTTP

Run on Kali, then Fedora:

```bash
curl --max-time 5 -I http://192.168.20.10/
```

Expected: HTTP 200. Inspect the corresponding source-specific HTTP
allow rule and the ESTABLISHED/RELATED counter on Firewall-Router.

On Ubuntu-Server:

```bash
sudo tail -n 10 /var/log/apache2/access.log
```

Find the new source address, HEAD request and 200 response.
Do not use unsynchronized timestamps alone to correlate hosts.

## 3. Test restricted ICMP

First, on Firewall-Router, verify the control:

```bash
ping -c 3 -W 2 192.168.30.10
```

Expected: replies. Then run the same command on Kali and Ubuntu-Server.
Expected: no replies and an increase in the matching source-to-internal
DROP counter on the router.

## 4. Test restricted TCP against a known available service

On Fedora, start a server serving an empty temporary directory:

```bash
lab_test_dir=$(mktemp -d)
python3 -m http.server 8080 --bind 192.168.30.10 --directory "$lab_test_dir"
```

Keep this terminal open. From Firewall-Router:

```bash
curl --max-time 5 -I http://192.168.30.10:8080/
```

Expected: HTTP 200. If the control fails, do not interpret other
timeouts as proof of segmentation.

Run the same curl command from Kali and Ubuntu-Server.
Expected: timeout, matching SYN log entries and increasing DROP counters.

On Firewall-Router:

```bash
sudo iptables -vnL FORWARD --line-numbers
sudo journalctl -k -n 100 --no-pager | grep 'LAB-INTERNAL-DENY'
```

Inspect SRC, DST, IN, OUT, PROTO, SPT, DPT and SYN. The two tested
sources are 192.168.10.10 and 192.168.20.10; destination is
192.168.30.10:8080. Retries can generate multiple records per attempt.
The LOG rule is rate-limited, so not every dropped packet must appear.
LOG itself is not a DROP verdict.

On Fedora, press Ctrl+C in the server terminal, then:

```bash
rmdir "$lab_test_dir"
ss -lnt 'sport = :8080'
```

Expected: no listening socket on port 8080.

## 5. Persistence and evidence

Persistence was already verified in the [original validation](validation.md).
On a fresh rebuild, save the intended rules using the installed persistence
mechanism and verify after a planned clean reboot that all nine FORWARD rules, both NAT rules and
IPv4 forwarding return. Do not infer persistence solely from a runtime export.
The existing VMs have intermittent boot problems; repeated reboots are not
needed to rerun the connectivity tests.

Keep new observations separately from the [published original extracts](../logs/README.md).
Record commands, source VM, result and counter changes, including failed controls.
The temporary TCP server must be stopped when finished.

## Interpretation limits

These tests validate specified IPv4 paths and ports. They do not demonstrate
router hardening, IPv6 enforcement, absence of application vulnerabilities
or a complete SOC detection pipeline. Router and Ubuntu now report synchronized
clocks, but check all participating hosts before new captures. Historical logs
retain their time limitations. Ubuntu synchronization after a normal startup
was subsequently verified; see [final revalidation](final-revalidation-2026-10-09.md).
See [DNS/NTP recovery](ntp-recovery.md).
