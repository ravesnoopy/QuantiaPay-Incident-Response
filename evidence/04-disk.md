# Evidence 04 — Disk Image
### Incident IR-2026-0847 · QuantiaPay

---

## Collection Details

| Field | Value |
|-------|-------|
| Command | `disk-image jump-02` |
| Collected | 2026-07-18 · ~03:12:30 |
| Host | `jump-02` (10.20.9.140) |
| Volatility | 🟡 Medium — non-volatile, safe to run in background |
| Collection order | **#4 of 5** |

---

## Raw Output

```
analyst@quantia-ir:~$ disk-image jump-02

✓ Disk imaging started in background. Non-volatile, safe to defer.
```

---

## Why Disk Was Collected Fourth

Disk imaging was intentionally deferred until after all volatile evidence was secured. Unlike RAM, disk contents survive power cycles and do not change during imaging (on a live system with write-blocking or post-isolation).

Collecting the disk image while netstat, ARP, and RAM were still available allowed the team to:
1. Secure evidence that would be destroyed first
2. Run disk imaging in the background without blocking the investigation
3. Maintain the forensically correct volatility order

---

## What the Disk Image Preserves

| Artifact | Location | Forensic Value |
|----------|----------|---------------|
| `rclone.exe` binary | Attacker staging path | Hash for threat intel matching and binary analysis |
| `C:\Users\Public\svc.exe` | `C:\Users\Public\` | Malicious binary associated with `OneDriveSync` Run key |
| Registry hives | `C:\Windows\System32\config\` | Confirm `OneDriveSync` Run key at time of imaging |
| Scheduled task XML | `C:\Windows\System32\Tasks\` | Full definition of `WindowsUpdateCheck` task |
| PowerShell history | `AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\` | May contain decoded or partial command history |
| Windows Event Logs | `C:\Windows\System32\winevt\Logs\` | Security, System, PowerShell logs for offline analysis |
| Prefetch files | `C:\Windows\Prefetch\` | Execution history — confirms which binaries ran and when |
| MFT (Master File Table) | NTFS metadata | File creation, modification, and access timestamps |

---

## Integrity Verification

Before analysis, the disk image should be verified with a cryptographic hash:

```bash
# Generate hash of disk image
sha256sum jump-02_disk.img > jump-02_disk.img.sha256

# Verify before each analysis session
sha256sum -c jump-02_disk.img.sha256
```

This ensures the image has not been altered since collection and maintains forensic admissibility.

---

## Recommended Next Steps for Disk Analysis

- [ ] Mount image in read-only mode for safe examination
- [ ] Extract and hash `rclone.exe` — submit to threat intel platforms
- [ ] Extract and hash `svc.exe` — analyze for additional capabilities
- [ ] Parse Windows Event Logs for full authentication and execution timeline
- [ ] Examine Prefetch for execution evidence of other tools
- [ ] Check MFT timestamps for timeline reconstruction
- [ ] Search for additional staging files or dropped payloads in temp paths

---

*Evidence 04 of 5 · Incident IR-2026-0847*
