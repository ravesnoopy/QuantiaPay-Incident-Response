# Attack Timeline — Full Reconstruction
### Incident IR-2026-0847 · QuantiaPay

---

## Phase 0 — Legitimate Activity (Pre-Attack Baseline)

> These events are **not part of the attack**. They establish the normal behavior of `svc_ci_deploy`
> immediately before the compromise. Confusing them with malicious activity is a documented analyst error.

---

### 02:47:03 — Normal Login
**Severity:** ℹ️ INFO

| Field | Value |
|-------|-------|
| Event | `AUTH INFO — LOGIN_OK svc_ci_deploy` |
| Source | `build-runner-03` (10.20.4.77) |
| Raw log | `LOGIN_OK svc_ci_deploy build-runner-03 (10.20.4.77)` |
| MITRE | — (legitimate activity) |

The service account `svc_ci_deploy` authenticated from its expected CI/CD runner host. This is normal baseline behavior — the account operating exactly as designed.

---

### 02:51:22 — Production Deployment
**Severity:** ℹ️ INFO

| Field | Value |
|-------|-------|
| Event | `CI/CD INFO — Pipeline #4471 deploy→prod` |
| Commit | `a3f9c2` |
| Approved by | Human reviewer |
| Raw log | `PIPELINE_RUN #4471 deploy→prod commit=a3f9c2 approved_by=human` |
| MITRE | — (legitimate activity) |

An approved commit was deployed to production by the CI/CD pipeline. Human-approved, expected, and consistent with normal operations.

> ⚠️ The attacker's token was likely harvested from this pipeline's build log output.

---

## Phase 1 — Initial Access & Lateral Movement

---

### 03:02:15 — ⚠️ Suspicious Login `[BREAKPOINT — e3]`
**Severity:** 🔴 HIGH

| Field | Value |
|-------|-------|
| Event | `AUTH HIGH — LOGIN_OK svc_ci_deploy` |
| Source | `jump-02` (10.20.9.140) |
| Raw log | `LOGIN_OK svc_ci_deploy jump-02 (10.20.9.140)` |
| MITRE | T1078.003 — Valid Accounts: Service Accounts |
| MITRE | T1021.006 — Remote Services: Windows Remote Management |

**This is where the attack begins.**

The same service account that ran the pipeline at 02:47 is now logging in from `jump-02`, an administrative host with no prior association to `svc_ci_deploy`. Three red flags simultaneously:

- ❌ Wrong source host — `jump-02` is not a CI/CD runner
- ❌ Wrong time — no scheduled pipeline runs at 03:02 AM
- ❌ Wrong behavior — service accounts do not initiate interactive WinRM sessions

The attacker used the token exposed in the 02:51 build log to authenticate as the service account and pivot to the administrative network.

---

### 03:03:41 — Active Directory Enumeration
**Severity:** 🔴 HIGH

| Field | Value |
|-------|-------|
| Event | `AD HIGH — LDAP mass query` |
| Host | `jump-02` (10.20.9.140) |
| Raw log | Mass LDAP query — infrastructure enumeration |
| MITRE | T1018 — Remote System Discovery |
| MITRE | T1087.002 — Account Discovery: Domain Account |

Immediately after gaining access, the attacker performed a mass LDAP query against Active Directory to enumerate hosts, users, and infrastructure. A service account performing bulk AD queries is a strong post-exploitation signal.

---

## Phase 2 — Execution & Privilege Discovery

---

### 03:05:08 — Encoded PowerShell Execution
**Severity:** 🔴 HIGH

| Field | Value |
|-------|-------|
| Event | `ENDPOINT HIGH — powershell -EncodedCommand (WinRM)` |
| Host | `jump-02` (10.20.9.140) |
| Process | `powershell.exe -EncodedCommand <base64>` |
| Parent | `wsmprovhost.exe` |
| Raw log | `powershell.exe -EncodedCommand <base64> parent=wsmprovhost.exe` |
| MITRE | T1059.001 — Command and Scripting Interpreter: PowerShell |
| MITRE | T1027 — Obfuscated Files or Information |

An encoded (base64) PowerShell command was executed via a remote WinRM session. The parent process `wsmprovhost.exe` confirms this was delivered over Windows Remote Management.

Base64 encoding is used to:
- Bypass command-line logging that looks for plaintext keywords
- Evade simple signature-based detection
- Deliver a multi-stage payload without writing a script to disk

---

### 03:06:33 — Privilege Enumeration
**Severity:** 🔴 HIGH

| Field | Value |
|-------|-------|
| Event | `ENDPOINT HIGH — net group "Domain Admins" /domain` |
| Host | `jump-02` (10.20.9.140) |
| Command | `net.exe group "Domain Admins" /domain` |
| MITRE | T1087.002 — Account Discovery: Domain Account |

The attacker queried which accounts hold Domain Admin privileges. This is a standard post-exploitation step to identify high-value targets for privilege escalation or to understand what the compromised account can already access.

---

## Phase 3 — Exfiltration

---

### 03:09:17 — External Connection Established
**Severity:** 🔴 CRITICAL

| Field | Value |
|-------|-------|
| Event | `NETWORK CRIT — connection to 198.51.100.23:443` |
| Host | `jump-02` (10.20.9.140) |
| Destination | `198.51.100.23:443` |
| Process | `rclone.exe` (PID 4188) |
| Raw log | `CONN_OUT jump-02 → 198.51.100.23:443 ESTABLISHED` |
| MITRE | T1041 — Exfiltration Over C2 Channel |

`rclone.exe` opened an outbound HTTPS connection to the attacker-controlled server. Port 443 was chosen deliberately to blend with normal encrypted web traffic and avoid triggering port-based firewall rules.

---

### 03:11:48 — Data Exfiltration Begins
**Severity:** 🔴 CRITICAL

| Field | Value |
|-------|-------|
| Event | `NETWORK CRIT — DATA_OUT 1.2 GB` |
| Host | `jump-02` (10.20.9.140) |
| Destination | `198.51.100.23` |
| Volume | 1.2 GB |
| Raw log | `DATA_OUT jump-02 → 198.51.100.23 vol=1.2GB` |
| MITRE | T1041 — Exfiltration Over C2 Channel |

Active data exfiltration confirmed. `rclone.exe` is transferring data to the external server at significant volume. The source data likely originates from financial infrastructure accessible through `jump-02`.

---

### 03:14:00 — 🚨 SIEM P1 Alert Fires
**Severity:** 🚨 CRITICAL

| Field | Value |
|-------|-------|
| Event | `SIEM CRIT — ALERTA P1` |
| Rule | `anomalous svc account + external exfil` |
| Raw log | `SIEM_ALERT P1 correlation_rule="anomalous svc account + external exfil"` |

The SIEM correlated the anomalous `svc_ci_deploy` login with the active external data transfer and triggered a Priority 1 alert. This is the moment the SOC analyst was notified and the response began.

**Time from breakpoint to detection: 12 minutes**

---

### 03:15:30 — Exfiltration Continues
**Severity:** 🔴 CRITICAL

| Field | Value |
|-------|-------|
| Event | `NETWORK CRIT — DATA_OUT 3.8 GB cumulative · ACTIVE` |
| Host | `jump-02` (10.20.9.140) |
| Volume | 3.8 GB (cumulative) |
| Status | Active at time of log entry |
| Raw log | `DATA_OUT jump-02 → 198.51.100.23 vol=3.8GB cumulative ACTIVE` |
| MITRE | T1041 — Exfiltration Over C2 Channel |

Transfer still active. Containment was applied shortly after, cutting the total exfiltrated volume to **5.3 GB** before the connection was severed.

---

## MITRE ATT&CK Summary

| # | Tactic | Technique | ID | Event |
|---|--------|-----------|-----|-------|
| 1 | Initial Access | Valid Accounts: Service Accounts | T1078.003 | Token stolen from build log |
| 2 | Lateral Movement | Remote Services: WinRM | T1021.006 | Login to jump-02 at 03:02 |
| 3 | Discovery | Remote System Discovery | T1018 | LDAP mass query at 03:03 |
| 4 | Discovery | Domain Account Discovery | T1087.002 | net group at 03:06 |
| 5 | Execution | PowerShell | T1059.001 | Encoded command at 03:05 |
| 6 | Defense Evasion | Obfuscated Files or Information | T1027 | Base64 encoding |
| 7 | Exfiltration | Exfiltration Over C2 Channel | T1041 | rclone.exe → 198.51.100.23 |
| 8 | Persistence | Scheduled Task | T1053.005 | WindowsUpdateCheck |
| 9 | Persistence | Registry Run Keys | T1547.001 | OneDriveSync |

---

*Last updated: 2026-07-18 · Incident IR-2026-0847*
