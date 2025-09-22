# <PROJECT TITLE>

**Date:** <YYYY-MM-DD>  
**Category:** <Forensics | SOC/IR | Threat Intelligence | Security Engineering>  
**Role:** <Analyst | Investigator>  

## Objective
What problem am I solving? What is the hypothesis or question?

## Environment
- OS/Host(s):
- Datasets/Images:
- Tools & versions: <Wireshark x.y, Hydra x.y, Volatility 3.x, etc.>

## MITRE ATT&CK Mapping (if applicable)
| Tactic | Technique | ID |
|---|---|---|
| Credential Access | Brute Force | T1110 |
| Discovery | Account Discovery | T1033 |

## Methodology (Steps)
1. ...
2. ...
3. ...

## Evidence
Insert screenshots with brief captions.
- `![caption](../images/<file>.png)`

## Findings
- What did I observe?
- Indicators (IPs, hashes, URIs)
- Correlations (logs ↔ packets ↔ host events)

## Detection & Response (if applicable)
- Proposed detection rule(s) / filter(s)
- IR steps (containment → eradication → recovery)

## Recommendations
- Hardening / policy
- Monitoring
- Preventive controls

## Lessons Learned
- What would I do differently next time?

## Appendix (Commands/Filters)
```bash
# examples
hydra -l user -P rockyou.txt ssh://10.0.0.5
tcp.port == 22 && tcp.flags.syn == 1
vol -f mem.raw windows.pslist
```