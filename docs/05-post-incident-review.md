# Phase 5 — Post-Incident Review
### Incident IR-2026-0847 · QuantiaPay

> The goal of this review is not to assign blame — it is to find the gaps in the system
> and close them before the next incident. A recommendation without an owner and a deadline is a wish, not a plan.

---

## 1. Control Gap Matrix

For each attack event: what control was missing or failed, and what type it is.

| Attack Event | Control That Failed | Control Type |
|-------------|---------------------|--------------|
| `svc_ci_deploy` token exposed in build log | No secrets management system in place for the CI/CD pipeline — credentials stored and logged in plaintext | **Preventive** |
| Anomalous login from `jump-02` at 3 AM — no alert fired | No behavioral baseline monitoring for service accounts — logins from non-approved hosts went undetected | **Detective** |
| Active Directory enumeration via LDAP and `net group` | No SIEM/EDR rules detecting enumeration commands or unusual service account behavior against AD | **Detective** |
| 5.3 GB exfiltrated to `198.51.100.23:443` before containment | No DLP or egress filtering to detect and block large outbound transfers to unknown external IPs | **Preventive** |
| `WindowsUpdateCheck` and `OneDriveSync` persistence created without alert | No baseline monitoring for scheduled task creation or Run key modifications — persistence went undetected until active eradication | **Corrective** |

---

## 2. Prioritized Recommendations

Three actionable recommendations ordered by impact and effort.

---

### Recommendation 1 — Secrets Management for CI/CD Pipeline

| Field | Detail |
|-------|--------|
| **What changes** | Implement a dedicated secrets manager (e.g., HashiCorp Vault, AWS Secrets Manager) for all CI/CD pipeline credentials. Eliminate token exposure in build logs. |
| **Owner** | DevOps Team |
| **Deadline** | 30 days |
| **How to verify** | Pipeline audit + validation that no secrets appear in build log output. Automated scanning of logs for credential patterns. |
| **What it prevents** | Initial access via stolen service account tokens — the root cause of this incident. |
| **Impact / Effort** | 🔴 High Impact / 🟡 Medium Effort |

---

### Recommendation 2 — Behavioral Monitoring for Service Accounts

| Field | Detail |
|-------|--------|
| **What changes** | Implement SIEM alerting for service account logins from hosts outside their approved baseline, and logins outside expected time windows. |
| **Owner** | SOC Team |
| **Deadline** | 7 days |
| **How to verify** | Simulate an anomalous login from `svc_ci_deploy` on a non-approved host and confirm the SIEM alert fires within expected SLA. |
| **What it prevents** | Lateral movement using stolen credentials — would have detected the breakpoint event (03:02:15) immediately instead of 12 minutes later. |
| **Impact / Effort** | 🔴 High Impact / 🟢 Low Effort |

---

### Recommendation 3 — Egress Filtering and Outbound Data Monitoring

| Field | Detail |
|-------|--------|
| **What changes** | Strengthen outbound firewall rules and deploy NDR/DLP alerting for large data transfers to external IPs not on an approved allowlist. |
| **Owner** | Network / Security Team |
| **Deadline** | 21 days |
| **How to verify** | Run a controlled test transfer to an unauthorized external IP and verify it is blocked and triggers an alert. |
| **What it prevents** | Data exfiltration — would have stopped or significantly reduced the 5.3 GB transfer before SIEM correlation detected it. |
| **Impact / Effort** | 🔴 High Impact / 🟡 Medium Effort |

---

## 3. Lessons Learned

| # | Lesson |
|---|--------|
| 1 | **Persistence sweep must be exhaustive.** A single missed mechanism (`WindowsUpdateCheck`) briefly restored attacker access during eradication. Never declare clean until every artifact is confirmed removed. |
| 2 | **Service accounts need behavioral baselines.** The anomalous login at 03:02 was the attack's starting point — 12 minutes passed before it was correlated. A baseline alert would have surfaced it instantly. |
| 3 | **Secrets in logs are credentials in the open.** The root cause was a token in a build log, not a sophisticated zero-day. Preventive controls here have disproportionate impact at low cost. |
| 4 | **Surgical containment preserves business continuity.** The `/32` firewall rule stopped exfiltration without touching production. Over-containing (e.g., blocking the entire subnet) would have been an own goal. |
| 5 | **Volatility order matters.** Capturing RAM before isolating the host recovered `rclone`'s config with the exfiltration destination — evidence that would have been lost on shutdown. |

---

*Score: 100/100 · Phase 5 complete · Incident IR-2026-0847*
