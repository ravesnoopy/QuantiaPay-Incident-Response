# Indicators of Compromise (IOCs)

This directory contains all identified Indicators of Compromise during the investigation.

Examples:

- IP Addresses
- User Accounts
- Processes
- Commands
- Hashes


Indicators of Compromise (IOCs)

Incident IR-2026-0847 · QuantiaPay


All indicators listed below were extracted from evidence collected during the active investigation.
These IOCs should be used for threat hunting, detection rule creation, and blocking across security controls.




Files in this folder

FileDescriptionnetwork.mdIP addresses, domains, ports involved in C2 and exfiltrationaccounts.mdCompromised accounts and credential indicatorshost.mdProcesses, persistence mechanisms, and artifacts found on endpointdetection-rules.mdSIEM / EDR detection rules derived from observed TTPs


Quick Reference — Critical IOCs

TypeValueSeverityIP Address198.51.100.23🔴 CriticalAccountsvc_ci_deploy🔴 CriticalHostjump-02 (10.20.9.140)🔴 CriticalProcessrclone.exe (PID 4188)🔴 CriticalPersistenceWindowsUpdateCheck (Scheduled Task)🔴 CriticalPersistenceOneDriveSync (Registry Run Key)🔴 CriticalBinaryC:\Users\Public\svc.exe🔴 Critical


MITRE ATT&CK Mapping

TacticTechniqueIDObservableInitial AccessValid Accounts: Service AccountsT1078.003svc_ci_deploy token exposed in build logLateral MovementRemote Services: Windows Remote ManagementT1021.006WinRM session from build-runner-03 → jump-02DiscoveryDomain Account DiscoveryT1087.002net group "Domain Admins" /domainDiscoveryRemote System DiscoveryT1018LDAP mass query at 03:03:41ExecutionCommand and Scripting Interpreter: PowerShellT1059.001powershell.exe -EncodedCommand <base64>ExfiltrationExfiltration Over C2 ChannelT1041rclone.exe → 198.51.100.23:443 (5.3 GB)PersistenceScheduled Task/JobT1053.005WindowsUpdateCheck taskPersistenceBoot or Logon Autostart: Registry Run KeysT1547.001OneDriveSync Run key


Last updated: 2026-07-18 · Analyst: LeonardoR
