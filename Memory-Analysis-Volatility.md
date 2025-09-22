# Memory Analysis with Volatility

**Date:** 2025-09-22  
**Category:** Forensics  
**Role:** Investigator

## Objective
Analyze a captured memory image to identify suspicious processes, persistence, and network artifacts.

## Environment
- Memory image: `mem.raw`
- Tool: Volatility 3.x

## Methodology
1. Process listing & anomalies (`windows.pslist`, `windows.pstree`).
2. Persistence checks (`windows.registry.printkey`, `windows.services`).
3. Network artifacts (`windows.netscan`).

## Evidence
- `![Volatility pstree](../images/vol-pstree.png)`

## Findings
- <Fill observations>

## Recommendations
- Capture triage playbook; baseline clean system; add YARA rules for memory scans.

## Appendix
```bash
vol -f mem.raw windows.pslist
vol -f mem.raw windows.pstree
vol -f mem.raw windows.netscan
```