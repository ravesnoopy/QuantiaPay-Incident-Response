# Phase 1 — Triage & Incident Declaration
### Incident IR-2026-0847 · QuantiaPay

---

## 1. Declaration Decision

Three conditions must be met to declare a security incident. All three were confirmed.

| Condition | Met? | Evidence |
|-----------|------|----------|
| Real threat | ✅ Yes | Active data transfer confirmed — `DATA_OUT jump-02 → 198.51.100.23` cumulative 3.8 GB and rising. No prior record of this transfer pattern. High probability of sensitive financial data exfiltration. |
| Assets affected | ✅ Yes | Service account `svc_ci_deploy` and administrative host `jump-02` (10.20.9.140) confirmed compromised. |
| Requires coordinated response | ✅ Yes | SOC, system administrators, and business stakeholders must coordinate containment without interrupting payment processing. Determining attacker entry while routine operations continue is the immediate priority. |

**Final Decision:**

> 🚨 **INCIDENT DECLARED — IR-2026-0847**
> Declared at: 03:00 AM
> Summary: Massive data transfer detected from `jump-02` to external IP `198.51.100.23` using service account `svc_ci_deploy`. Volume and context indicate active exfiltration. Immediate containment and investigation required.

---

## 2. Breakpoint Analysis

**Where does the attack actually begin?**

```
03:02:15  AUTH HIGH  LOGIN_OK svc_ci_deploy ← jump-02 (10.20.9.140)
```

This is the breakpoint — event **e3**.

The service account `svc_ci_deploy` is used exclusively by automated CI/CD pipeline processes. Its login to `jump-02`, an administrative host, represents a clear deviation from baseline:

- No prior record of this account accessing `jump-02`
- Occurred at 03:02 AM, outside any scheduled pipeline window
- No approved change or maintenance window existed to justify it
- Immediately followed by AD enumeration and encoded PowerShell execution

Everything before this event (02:47 login to `build-runner-03`, pipeline #4471 deployment) was **legitimate activity**. The attack begins the moment the credential is used outside its expected context.

---

## 3. Scope, Impact & Confidence Assessment

### Assets

| | Confirmed | Potential | Confidence |
|-|-----------|-----------|------------|
| **Compromised** | `svc_ci_deploy` service account · `jump-02` (10.20.9.140) | Attacker performed infrastructure enumeration — other admin systems at risk, though no additional compromise confirmed at time of declaration | HIGH |

### Data

| | Confirmed | Potential | Confidence |
|-|-----------|-----------|------------|
| **Exfiltrated** | Active outbound transfer from `jump-02` to external IP detected. Host has access to financial infrastructure. High probability of sensitive data exposure. | Exact file set exfiltrated unknown. Full scope requires log review, access audits, and event correlation. | MEDIUM |

---

## 4. Pre-Attack Legitimate Activity (Do Not Confuse)

The following events are **not part of the attack** — they represent normal pipeline behavior occurring immediately before the incident:

```
02:47:03  AUTH INFO   LOGIN_OK svc_ci_deploy ← build-runner-03 (10.20.4.77)   [LEGITIMATE]
02:51:22  CI/CD INFO  Pipeline #4471 deploy→prod  commit=a3f9c2  approved_by=human  [LEGITIMATE]
```

Misidentifying these as malicious would be an error. The attack begins at `03:02:15`.

---

*Score: 100/100 · Phase 1 complete · Incident IR-2026-0847*
