# pf Firewall Configuration & Monitoring on macOS

**Date:** 2025-09-22  
**Category:** Security Engineering  
**Role:** Engineer

## Objective
Harden macOS network perimeter with `pf`, enable packet logging, and basic alerting.

## Steps (Outline)
1. Create `/etc/pf.anchors/custom.conf` with explicit allowlist rules and default block.
2. Enable logging on suspicious ports; verify with `tcpdump -n -e -ttt -i pflog0`.
3. Add launchd job to watch logs and notify on thresholds (e.g., via `ntfy`).

## Evidence
- `![pf logs screenshot](../images/pf-logs.png)`

## Notes
- Keep public repo redacted; no private IPs or secrets.