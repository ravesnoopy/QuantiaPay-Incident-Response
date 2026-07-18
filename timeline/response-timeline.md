# Response Timeline — Analyst Actions
### Incident IR-2026-0847 · QuantiaPay

> Chronological record of every analyst action taken from detection through verification.
> Source: Session Log IR-2026-0847 · Score: 99/100

---

## Detection

### 03:14:00 — SIEM P1 Alert
SIEM correlation rule fired. Analyst notified. Investigation begins.

---

## Phase 1 — Triage & Declaration `Score: 100/100`

### [11:12:37 PM] — Incident Declared
```
F1 OK — Incident declared · breakpoint=e3 · score=100/100
```

- Breakpoint identified: `03:02:15` — `svc_ci_deploy` login from `jump-02`
- Pre-attack legitimate activity correctly excluded (02:47 and 02:51 events)
- Three declaration conditions confirmed: real threat, assets affected, coordinated response required
- Incident declared as **IR-2026-0847**

---

## Phase 2 — Containment `Score: 100/100`

### [11:12:39 PM] — EDR Isolation Applied
```
F2 OK — EDR isolation · jump-02 ISOLATED · RAM preserved
```

- `jump-02` isolated via EDR — host kept running to preserve RAM
- Attacker communication interrupted
- No production systems affected

### [11:12:57 PM] — Surgical Firewall Rule Applied
```
F2 OK — Surgical /32 rule applied · exfil blocked · production routes intact
```

```bash
fw add-rule deny out src 10.20.9.140/32 dst 198.51.100.23/32 port 443
```

- Exfiltration channel blocked
- Rule scoped to single host → single IP to avoid production impact
- Total data exfiltrated at containment: **5.3 GB**

### [11:12:59 PM] — Containment Phase Closed
```
F2 INFO — Containment closed · score=100/100 · exfil=5.3 GB
```

---

## Phase 3 — Evidence Collection `Score: 100/100`

### [11:13:35 PM] — All 5 Evidence Sources Captured
```
F3 INFO — Evidence: 5 sources captured · score=100
```

| Order | Evidence | Command | Finding |
|-------|----------|---------|---------|
| 1 | Network connections | `netstat -anob` | `rclone.exe` PID 4188 → `198.51.100.23:443` |
| 2 | ARP table | `arp -a` | Gateway `10.20.9.1 → aa-bb-cc-11-22-33` |
| 3 | RAM image | `ram-capture jump-02` | 16 GB · `rclone.exe` + `temp.conf` with exfil dest |
| 4 | Disk image | `disk-image jump-02` | Started in background |
| 5 | Log export | `export-logs siem firewall` | 1.2 GB exported |

---

## Phase 4 — Eradication `Score: 95/100`

### [11:14:20 PM] — ⚠️ Re-infection Event
```
F4 ERR — RE-INFECTION: WindowsUpdateCheck reactivated the attacker
```

The scheduled task `WindowsUpdateCheck` executed before it was removed, briefly restoring the attacker's foothold. Eradication scope was expanded immediately.

### [11:20:36 PM] — Persistence Mechanism 1 Removed
```
F4 INFO — Eradication · score=75
```

```bash
analyst@quantia-ir:~$ remove-persistence "OneDriveSync"
✓ removed OneDriveSync
```

### [11:20:57 PM] — Persistence Mechanism 2 Removed + Credential Rotated
```
F4 INFO — Eradication · score=95
```

```bash
analyst@quantia-ir:~$ remove-persistence "WindowsUpdateCheck"
✓ removed WindowsUpdateCheck

analyst@quantia-ir:~$ rotate-token svc_ci_deploy
✓ Token rotated. Old credential invalidated.

analyst@quantia-ir:~$ monitor-rule add
✓ Monitoring rule added on svc_ci_deploy + 198.51.100.23 family.
```

> **Score note:** 95/100 reflects the re-infection event during eradication. Both persistence mechanisms were fully removed with zero collateral damage to legitimate services.

---

## Phase 5 — Verification `Score: 100/100`

### [11:24:08 PM] — All 6 Checkpoints Verified
```
F5 OK — Verification complete · 6/6 OK
```

```bash
analyst@quantia-ir:~$ verify all

✓ account-status:      Token rotated, old credential invalid.
✓ persistence-clear:   No persistence mechanisms remaining.
✓ attacker-access:     No active attacker session detected.
✓ root-cause-fixed:    Root cause addressed — build pipeline no longer leaks secrets.
✓ lateral-movement:    Sweep complete: no additional compromised hosts found.
✓ monitoring-active:   Monitoring rule active.
```

---

## Final Score

```
─────────────────────────────────────────
SCORE GLOBAL: 99/100

  Phase 1 — Triage        100/100   Correct breakpoint
  Phase 2 — Containment   100/100   EDR + /32 rule — perfect
  Phase 3 — Evidence      100/100   5/5 sources captured
  Phase 4 — Eradication    95/100   2/2 removed · 0 collateral
  Phase 5 — Verification  100/100   6/6 checks OK
─────────────────────────────────────────
```

---

## Response Metrics

| Metric | Value |
|--------|-------|
| Breakpoint to SIEM alert | 12 minutes |
| SIEM alert to EDR isolation | ~1 minute |
| SIEM alert to firewall rule | ~1 minute |
| Total data exfiltrated | 5.3 GB |
| Production services disrupted | 0 |
| Legitimate services affected by containment | 0 |
| Persistence mechanisms removed | 2 / 2 |
| Verification checkpoints passed | 6 / 6 |
| Collateral damage | None |

---

*Session Log: IR-2026-0847 · Generated: 2026-07-18T04:45:52.530Z*
