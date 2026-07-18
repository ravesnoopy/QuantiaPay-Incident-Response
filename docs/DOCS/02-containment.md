# Phase 2 — Containment & Evidence Preservation
### Incident IR-2026-0847 · QuantiaPay

---

## 1. Isolation Method

**Chosen method: EDR Isolation of `jump-02`**

```
[11:12:39 PM]  F2 OK — EDR isolation applied · jump-02 ISOLATED · RAM preserved
```

The host was isolated via EDR (Endpoint Detection & Response) rather than a hard shutdown or network-level block alone. This decision was deliberate:

**Why EDR isolation:**
- Stops attacker communication immediately without powering off the host
- **Preserves RAM** — volatile evidence (active processes, credentials, encryption keys, malware in memory) survives
- Keeps the host available for forensic investigation
- Does not interrupt other production systems or payment processing

**Trade-off:**
- The attacker process (`rclone.exe`) remains in memory momentarily — this is acceptable and expected when RAM capture is the immediate next step
- Shutting down would have destroyed volatile evidence irreversibly

---

## 2. Firewall Rule Applied

```bash
fw add-rule deny out src 10.20.9.140/32 dst 198.51.100.23/32 port 443
```

```
[11:12:57 PM]  F2 OK — Surgical /32 rule applied · exfil blocked · production routes intact
```

**Why this exact scope:**

| Decision | Reason |
|----------|--------|
| Source `/32` (single host) | Targets only `jump-02` — no other host affected |
| Destination `/32` (single IP) | Blocks only `198.51.100.23` — the confirmed attacker IP |
| Port `443` only | Matches the observed exfiltration channel exactly |

A broader rule (e.g., blocking all outbound from the subnet) would have risked interrupting legitimate production traffic — unacceptable for a 24/7 payment processor.

---

## 3. Consequences & Lessons

**Positive outcomes:**
- Exfiltration stopped at 5.3 GB (active transfer cut)
- Production routes remained intact — zero collateral damage
- RAM preserved for forensic analysis

**Error encountered during eradication:**

```
[11:14:20 PM]  F4 ERR — RE-INFECTION: WindowsUpdateCheck reactivated the attacker
```

During the eradication phase, the scheduled task `WindowsUpdateCheck` executed before it was removed, briefly reactivating the attacker's foothold. This revealed that **containment alone is insufficient** when persistence mechanisms remain active. A full sweep for all persistence artifacts must be completed before considering the host clean.

---

## 4. Evidence Preservation Sequence

Evidence was captured in order of volatility — most ephemeral first, most persistent last.

| Order | Evidence Type | Command | Why at this moment |
|-------|--------------|---------|-------------------|
| 1 | **Active network connections** | `netstat -anob` | Active connections and their associated processes disappear the moment the attacker disconnects or the host is isolated. Must be first. |
| 2 | **ARP table** | `arp -a` | ARP cache entries expire within minutes. Captures recent host-to-host communication on the local segment. |
| 3 | **RAM image** | `ram-capture jump-02` | Full memory dump (16 GB). Contains running processes, credentials, encryption keys, malware artifacts, and `rclone`'s `temp.conf` with the exfiltration destination. Cannot be recovered after shutdown. |
| 4 | **Disk image** | `disk-image jump-02` | Forensic copy of storage. Non-volatile — safe to defer. Started in background while investigation continues. |
| 5 | **Log export** | `export-logs siem firewall` | SIEM and firewall logs (1.2 GB). At risk of rotation or overwrite — exported before storage fills. |

### Key findings from evidence

```
# netstat -anob
Proto  Local              Foreign              State   PID   Process
TCP    10.20.9.140:5421   198.51.100.23:443    ESTAB   4188  rclone.exe

# RAM capture result
jump-02_mem.raw (16 GB)
→ rclone.exe confirmed in memory
→ temp.conf found: destination = 198.51.100.23
```

---

*Score: 100/100 · Phase 2 complete · Incident IR-2026-0847*
