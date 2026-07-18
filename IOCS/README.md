# Indicators of Compromise (IOCs)
### Incident IR-2026-0847 · QuantiaPay

> All indicators listed below were extracted from evidence collected during the active investigation.
> These IOCs should be used for threat hunting, detection rule creation, and blocking across security controls.

---

## Files in this folder

| File | Description |
|------|-------------|
| [`network.md`](./network.md) | IP addresses, domains, ports involved in C2 and exfiltration |
| [`accounts.md`](./accounts.md) | Compromised accounts and credential indicators |
| [`host.md`](./host.md) | Processes, persistence mechanisms, and artifacts found on endpoint |
| [`detection-rules.md`](./detection-rules.md) | SIEM / EDR detection rules derived from observed TTPs |

---

## Quick Reference — Critical IOCs

| Type | Value | Severity |
|------|-------|----------|
| IP Address | `198.51.100.23` | 🔴 Critical |
| Account | `svc_ci_deploy` | 🔴 Critical |
| Host | `jump-02 (10.20.9.140)` | 🔴 Critical |
| Process | `rclone.exe` (PID 4188) | 🔴 Critical |
| Persistence | `WindowsUpdateCheck` (Scheduled Task) | 🔴 Critical |
| Persistence | `OneDriveSync` (Registry Run Key) | 🔴 Critical |
| Binary | `C:\Users\Public\svc.exe` | 🔴 Critical |

---

## MITRE ATT&CK Mapping

| Tactic | Technique | ID | Observable |
|--------|-----------|----|------------|
| Initial Access | Valid Accounts: Service Accounts | T1078.003 | `svc_ci_deploy` token exposed in build log |
| Lateral Movement | Remote Services: Windows Remote Management | T1021.006 | WinRM session from build-runner-03 → jump-02 |
| Discovery | Domain Account Discovery | T1087.002 | `net group "Domain Admins" /domain` |
| Discovery | Remote System Discovery | T1018 | LDAP mass query at 03:03:41 |
| Execution | Command and Scripting Interpreter: PowerShell | T1059.001 | `powershell.exe -EncodedCommand <base64>` |
| Exfiltration | Exfiltration Over C2 Channel | T1041 | `rclone.exe` → `198.51.100.23:443` (5.3 GB) |
| Persistence | Scheduled Task/Job | T1053.005 | `WindowsUpdateCheck` task |
| Persistence | Boot or Logon Autostart: Registry Run Keys | T1547.001 | `OneDriveSync` Run key |

---

*Last updated: 2026-07-18 · Analyst: [Name]*
