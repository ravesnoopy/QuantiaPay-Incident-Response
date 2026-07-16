# Incident Response Investigation – QuantiaPay

## Overview

This repository documents the investigation and response to a simulated cybersecurity incident affecting QuantiaPay, a fictional financial technology company.

The scenario involved suspicious authentication activity, unauthorized access to a CI/CD service account, lateral movement, and the exfiltration of sensitive corporate data. The investigation follows a structured incident response process based on industry best practices.

---

## Objectives

- Identify the initial indicators of compromise (IOCs).
- Analyze attacker activity.
- Contain the incident.
- Preserve forensic evidence.
- Recommend eradication and recovery actions.
- Document findings in a professional incident report.

---

## Scenario Summary

The Security Operations Center (SOC) detected abnormal activity involving the service account **svc_ci_deploy**. Investigation revealed unauthorized access through the **jump-02** host, followed by privilege abuse and the transfer of more than 4 GB of sensitive information outside the organization's network.

The incident required immediate containment, evidence preservation, and recovery planning.

---

## Skills Demonstrated

- Incident Response
- Security Monitoring
- Threat Analysis
- Evidence Collection
- IOC Identification
- Timeline Analysis
- Documentation
- Blue Team Methodology

---

## Tools & Technologies

- Elastic / Kibana
- Sysmon
- Windows Event Logs
- PowerShell
- Network Analysis
- Digital Forensics Concepts

---

## Repository Structure

```text
report/
evidence/
images/
timeline/
```

---

## Key Deliverables

- Executive Incident Report
- Triage Analysis
- Evidence Documentation
- Incident Timeline
- Containment Strategy
- Recovery Recommendations

---

## Lessons Learned

This project strengthened practical skills in incident handling, structured documentation, evidence preservation, and communicating technical findings in a format suitable for SOC environments.

---

## Disclaimer

This project is based on a fictional training scenario created for educational purposes. No real systems or organizations were involved.
