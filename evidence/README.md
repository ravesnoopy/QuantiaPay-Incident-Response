# Evidence
### Incident IR-2026-0847 · QuantiaPay

> This folder contains all forensic evidence collected during the incident response.
> Evidence was captured in strict volatility order to preserve integrity.
> All artifacts were secured before containment actions were applied.

---

## Chain of Custody

| Field | Value |
|-------|-------|
| Incident ID | IR-2026-0847 |
| Collection Start | 2026-07-18 · 03:12:00 |
| Collected By | SOC Analyst |
| Host | `jump-02` (10.20.9.140) |
| Collection Method | Live forensics — host running, RAM preserved |
| Total Evidence Sources | 5 / 5 captured ✅ |

---

## Files in this folder

| File | Evidence Type | Volatility |
|------|--------------|------------|
| [`01-netstat.md`](./01-netstat.md) | Active network connections at time of capture | 🔴 Highest |
| [`02-arp.md`](./02-arp.md) | ARP table — recent host communication | 🔴 High |
| [`03-ram.md`](./03-ram.md) | RAM image findings — processes, config, malware in memory | 🔴 High |
| [`04-disk.md`](./04-disk.md) | Disk image — non-volatile forensic copy | 🟡 Medium |
| [`05-logs.md`](./05-logs.md) | SIEM and firewall log export | 🟡 Medium |

---

## Collection Order (Volatility Sequence)

```
1. netstat -anob      → Active connections + PIDs       [most ephemeral]
2. arp -a             → ARP cache entries
3. ram-capture        → Full memory image (16 GB)
4. disk-image         → Forensic disk copy
5. export-logs        → SIEM + firewall logs (1.2 GB)   [least ephemeral]
```

> Volatile evidence disappears when the host is powered off or the attacker disconnects.
> This sequence was completed **before** EDR isolation was applied.

---

## Key Findings Summary

| Evidence Source | Critical Finding |
|----------------|-----------------|
| netstat | `rclone.exe` (PID 4188) actively connected to `198.51.100.23:443` |
| ARP | Gateway MAC captured — `10.20.9.1 → aa-bb-cc-11-22-33` |
| RAM | `rclone.exe` confirmed in memory + `temp.conf` containing exfil destination |
| Disk | Forensic copy secured for long-term analysis |
| Logs | 1.2 GB of SIEM and firewall events exported and preserved |

---

*Last updated: 2026-07-18 · Incident IR-2026-0847*
