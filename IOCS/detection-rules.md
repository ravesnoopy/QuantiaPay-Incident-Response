# Detection Rules
### Incident IR-2026-0847 · QuantiaPay

> Rules derived from TTPs observed during this incident.
> Implement in your SIEM / EDR to detect similar activity or re-infection.

---

## Rule 1 — Service Account Login from Non-Baseline Host

**Detects:** Lateral movement using stolen service account credentials

```
# SIEM Pseudocode
ALERT when:
  event_type = "authentication_success"
  AND account = "svc_ci_deploy"
  AND src_host NOT IN ["build-runner-03"]

Severity: HIGH
MITRE: T1078.003 — Valid Accounts: Service Accounts
```

---

## Rule 2 — PowerShell Encoded Command via WinRM

**Detects:** Remote encoded PowerShell execution (common post-exploitation pattern)

```
# SIEM / EDR Pseudocode
ALERT when:
  process_name = "powershell.exe"
  AND command_line CONTAINS "-EncodedCommand"
  AND parent_process = "wsmprovhost.exe"

Severity: HIGH
MITRE: T1059.001 — Command and Scripting Interpreter: PowerShell
       T1021.006 — Remote Services: WinRM
```

---

## Rule 3 — Domain Admin Enumeration

**Detects:** Active Directory privilege discovery

```
# SIEM / EDR Pseudocode
ALERT when:
  process_name = "net.exe"
  AND command_line CONTAINS "Domain Admins"
  AND command_line CONTAINS "/domain"

Severity: MEDIUM
MITRE: T1087.002 — Account Discovery: Domain Account
```

---

## Rule 4 — rclone.exe on Non-Approved Host

**Detects:** Use of rclone as an exfiltration tool

```
# EDR Pseudocode
ALERT when:
  process_name = "rclone.exe"
  AND host NOT IN [approved_cloud_sync_hosts]

Severity: CRITICAL
MITRE: T1041 — Exfiltration Over C2 Channel
```

---

## Rule 5 — Large Outbound Data Transfer

**Detects:** Data exfiltration by volume threshold

```
# Firewall / NDR Pseudocode
ALERT when:
  direction = "outbound"
  AND dst_ip NOT IN [approved_external_ips]
  AND bytes_out > 500_000_000   # 500 MB threshold

Severity: CRITICAL
MITRE: T1041 — Exfiltration Over C2 Channel
```

---

## Rule 6 — Outbound Connection to Known Bad IP

**Detects:** Any communication with the attacker's C2 infrastructure

```
# Firewall / SIEM Pseudocode
ALERT when:
  dst_ip = "198.51.100.23"

Severity: CRITICAL
Action: BLOCK + ALERT
MITRE: T1041 — Exfiltration Over C2 Channel
```

---

## Rule 7 — Executable Dropped in Public User Directory

**Detects:** Attacker staging binaries in world-writable paths

```
# EDR Pseudocode
ALERT when:
  event_type = "file_creation"
  AND file_path CONTAINS "C:\Users\Public\"
  AND file_extension IN [".exe", ".dll", ".ps1", ".bat"]

Severity: HIGH
MITRE: T1547.001 — Boot or Logon Autostart: Registry Run Keys
```

---

## Rule 8 — New Scheduled Task with Encoded PowerShell

**Detects:** Persistence via scheduled task executing obfuscated code

```
# EDR / Windows Event Log (Event ID 4698)
ALERT when:
  event_id = 4698        # Scheduled task created
  AND task_action CONTAINS "powershell"
  AND task_action CONTAINS "-enc"

Severity: HIGH
MITRE: T1053.005 — Scheduled Task/Job: Scheduled Task
```

---

## Monitoring Rule Active (Post-Incident)

The following composite rule was applied after containment and remains active:

```
MONITOR:
  account   = svc_ci_deploy
  OR dst_ip = 198.51.100.23
  OR process = rclone.exe

Action: ALERT SOC immediately
```

---

*Last updated: 2026-07-18 · Incident IR-2026-0847*
