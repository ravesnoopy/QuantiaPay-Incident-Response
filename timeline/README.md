# Attack Timeline
### Incident IR-2026-0847 · QuantiaPay

> Complete chronological reconstruction of the attack from first legitimate activity
> through containment and eradication. Built from SIEM, firewall, EDR, and authentication logs.

---

## Files in this folder

| File | Description |
|------|-------------|
| [`attack-timeline.md`](./attack-timeline.md) | Full event-by-event timeline with context and MITRE mapping |
| [`response-timeline.md`](./response-timeline.md) | Analyst response actions from detection to verification |

---

## Attack at a Glance

```
02:47  ─── Legitimate CI/CD activity begins
02:51  ─── Pipeline #4471 deployed to production
           │
03:02  ─── ⚠ BREAKPOINT — svc_ci_deploy logs into jump-02 [ANOMALOUS]
03:03  ─── AD enumeration (LDAP mass query)
03:05  ─── Encoded PowerShell via WinRM
03:06  ─── net group "Domain Admins" /domain
           │
03:09  ─── 🔴 External connection to 198.51.100.23:443
03:11  ─── 🔴 Exfiltration begins — 1.2 GB
03:14  ─── 🚨 SIEM P1 alert fires
03:15  ─── 🔴 Exfiltration reaches 3.8 GB — ACTIVE
           │
03:12  ─── ✅ EDR isolation applied (jump-02)
03:12  ─── ✅ Firewall rule /32 blocks exfil
           └── Total exfiltrated: 5.3 GB
```

---

## Key Timestamps

| Timestamp | Event |
|-----------|-------|
| `02:47:03` | First legitimate login — baseline reference |
| `03:02:15` | **Breakpoint** — attack begins |
| `03:09:17` | Exfiltration channel opened |
| `03:14:00` | SIEM P1 alert — analyst notified |
| `03:12:39` | EDR isolation applied |
| `03:12:57` | Firewall rule blocks exfiltration |

**Detection-to-containment:** ~12 minutes from breakpoint · ~1 minute from P1 alert

---

*Last updated: 2026-07-18 · Incident IR-2026-0847*
