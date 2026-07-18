# Host & Endpoint Indicators of Compromise
### Incident IR-2026-0847 · QuantiaPay

---

## Affected Host

| Field | Value |
|-------|-------|
| Hostname | `jump-02` |
| IP Address | `10.20.9.140` |
| Role | Administrative jump host |
| OS | Windows |
| Status | 🔴 Isolated via EDR · Pending rebuild |

---

## Malicious Processes

| Process | PID | Parent | Command Line | Purpose |
|---------|-----|--------|--------------|---------|
| `rclone.exe` | 4188 | — | `rclone.exe` (config: `temp.conf`) | Data exfiltration tool |
| `powershell.exe` | — | `wsmprovhost.exe` | `powershell.exe -EncodedCommand <base64>` | Encoded payload execution via WinRM |
| `net.exe` | — | — | `net group "Domain Admins" /domain` | Privilege/AD enumeration |

### rclone Evidence (from RAM capture)
```
RAM image: jump-02_mem.raw (16 GB)
Process: rclone.exe  PID=4188
Config:  temp.conf  →  destination: 198.51.100.23
```
> `rclone` is a legitimate cloud sync utility. Its presence on an administrative host with a temporary config pointing to an external IP is a strong exfiltration indicator.

---

## Persistence Mechanisms

### 1. Scheduled Task — `WindowsUpdateCheck`

| Field | Value |
|-------|-------|
| Name | `WindowsUpdateCheck` |
| Type | Windows Scheduled Task |
| Trigger | Every 30 minutes |
| Action | `powershell -enc <base64>` |
| Status | ✅ Removed |

> **Note:** This persistence mechanism **reactivated the attacker** during the eradication phase before it was identified and removed. Removal of `OneDriveSync` alone was insufficient — full sweep was required.

---

### 2. Registry Run Key — `OneDriveSync`

| Field | Value |
|-------|-------|
| Name | `OneDriveSync` |
| Type | Registry Run Key (`HKLM\...\Run` or `HKCU\...\Run`) |
| Binary | `C:\Users\Public\svc.exe` |
| Status | ✅ Removed |

> The binary path `C:\Users\Public\` is a common attacker staging location — world-writable, rarely monitored.

---

## Malicious Binary

| File | Path | Status |
|------|------|--------|
| `svc.exe` | `C:\Users\Public\svc.exe` | ✅ Removed with persistence mechanism |

---

## Network Connection on Host (from netstat)

```
Proto  Local Address        Foreign Address       State   PID   Process
TCP    10.20.9.140:5421     198.51.100.23:443     ESTAB   4188  rclone.exe
```

---

## ARP Table (at time of capture)

```
Interface: 10.20.9.140 --- 0x4
  10.20.9.1     aa-bb-cc-11-22-33   dynamic
```

---

## Evidence Collected from Host

| Evidence Type | Command Used | Output |
|---------------|-------------|--------|
| Active connections | `netstat -anob` | `rclone.exe` PID 4188 → 198.51.100.23:443 |
| ARP table | `arp -a` | Gateway MAC captured |
| RAM image | `ram-capture jump-02` | `jump-02_mem.raw` (16 GB) — contains `rclone.exe` + `temp.conf` |
| Disk image | `disk-image jump-02` | Forensic copy — initiated in background |
| Logs | `export-logs siem firewall` | 1.2 GB exported (SIEM + firewall) |

---

## Endpoint Timeline

```
03:02:15  svc_ci_deploy authenticates to jump-02 via WinRM
03:03:41  LDAP mass query — Active Directory host enumeration
03:05:08  powershell.exe -EncodedCommand launched (parent: wsmprovhost.exe)
03:06:33  net.exe group "Domain Admins" /domain
03:09:17  rclone.exe opens connection → 198.51.100.23:443
03:11:48  DATA_OUT: 1.2 GB exfiltrated
03:15:30  DATA_OUT: 3.8 GB cumulative — still active
03:14:00  SIEM P1 alert fires
```

---

## Recovery Decision

**Action:** Rebuild `jump-02` from trusted image
**Justification:** Host compromised by attacker with valid credentials and active persistence mechanisms. Rebuilding provides higher integrity guarantee than cleaning a previously compromised system.
**RTO:** 2 hours

---

*Last updated: 2026-07-18 · Incident IR-2026-0847*
