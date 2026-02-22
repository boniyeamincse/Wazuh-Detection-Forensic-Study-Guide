````markdown
# Quick Queries — Formatted Reference

This page provides concise, ready-to-copy KQL filters and short notes for common Wazuh/SIEM tasks. Paste these into the Wazuh Dashboard `Discover` search bar or a panel query.

1️⃣ Basic Filtering

- Filter by Agent

```
agent.name: "server01"
agent.id: "001"
```

- Filter by Rule ID

```
Single: rule.id: 1002
Multiple: rule.id: (1002 OR 5710 OR 18107)
```

- Filter by Severity

```
High severity: rule.level >= 10
Critical only: rule.level >= 12
```

2️⃣ Authentication Monitoring

- Windows Failed Login

```
rule.id: 18107
data.win.system.eventID: 4625
```

- Windows Successful Login

```
data.win.system.eventID: 4624
```

- SSH Failed Login

```
rule.groups: "ssh" AND rule.description: "Failed password"
```

- Brute Force Detection

```
rule.groups: "authentication_failed"
```

3️⃣ Process Monitoring

- PowerShell Execution

```
data.win.eventdata.Image: "*powershell.exe"
```

- Encoded PowerShell

```
data.win.eventdata.CommandLine: "*EncodedCommand*"
```

- Suspicious Parent-Child Process

```
data.win.eventdata.ParentImage: "*winword.exe" AND data.win.eventdata.Image: "*cmd.exe"
```

4️⃣ File Integrity Monitoring (FIM)

- File Added

```
rule.groups: "syscheck" AND syscheck.event: "added"
```

- File Modified

```
syscheck.event: "modified"
```

- Critical Folder Monitoring

```
syscheck.path: "C:\\Windows\\System32\\*"
```

5️⃣ Network & IP Monitoring

- Source IP

```
data.srcip: 192.168.1.10
```

- IP Range

```
data.srcip: 192.168.1.0/24
```

- External IP Detection

```
NOT data.srcip: (10.* OR 192.168.* OR 172.16.*)
```

6️⃣ Malware / Threat Alerts

- Antivirus Detection

```
rule.groups: "virus"
```

- YARA Detection

```
rule.groups: "yara"
```

- Suspicious File Hash

```
data.hash: *
```

7️⃣ Time-Based Queries

```
Last 24 hours: @timestamp >= now-24h
Last 7 days: @timestamp >= now-7d
Custom: @timestamp >= "2026-02-01" AND @timestamp <= "2026-02-22"
```

8️⃣ Threat Hunting / SOC Queries

- Lateral Movement

```
data.win.system.eventID: 4624 AND data.win.eventdata.LogonType: 3
```

- Privilege Escalation

```
rule.description: "special privileges assigned"
```

- Suspicious Service Creation

```
data.win.system.eventID: 7045
```

- Registry Persistence

```
rule.description: "*Run*"
```

9️⃣ Advanced Techniques

- Field Exists

```
data.srcip: *
```

- Missing Field

```
NOT data.srcip: *
```

- Wildcard Search

```
rule.description: "*error*"
```

- Combined Filters

```
agent.name: "server01" AND rule.level >= 10 AND NOT rule.groups: "pci_dss"
```

🔟 Dashboard Quick Filters (SOC)

- Critical Alerts

```
rule.level >= 12
```

- Authentication Monitoring

```
rule.groups: ("authentication_failed" OR "authentication_success")
```

- Top Attacking IPs (use aggregation)

```
data.srcip: *  # then create a Terms aggregation on data.srcip.keyword in the dashboard
```

---

If you want this exported as a printable cheatsheet or added to `README.md` with quick links, tell me which format (PDF / Markdown / HTML) and I will produce it.

````
