# Account & Credential Indicators of Compromise
### Incident IR-2026-0847 · QuantiaPay

---

## Compromised Account

| Account | Type | Domain | Status at Containment |
|---------|------|--------|-----------------------|
| `svc_ci_deploy` | Service Account | QuantiaPay | ✅ Token rotated · old credential invalidated |

---

## How the Credential Was Abused

### Root Cause
The authentication token for `svc_ci_deploy` was **exposed in a public build log** from the CI/CD pipeline. The attacker retrieved it and used it to authenticate interactively — a capability the account was never intended to have.

### Behavioral Baseline vs. Observed Activity

| Attribute | Normal Behavior | Observed (Malicious) |
|-----------|----------------|----------------------|
| Login source | `build-runner-03` (10.20.4.77) | `jump-02` (10.20.9.140) ⚠️ |
| Login time | Business hours / pipeline schedule | 03:02:15 AM ⚠️ |
| Activity type | Automated CI/CD pipeline tasks | Interactive WinRM session ⚠️ |
| Commands executed | Pipeline deployment steps | PowerShell encoded commands, AD enumeration ⚠️ |

---

## Authentication Timeline

```
02:47:03  AUTH INFO   svc_ci_deploy login OK ← build-runner-03  [LEGITIMATE]
02:51:22  CI/CD INFO  Pipeline #4471 deploy→prod                 [LEGITIMATE]
03:02:15  AUTH HIGH   svc_ci_deploy login OK ← jump-02           [ANOMALOUS ← BREAKPOINT]
```

> **Breakpoint (e3):** The 03:02:15 login from `jump-02` is the start of the attack chain.
> Same credential, different host, off-hours, no scheduled change to justify it.

---

## Privilege Enumeration Observed

After authenticating to `jump-02`, the attacker immediately queried domain privilege groups:

```
03:06:33  net.exe group "Domain Admins" /domain
03:03:41  LDAP mass query (host enumeration)
```

This indicates the attacker was performing **post-exploitation discovery** to identify further targets for privilege escalation or lateral movement.

---

## Containment Actions Taken

| Action | Result |
|--------|--------|
| Token revoked | ✅ `svc_ci_deploy` old credential invalidated |
| Token rotated | ✅ New credential issued through secrets manager |
| Pipeline patched | ✅ Build logs no longer expose secrets |
| Monitoring rule added | ✅ Alert on any `svc_ci_deploy` login from non-baseline host |

---

## Recommended Detection

Alert on `svc_ci_deploy` (or any service account) authenticating from a host not in its known-good baseline:

```
ALERT  account=svc_ci_deploy  AND  src_host NOT IN [build-runner-03]
```

---

*Last updated: 2026-07-18 · Incident IR-2026-0847*
