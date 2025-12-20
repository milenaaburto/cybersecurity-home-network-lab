# Security Policy

## Objective
Protect the internal network by restricting access from untrusted and semi-trusted zones.

## Rules Summary
- Deny all traffic from External to Internal network
- Allow HTTP traffic from External to DMZ
- Deny DMZ-initiated traffic to Internal network
- Allow Internal users to access DMZ services

## Security Rationale
These rules reduce attack surface, prevent lateral movement,
and enforce least-privilege network access.
