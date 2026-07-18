# Network Indicators of Compromise
### Incident IR-2026-0847 · QuantiaPay

---

## External IP Addresses

| IP Address | Port | Protocol | Role | First Seen | Last Seen |
|------------|------|----------|------|------------|-----------|
| `198.51.100.23` | 443 | TCP/HTTPS | C2 & Exfiltration destination | 03:09:17 | 03:15:30+ |

**Context:**
- Connection originated from `jump-02` (`10.20.9.140`)
- Process responsible: `rclone.exe` (PID 4188)
- Total data exfiltrated: **5.3 GB**
- Connection state at detection: `ESTABLISHED`
- Traffic disguised over port 443 (HTTPS) to evade inspection

---

## Internal Hosts Involved

| Hostname | IP Address | Role in Incident |
|----------|------------|-----------------|
| `build-runner-03` | `10.20.4.77` | Origin of `svc_ci_deploy` credential (legitimate runner — likely token source) |
| `jump-02` | `10.20.9.140` | Compromised administrative host — lateral movement target and exfil source |

---

## Network Traffic Timeline

```
03:02:15  AUTH    svc_ci_deploy login → jump-02 (10.20.9.140)        [ANOMALOUS]
03:09:17  CONN    jump-02 → 198.51.100.23:443  ESTABLISHED           [MALICIOUS]
03:11:48  DATA    jump-02 → 198.51.100.23  vol=1.2 GB                [EXFILTRATION]
03:14:00  ALERT   SIEM P1 fired — anomalous svc account + exfil      [DETECTION]
03:15:30  DATA    jump-02 → 198.51.100.23  vol=3.8 GB cumulative     [ACTIVE]
```

---

## Firewall Rule Applied (Containment)

```
fw add-rule deny out src 10.20.9.140/32 dst 198.51.100.23/32 port 443
```

Scope: surgical `/32` — blocked only the compromised host to the attacker IP.
Production routes remained unaffected.

---

## Recommended Blocks

```
# Block exfiltration destination
198.51.100.23/32  →  DENY ALL (inbound + outbound)

# Alert on any new connection attempts to this IP from any internal host
ALERT  src 10.0.0.0/8  dst 198.51.100.23  any
```

---

*Last updated: 2026-07-18 · Incident IR-2026-0847*
