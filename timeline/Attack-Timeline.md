# Attack Timeline

|  Time | -------Event-------| Description
|-------|--------------------|
| 02:47 | Normal Login | `svc_ci_deploy` authenticated from the expected CI/CD runner (`build-runner-03`). |

| 02:51 | Production Deployment | Pipeline #4471 deployed an approved production commit. |

| 03:02 | Suspicious Login | `svc_ci_deploy` authenticated from `jump-02`, an unusual administrative host. |

| 03:05 | PowerShell Execution | Encoded PowerShell command executed remotely through WinRM. |

| 03:06 | Privilege Enumeration | `net group "Domain Admins" /domain` executed. |

| 03:09 | External Connection | Outbound HTTPS connection established to `198.51.100.23`. |

| 03:11 | Data Exfiltration | Large outbound data transfer detected (1.2 GB). |

| 03:14 | SIEM Alert | Correlation rule triggered a Priority 1 incident. |

| 03:15 | Active Exfiltration | Total outbound transfer reached approximately 3.8 GB before containment. |

