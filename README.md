# QuantiaPay Incident Response

> Investigation and response to a simulated cybersecurity incident involving credential compromise, lateral movement, and data exfiltration within the QuantiaPay environment.

---

## Overview

This repository documents a complete Incident Response investigation performed during a simulated enterprise security scenario.

The investigation covers the full incident lifecycle, including:

- Detection
- Triage
- Containment
- Evidence Collection
- Eradication
- Recovery
- Post-Incident Review

---

## Scenario

The Security Operations Center (SOC) detected suspicious activity involving the service account **svc_ci_deploy**.

The investigation identified unauthorized access to the administrative host **jump-02**, execution of encoded PowerShell commands, privilege enumeration, and the exfiltration of sensitive data to an external IP address.

---

## Skills Demonstrated

- Incident Response
- Digital Forensics
- Threat Analysis
- IOC Identification
- Security Monitoring
- Evidence Collection
- Documentation
- Blue Team Operations

---

## Technologies

- Elastic
- Kibana
- Sysmon
- Windows Event Logs
- PowerShell
- Network Analysis

---

## Repository Structure

```

docs/
evidence/
images/
iocs/
timeline/

```

---

## Disclaimer

This repository documents a fictional cybersecurity incident created for educational purposes.

No real systems, organizations, or sensitive information were involved.
