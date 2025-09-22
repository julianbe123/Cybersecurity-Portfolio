# Case Study: CoreRepair Malware Investigation on macOS

**Date:** Aug 2025  
**Category:** Digital Forensics & Incident Response  
**Role:** Investigator  

---

## 🧩 1. Initial Discovery
- Found two **unsigned frameworks** in `/Library/Frameworks`:
  - `CoreRepairCore.framework`
  - `CoreRepairKit.framework`
- Contained an XPC service (`CoreRepairCoreXPCService`) pretending to be Apple:
  - `com.apple.CoreRepairCoreXPCService`
- Validation:
  ```bash
  spctl --assess --verbose /Library/Frameworks/CoreRepairCore.framework
  codesign -dv --verbose=4 /Library/Frameworks/CoreRepairCore.framework
  ```
  → Gatekeeper rejected the binary: *“resource envelope is obsolete”*.  

- Strings analysis revealed **hard-coded Apple-like endpoints**:  
  - `https://gs.apple[.]com:443`  
  - `http://gg.apple[.]com/fdrtrustobject`

---

## 📡 2. Behavior Observed
- Logs showed **failed attempts to hook into coreaudiod**:  
  ```
  [com.apple.corerepair:device] Failed to copy key RepairStatus
  ```
- Imported frameworks confirmed **networking + IPC intent**:
  - XPC
  - Mach ports
  - BSD sockets
- Launch attempts blocked by `launchd`:  
  *“Path not allowed in target domain … service did not ship in requestor’s bundle”*  
- No persistence items (`~/Library/LaunchAgents`, `/Library/LaunchDaemons`) tied directly to CoreRepair.

---

## 📦 3. Root Cause – Installation Vector
- Timeline review:
  - **Aug 15** → Third-party AV software installed.  
  - **Aug 17 ~02:44** → CoreRepair frameworks appeared.  
- Evidence:
  ```bash
  ls -l /var/db/receipts | grep com.vendor
  stat /Library/Frameworks/CoreRepairCore.framework
  ```
- `pkgutil --pkgs` showed vendor receipts, but **no CoreRepair entries**.  
- Assessment: **payload was dropped by AV hub postinstall scripts**, not as a declared package.

---

## 🔥 4. Containment & Cleanup
- Quarantined suspicious binaries:
  ```bash
  mkdir -p /Users/Shared/quarantine/CoreRepair
  mv /Library/Frameworks/CoreRepair* /Users/Shared/quarantine/CoreRepair/
  ```
- Removed vendor apps and support files:
  ```bash
  sudo rm -rf /Applications/AVGAntivirus.app
  sudo rm -rf /Applications/AVG\ Secure\ VPN.app
  sudo rm -rf /Library/Application\ Support/AVG*
  ```
- Deleted related LaunchDaemons:
  ```bash
  sudo rm /Library/LaunchDaemons/com.vendor.*
  ```
- Verified cleanup:
  ```bash
  pkgutil --pkgs | grep avg
  # → no results
  ```

---

## 🕵️ 5. Persistence Hijacking (Second Stage)
- Discovered **fake binaries** in `/usr/libexec`:
  - `tmp_cleaner` (SHA256: b59bfb7...e40d)  
  - `locate.updatedb` (SHA256: bbb8530...d464)  
  - `gkreport` (SHA256: 67d87ac...34c7)  
- All unsigned shell/Bash scripts, **timestamped Aug 17 02:44**, matching CoreRepair drop.  
- Scheduled via LaunchDaemons (`com.apple.locate.plist`, `com.apple.gkreport.plist`).  
- Legitimate Apple daemons (`newsyslog`, `systemstats`, `logkextloadsd`) were untouched.  

---

## 🌐 6. Network Evidence
- Captured outbound traffic using `tcpdump`:
  ```bash
  sudo tcpdump -i any -w /var/log/tailgate/midnight.pcap
  ```
- Observed connections at **midnight** to redacted IPs:
  - `203.xx.xx.72:80` (unsecured HTTP)  
  - `185.xx.xx.43`  
  - `17.xx.xx.136`
- Connections were short-lived, likely **beaconing attempts**.  
- WHOIS and ASN lookups suggested **Singapore ISP (SingNet / ASN 3758)**.  
- Did not match known Apple or vendor infrastructure.  

---

## 📌 7. Assessment
- Malware was designed to:
  - Masquerade as Apple frameworks in `/Library/Frameworks`.  
  - Piggyback onto **coreaudiod** and trusted services.  
  - Establish **persistence via fake system daemons** (`tmp_cleaner`, `gkreport`).  
  - Beacon to external IPs over HTTP.  

- **Limitations of Execution**:
  - Gatekeeper blocked unsigned binaries.  
  - Launchd denied service registration.  
  - No evidence of large-scale exfiltration.  

---

## ✅ 8. Lessons Learned
- Always **verify code signatures** with `codesign` & `spctl`.  
- **Audit /usr/libexec and /System/Library/LaunchDaemons** for fake scripts.  
- Use `fs_usage` and `execsnoop` for runtime correlation:
  ```bash
  sudo fs_usage -w -f exec | grep -E "tmp_cleaner|gkreport"
  sudo fs_usage -w -f filesystem | grep -E "tmp_cleaner|gkreport"
  ```
- Harden system with:
  - File Integrity Monitoring (tripwire, osquery).  
  - Logging + IDS rules for Apple-signed path deviations.  

---

📂 **Status:** Frameworks quarantined, fake daemons removed, AV software uninstalled.  
System clean as of Aug 26, 2025.
