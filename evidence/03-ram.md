# Evidence 03 — RAM Image
### Incident IR-2026-0847 · QuantiaPay

---

## Collection Details

| Field | Value |
|-------|-------|
| Command | `ram-capture jump-02` |
| Output file | `jump-02_mem.raw` |
| Image size | 16 GB |
| Collected | 2026-07-18 · ~03:12:10 |
| Host | `jump-02` (10.20.9.140) |
| Volatility | 🔴 High — destroyed permanently on shutdown or reboot |
| Collection order | **#3 of 5** |

---

## Raw Output

```
analyst@quantia-ir:~$ ram-capture jump-02

Capturing [████████████████████] 100%

✓ RAM image saved: jump-02_mem.raw (16 GB)
  Process captured: rclone.exe
  Config found:     temp.conf → destination: 198.51.100.23
```

---

## Key Findings from RAM

### 1. rclone.exe confirmed in memory

`rclone.exe` was running as an active process at the time of capture. The RAM image confirms:
- The process was live and executing the exfiltration transfer
- It was not a residual artifact — it was actively connected to `198.51.100.23` (consistent with netstat evidence)

### 2. temp.conf recovered

A temporary configuration file used by `rclone.exe` was found in memory:

```
Config file: temp.conf
Destination: 198.51.100.23
```

| Finding | Significance |
|---------|-------------|
| Config file named `temp.conf` | Attacker used a temporary config to avoid leaving persistent traces on disk |
| Destination: `198.51.100.23` | Directly links the process to the external exfiltration IP — confirms intent |
| Found in RAM, not on disk | Would have been lost on shutdown — RAM capture was essential |

---

## Why RAM Capture Was Critical Here

| If RAM had NOT been captured | What would have been lost |
|-----------------------------|--------------------------|
| `temp.conf` contents | Exfiltration destination would require network forensics to reconstruct |
| Confirmation of `rclone.exe` as the active process | PID 4188 from netstat would have no process to link to after shutdown |
| Any encryption keys or credentials held in memory | Unrecoverable — plaintext secrets exist only in RAM while the process runs |
| Evidence of encoded PowerShell payload | Decoded payload may only exist in memory during execution |

> The decision to use EDR isolation instead of shutting down the host was specifically made to preserve this evidence. A hard shutdown at containment time would have destroyed 16 GB of forensic data.

---

## Recommended Next Steps for RAM Analysis

The following analysis should be performed on `jump-02_mem.raw` using memory forensics tools (e.g., Volatility):

- [ ] Extract full process tree to identify parent/child relationships
- [ ] Dump `rclone.exe` process memory for binary analysis
- [ ] Search for decoded PowerShell payload content
- [ ] Extract any network socket artifacts beyond what netstat captured
- [ ] Search for additional credentials or tokens held in memory

---

*Evidence 03 of 5 · Incident IR-2026-0847*
