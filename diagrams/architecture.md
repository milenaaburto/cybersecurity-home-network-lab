# Network Architecture

The VirtualBox lab uses three routed IPv4 zones: an attacker test network, an Apache DMZ, and an internal workstation network. An Ubuntu firewall-router controls forwarded traffic. The Wazuh manager shares InternalNet with Fedora and has a separate host-only interface for dashboard access from Windows.

See the [implemented topology](virtualbox-topology.md), [forwarding policy](../firewall-rules/security-policy.md), and [Wazuh investigation](../docs/siem-lab.md).
