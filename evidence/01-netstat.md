# Evidence 01 — Active Network Connections (netstat)
### Incident IR-2026-0847 · QuantiaPay

---

## Collection Details

| Field | Value |
|-------|-------|
| Command | `netstat -anob` |
| Collected | 2026-07-18 · ~03:12:00 |
| Host | `jump-02` (10.20.9.140) |
| Volatility | 🔴 Highest — disappears on disconnect or isolation |
| Collection order | **#1 of 5** |

---

## Raw Output

```
analyst@quantia-ir:~$ netstat -anob

Proto  Local Address          Foreign Address        State    PID   Process
TCP    10.20.9.140:5421       198.51.100.23:443      ESTAB    4188  rclone.exe
```

✓ Active connections captured. Evidence secured.

---

## Analysis

### What this tells us

| Field | Value | Significance |
|-------|-------|--------------|
| Local address | `10.20.9.140:5421` | `jump-02` — the compromised host |
| Foreign address | `198.51.100.23:443` | External attacker-controlled IP, port 443 (HTTPS) |
| State | `ESTABLISHED` | Live connection — exfiltration actively in progress at time of capture |
| PID | `4188` | Directly links the connection to a specific process |
| Process | `rclone.exe` | Cloud sync tool repurposed as exfiltration utility |

### Why rclone?

`rclone` is a legitimate open-source file transfer tool commonly used for cloud storage sync. Attackers abuse it because:
- It uses HTTPS (port 443) — blends with normal web traffic
- It is not flagged by most AV solutions out of the box
- It supports numerous storage backends and can connect to attacker-controlled servers

### Connection to other evidence

- **RAM capture (Evidence 03)** confirmed `rclone.exe` running in memory with `temp.conf` containing `198.51.100.23` as destination
- **Firewall logs (Evidence 05)** show this connection beginning at `03:09:17`
- **SIEM alert** at `03:14:00` correlated this connection with the anomalous `svc_ci_deploy` login

---

## Containment Action Triggered

This finding directly informed the firewall rule applied during containment:

```bash
fw add-rule deny out src 10.20.9.140/32 dst 198.51.100.23/32 port 443
```

---

*Evidence 01 of 5 · Incident IR-2026-0847*
