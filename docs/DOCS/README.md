# Documentation
### Incident IR-2026-0847 · QuantiaPay

> This folder contains the full written documentation of the incident lifecycle,
> from initial triage through post-incident review.

---

## Files in this folder

| File | Description |
|------|-------------|
| [`01-declaration.md`](./01-declaration.md) | Incident declaration, breakpoint analysis, scope and confidence assessment |
| [`02-containment.md`](./02-containment.md) | Containment method, firewall rule, evidence preservation sequence |
| [`03-eradication.md`](./03-eradication.md) | Root cause, persistence mechanisms removed, 6-point verification |
| [`04-executive-summary.md`](./04-executive-summary.md) | BLUF report written for non-technical leadership |
| [`05-post-incident-review.md`](./05-post-incident-review.md) | Control gap matrix and prioritized recommendations |

---

## Incident at a Glance

| Field | Value |
|-------|-------|
| Incident ID | IR-2026-0847 |
| Organization | QuantiaPay |
| Severity | P1 — Critical |
| Detection Time | 03:14:00 |
| Declared At | 03:00 AM |
| Declared By | SOC Analyst |
| Containment | 03:12:57 (EDR isolation + firewall rule) |
| Data Exfiltrated | 5.3 GB |
| Systems Affected | `jump-02` (10.20.9.140) |
| Accounts Compromised | `svc_ci_deploy` |
| Root Cause | Service account token exposed in public build log |
| Final Score | 99 / 100 |

---

*Last updated: 2026-07-18 · Incident IR-2026-0847*
