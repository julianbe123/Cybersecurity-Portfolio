# Brute Force Detection (Hydra + Wireshark) & IR Plan

**Date:** 2025-09-22  
**Category:** SOC/Incident Response  
**Role:** Analyst

## Objective
Simulate SSH brute force attempts with **Hydra** and detect the pattern using **Wireshark** and **Windows Event Logs**, then propose an **Incident Response Plan**.

## Environment
- Test VM(s): Kali (attacker), Ubuntu/Windows Server (victim)
- Tools: Hydra, Wireshark, Event Viewer / sysmon (optional)

## Methodology (Steps)
1. Launch Hydra against SSH on target VM. Example:  
   ```bash
   hydra -l julian -P rockyou.txt ssh://192.168.56.10 -t 4 -V
   ```
2. Capture network traffic on the victim or a span port. Filter in Wireshark:  
   ```
   tcp.port == 22
   ```
3. Note repeated failed login attempts (frequent `SYN` and `ACK` patterns; possible `RST`s).  
4. On Windows targets, correlate **Event ID 4625** (failed logon) timestamps with pcap times.  
5. Extract indicators (attacker IP, usernames attempted).

## Evidence
- `![SSH attempts in Wireshark](../images/wireshark-bruteforce.png)`
- `![Windows Event ID 4625 correlation](../images/event-4625.png)`

## Findings
- High-frequency authentication attempts from `ATTACKER_IP` to port 22 with randomized credentials.
- Correlated 4625 failures at matching timestamps.

## Detection & Response
**Sigma-style idea:** Alert when `>N` failed logons from a single IP within `M` minutes.  
**IR Plan (quick):**
- **Detect:** monitor 4625 spikes, abnormal SSH traffic.
- **Contain:** block source IP, rate-limit/geo-fence SSH, disable targeted accounts.
- **Eradicate:** enforce strong passwords & account lockout; restrict SSH to VPN.
- **Recover:** reset credentials; patch; re-enable services with hardened configs.
- **Lessons:** centralize logs; enable MFA; deploy IDS/IPS for brute force patterns.

## Recommendations
- Enforce **MFA**, **Fail2ban/sshguard**, **account lockout**, and **VPN-only** SSH access.
- Add a dashboard showing failed vs successful auth over time.

## Appendix (Commands/Filters)
```bash
# Fail2ban quick check
sudo fail2ban-client status sshd
# Wireshark display filters
tcp.port == 22
tcp.stream eq 5
```