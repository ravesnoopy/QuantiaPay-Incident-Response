# Evidence 05 — SIEM & Firewall Log Export
### Incident IR-2026-0847 · QuantiaPay

---

## Collection Details

| Field | Value |
|-------|-------|
| Command | `export-logs siem firewall` |
| Export size | 1.2 GB |
| Collected | 2026-07-18 · ~03:13:00 |
| Sources | SIEM · Firewall |
| Volatility | 🟡 Medium — at risk of rotation or overwrite |
| Collection order | **#5 of 5** |

---

## Raw Output

```
analyst@quantia-ir:~$ export-logs siem firewall

✓ Logs exported: siem, firewall (1.2 GB).
```

---

## Log Sources

### SIEM Logs

The SIEM aggregated events from multiple sources during the incident window. Key event categories captured:

| Event Category | Source | Notable Events |
|---------------|--------|----------------|
| Authentication | Windows Security Log | `svc_ci_deploy` login events at 02:47, 03:02 |
| Endpoint activity | Sysmon / EDR | PowerShell execution, `net.exe`, `rclone.exe` spawn |
| Network | Firewall / NDR | Outbound connections, DATA_OUT volume alerts |
| AD / LDAP | Domain Controller | Mass LDAP query at 03:03:41 |
| SIEM correlation | Internal | P1 alert fired at 03:14:00 |

### Firewall Logs

Firewall logs capture full egress traffic for the incident window:

| Timestamp | Event | Volume |
|-----------|-------|--------|
| 03:09:17 | `CONN_OUT jump-02 → 198.51.100.23:443 ESTABLISHED` | — |
| 03:11:48 | `DATA_OUT jump-02 → 198.51.100.23` | 1.2 GB |
| 03:15:30 | `DATA_OUT jump-02 → 198.51.100.23 cumulative ACTIVE` | 3.8 GB |
| 03:12:57 | Firewall rule applied — connection blocked | — |

---

## Full Incident Event Log (from SIEM export)

```
02:47:03  AUTH INFO    LOGIN_OK svc_ci_deploy ← build-runner-03 (10.20.4.77)
02:51:22  CI/CD INFO   Pipeline #4471 deploy→prod  commit=a3f9c2  approved_by=human
03:02:15  AUTH HIGH    LOGIN_OK svc_ci_deploy ← jump-02 (10.20.9.140)          ← BREAKPOINT
03:03:41  AD   HIGH    LDAP mass query — infrastructure enumeration
03:05:08  EDR  HIGH    powershell.exe -EncodedCommand  parent=wsmprovhost.exe
03:06:33  EDR  HIGH    net.exe group "Domain Admins" /domain
03:09:17  NET  CRIT    CONN_OUT jump-02 → 198.51.100.23:443 ESTABLISHED
03:11:48  NET  CRIT    DATA_OUT jump-02 → 198.51.100.23  vol=1.2 GB
03:14:00  SIEM CRIT    P1 ALERT — anomalous svc account + external exfil
03:15:30  NET  CRIT    DATA_OUT jump-02 → 198.51.100.23  vol=3.8 GB  ACTIVE
```

---

## SIEM P1 Alert — Correlation Rule

The alert that initiated the response was triggered by a correlation rule combining multiple signals:

```
SIEM_ALERT P1
Rule: "anomalous svc account + external exfil"

Correlated signals:
  [1] svc_ci_deploy login from non-baseline host (jump-02)
  [2] Outbound connection to unknown external IP
  [3] Large data volume exiting the network

Fired at: 03:14:00
```

> This rule did not exist before the incident. It was added post-containment as part of the monitoring enhancement. The initial detection relied on volume thresholds, not behavioral correlation.

---

## Why Log Export Was Collected Last

Logs are the least volatile of the five evidence sources:
- They are written to persistent storage
- They are retained on a rolling window (risk: rotation after days/weeks)
- They can be collected while other more ephemeral evidence is captured first

However, they must be collected before log rotation policies overwrite them. Exporting during the active response phase — before any system changes — ensures logs reflect the pre-containment state.

---

*Evidence 05 of 5 · Incident IR-2026-0847*
