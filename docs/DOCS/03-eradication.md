# Phase 3 — Eradication & Verification
### Incident IR-2026-0847 · QuantiaPay

---

## 1. Root Cause

**How did the attacker get in?**

The service account `svc_ci_deploy` authentication token was **exposed in a public build log** from the CI/CD pipeline. The attacker retrieved the token from that log and used it to authenticate interactively to `jump-02` — a host and usage pattern the account was never intended to have.

```
Root cause confirmed:
→ svc_ci_deploy token leaked in build pipeline log (public)
→ Attacker used token to authenticate to jump-02 at 03:02:15
→ No MFA, no baseline alerting, no secrets management in place
```

**Attack duration:** Approximately 13 minutes of active exfiltration (03:02:15 → 03:15:30+) before containment.

---

## 2. Persistence Mechanisms Removed

Two persistence mechanisms were identified and removed. One of them **reactivated the attacker** before it was caught — both required full removal before the host could be considered clean.

### Mechanism 1 — Scheduled Task: `WindowsUpdateCheck`

| Field | Value |
|-------|-------|
| Type | Windows Scheduled Task |
| Name | `WindowsUpdateCheck` |
| Trigger | Every 30 minutes |
| Action | `powershell -enc <base64>` |
| Risk | Reactivated attacker session during eradication |
| Status | ✅ Removed |

```bash
analyst@quantia-ir:~$ remove-persistence "WindowsUpdateCheck"
✓ removed WindowsUpdateCheck
```

> ⚠️ This task executed once before removal, briefly restoring the attacker's foothold. This is why persistence sweep must be thorough and fast — a single missed mechanism is enough for re-entry.

---

### Mechanism 2 — Registry Run Key: `OneDriveSync`

| Field | Value |
|-------|-------|
| Type | Registry Run Key |
| Name | `OneDriveSync` |
| Binary | `C:\Users\Public\svc.exe` |
| Status | ✅ Removed |

```bash
analyst@quantia-ir:~$ remove-persistence "OneDriveSync"
✓ removed OneDriveSync
```

---

### Credential Actions

```bash
analyst@quantia-ir:~$ rotate-token svc_ci_deploy
✓ Token rotated. Old credential invalidated.

analyst@quantia-ir:~$ monitor-rule add
✓ Monitoring rule added on svc_ci_deploy + 198.51.100.23 family.
```

---

## 3. Six-Point Verification

Each checkpoint backed by concrete evidence — no assumptions.

| # | Checkpoint | Status | Evidence |
|---|-----------|--------|----------|
| 1 | `svc_ci_deploy` credential invalidated | ✅ Resolved | `Token rotated, old credential invalid.` |
| 2 | No persistence mechanisms remaining on `jump-02` | ✅ Resolved | `No persistence mechanisms remaining.` |
| 3 | Attacker access cut | ✅ Resolved | `No active attacker session detected.` |
| 4 | Root cause fixed | ✅ Resolved | `Root cause addressed — build pipeline no longer leaks secrets.` |
| 5 | No additional compromised hosts | ✅ Resolved | `Sweep complete: no additional compromised hosts found.` |
| 6 | Active monitoring in place | ✅ Resolved | `Monitoring rule active.` |

```
analyst@quantia-ir:~$ verify all
✓ account-status:      Token rotated, old credential invalid.
✓ persistence-clear:   No persistence mechanisms remaining.
✓ attacker-access:     No active attacker session detected.
✓ root-cause-fixed:    Root cause addressed — build pipeline no longer leaks secrets.
✓ lateral-movement:    Sweep complete: no additional compromised hosts found.
✓ monitoring-active:   Monitoring rule active.
```

---

## 4. Recovery Decision

| Field | Decision |
|-------|----------|
| Action | **Rebuild** `jump-02` from trusted image |
| RTO | 2 hours |

**Justification:**

The host was accessed by an attacker with valid credentials who deployed persistence mechanisms and ran arbitrary code. Even with all identified artifacts removed, attempting to clean a compromised administrative host carries residual risk of undetected modifications. Rebuilding from a known-good image provides a higher integrity guarantee and eliminates that risk entirely.

The 2-hour RTO reflects the criticality of `jump-02` as an administrative asset while ensuring the rebuild process is not rushed in a way that could reintroduce vulnerabilities.

---

*Score: 95/100 · Phase 3 complete · Incident IR-2026-0847*
