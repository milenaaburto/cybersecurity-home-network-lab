# ACL Configuration

## PROTECT_INTERNAL
Blocks traffic from External and DMZ networks to the Internal LAN.

## EXTERNAL_TO_DMZ
Allows HTTP traffic from External network to the DMZ server.
Denies all other External-to-DMZ traffic.

## Validation
- External access to Internal: Blocked
- External HTTP to DMZ: Allowed
- DMZ lateral movement: Blocked
